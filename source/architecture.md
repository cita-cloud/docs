# 架构设计

`CITA-Cloud`总体设计上采用微服务架构，划分为`Controller`，`Network`，`Consensus`，`Storage`，`Executor`，`KMS`六个微服务。

微服务接口定义参见[cita_cloud_proto](https://github.com/cita-cloud/cita_cloud_proto)。

微服务之间相互解耦，达到不同实现可以灵活替换，自由组合的目的。解耦设计的细节参见[底层链技术白皮书](https://talk.citahub.com/t/topic/1663)。

为了能够快速构建起完整的，成熟的生态。在之前解耦的基础上，让解耦出的每一个微服务都能独立完成某项功能，每个微服务的接口能够自洽。使其不只是作为`CITA-Cloud`的组件存在，还可以拥有自己独立的生态。

## Network

`Network`微服务，维护与其他节点之间的网络连接，为本节点的其他微服务提供网络服务。

主要服务有收/发网络消息和节点管理，状态查询。

### 接收网络消息

收的部分采用了控制反转，收到的网络消息根据消息头中的`module`字段分发到其他微服务，并通过回调其他微服务的`gRPC`接口的方式在微服务间传递网络消息。

因此只有一个注册接口，用于其他需要网络服务的微服务注册信息。

```
// 注册网络服务接口
rpc RegisterNetworkMsgHandler(RegisterInfo) returns (common.StatusCode);

// 注册网络服务所需的信息
message RegisterInfo {
    string module_name = 1;  // 微服务名称
    string hostname = 2;     // 网络消息分发时，回调地址的域名
    string port = 3;         // 网络消息分发时，回调地址的端口
}

// 网络消息分发时，回调的 gRPC 接口
// 注册网络服务的其他微服务必须实现该接口
service NetworkMsgHandlerService {
    rpc ProcessNetworkMsg(NetworkMsg) returns (common.StatusCode);
}

// 网络消息结构
message NetworkMsg {
    string module = 1;  // 接收的微服务名称
    string type = 2;    // 消息类型，用于在同一个微服务内区分不同的消息
    uint64 origin = 3;  // 消息的节点标识
    bytes msg = 4;      // 消息数据
}
```

### 发送网络消息
发的部分提供了单播(`SendMsg`)和广播(`Broadcast`)两个接口。

```
// 发送消息给一个特定的节点
// 通过消息中的 origin 字段指定接收节点的标识
rpc SendMsg(NetworkMsg) returns (common.StatusCode);

// 广播消息
// 消息中的 origin 字段被忽略
rpc Broadcast(NetworkMsg) returns (common.StatusCode);
```

关于消息中的`origin`字段，在收到网络消息之后，需要对其进行一个处理。

发送时填的是接收节点的标识，接收到之后会讲该字段修改为发送放的标识。

### 状态查询

```
message NetworkStatusResponse {
    uint64 peer_count = 1;
}

rpc GetNetworkStatus(common.Empty) returns (NetworkStatusResponse);
```

查询网络连接状态的接口，返回当前连接的节点数量。


```
message NodeNetInfo {
    string multi_address = 1;
    uint64 origin = 2;
}

message TotalNodeNetInfo {
    repeated NodeNetInfo nodes = 1;
}

rpc GetPeersNetInfo(common.Empty) returns (common.TotalNodeNetInfo);
```

查询节点网络信息的接口，返回连接的邻居节点的网络地址和标识信息。

### 节点管理

```
message NodeNetInfo {
    string multi_address = 1;
    uint64 origin = 2;
}

rpc AddNode(common.NodeNetInfo) returns (common.StatusCode);
```

增加节点信息的接口(`AddNode`)，用于临时增加一个节点到网络中。

## Storage

`Storage`微服务，主要提供`KV`存储相关的功能，涵盖了常用的增删改查功能。接口上有：`Store`(其语义是`updata`，同时包含增和改的功能)，`Load`，`Delete`。

针对区块链业务，预先定义了不同的`region`：

```
enum Regions {
    GLOBAL = 0;
    TRANSACTIONS = 1;
    HEADERS = 2;
    BODIES = 3;
    BLOCK_HASH = 4;
    PROOF = 5;
    RESULT = 6;
    TRANSACTION_HASH2BLOCK_HEIGHT = 7;
    BLOCK_HASH2BLOCK_HEIGHT = 8;  // In SQL db, reuse 4
    TRANSACTION_INDEX = 9;
    BUTTON = 10;
}
```

用户可以根据自己的需要调整`region`列表。

具体实现上，就是已有存储系统的简单封装。

当前实现有基于`sqlite`的[storage_sqlite](https://github.com/cita-cloud/storage_sqlite)，基于`rocksdb`的[storage_rocksdb](https://github.com/cita-cloud/storage_rocksdb)，以及基于`tikv`的[storage_tikv](https://github.com/cita-cloud/storage_tikv)。

`region`的实现，基于`sqlite`时是用不同的`table`来实现；基于`tikv`时是直接把`region`序列化之后拼接在`key`的前面；基于`rocksdb`时是用不同的`ColumnFamily`来实现。

#### KMS

`KMS`微服务，主要提供私钥加密存储，以及相关的密码学服务。接口有：`GenerateKeyPair`，`HashData`，`VerifyDataHash`，`SignMessage`，`RecoverSignature`，形成——生成密钥对/哈希以及相关的验证/签名以及相关的验证——这么一个自洽的功能集合。此外还有一个查询接口`GetCryptoInfo`。可以认为是一个软件加密机加上一个密码学工具箱。

目前的实现[kms_eth](https://github.com/cita-cloud/kms_eth)和[kms_sm](https://github.com/cita-cloud/kms_sm)，分别实现了`secp256k1+keccak`和`sm2+sm3`两种密码学算法组合。

#### Executor

`Executor`微服务，主要提供根据交易改变链上状态以及查询链上状态的功能。接口有：`Exec`和`Call`，分别对应执行和查询。

至于执行的细节，比如采用何种`VM`；状态有哪些内容；状态如何组织和保存，都由具体实现来决定。

后续会给出一些兼容已有链的特定实现，比如[executor_chaincode](https://github.com/cita-cloud/executor_chaincode)就是兼容`Fabric`的实现。[executor_evm](https://github.com/cita-cloud/executor_evm)则是兼容`以太坊`的实现。

也可以针对一些特定应用场景，提供特定的`VM`和智能合约编程语言。比如可信计算，隐私计算，数据格式转换等。

甚至可以不提供智能合约，直接针对具体应用实现，类似于原生合约。

#### Consensus

`Consensus`微服务，主要提供让提案在多个共识参与方之间达成一致的功能。接口包括：

1. 实现在`controller`中的：`GetProposal`本节点提交的提案；`CheckProposal`检查别的节点提交的提案；`CommitBlock`确认一个提案。
2. 实现在`Consensus`中的：`CheckBlock`检查同步自别的节点的已经确认的提案；`Reconfigure`处理系统配置变更。

单独这个微服务的功能，可以认为是一个分歧解决机。

实现上，目前尝试了`PoX`类型的[consensus_proof_of_sleep](https://github.com/cita-cloud/consensus_proof_of_sleep)，基于`raft`的[consensus_raft](https://github.com/cita-cloud/consensus_raft)，以及基于`bft`的[consensus_bft](https://github.com/cita-cloud/consensus_bft)。

#### Controller

`Controller`微服务在整个区块链中处于核心的位置，主导所有主要的流程，并给上层用户提供`RPC`接口。

接口除了前述的针对`Consensus`微服务的接口，就是针对上层用户的`RPC`接口。其中最重要的是`SendRawTransaction`发送交易接口，剩下的都是一些信息查询接口。

单独就这个微服务来说，可以认为是一个提案管理系统。用户通过发送交易接口，提交原始交易数据，`Controller`管理这些原始交易数据。通过计算原始交易数据的哈希，组装`CompactBlock`，以及再次哈希，形成`Consensus`需要的提案，管理这些提案。这里所说的管理，包括持久化，同步，以及验证其合法性。

`Blockchain.proto`文件中定义了一套交易和块的数据结构，但是前面所述的从原始交易数据如何产生最终`Consensus`需要的提案，并且这个过程还是要可验证的，这些都由具体实现决定。未来我们会提供一个框架，方便用户自定义整个流程，甚至是自定义交易和块等核心数据结构。

实现上，目前只有一个[controller_poc](https://github.com/cita-cloud/controller_poc)。主要模块还是以自己实现为主，只有同步模块使用了`Syncthing`。实现上的技术细节参见[项目Wiki](https://github.com/cita-cloud/controller_poc/wiki)。整个实现的代码量还是比较大，后续会考虑进一步细化设计，让尽量多的模块能够复用已有的软件或者库。
