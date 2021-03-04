# 底层链
### 整体设计原则
`CITA-Cloud`总体设计原则是结合区块链技术和云原生技术，实现一个柔性集成开放平台。

具体到底层链部分，则希望底层链能够有足够的灵活性，可以适应各种各样的企业应用场景。

设计上依然采用微服务架构，划分为`Controller`，`Network`，`Consensus`，`Storage`，`Executor`，`KMS`六个微服务。微服务之间相互解耦，达到不同实现可以灵活替换，自由组合的目的。

解耦设计的细节参见[底层链技术白皮书](https://talk.citahub.com/t/topic/1663)。

在此基础上，为了能够快速构建起完整的，成熟的生态。在之前解耦的基础上，让解耦出的每一个微服务都能独立完成某项功能，每个微服务的接口能够自洽，方便直接复用已有的库或者软件。

### 协议详解
#### Network
`Network`微服务，主要提供网络部分的功能，分为收/发两大部分功能。收的部分采用了控制反转，收到的网络报文根据报文头中的字段分发到不同的`Grpc`地址，因此只提供了一个注册接口(`RegisterNetworkMsgHandler`);发的部分提供了单播(`SendMsg`)和广播(`Broadcast`)两个接口。此外就是一个查询网络连接状态的接口(GetNetworkStatus)。

实现上就是已有网络库的简单封装。其中一个实现[network_direct](https://github.com/cita-cloud/network_direct)直接使用系统网络库，内部实现为节点的全互联。另外一个实现[network_p2p](https://github.com/cita-cloud/network_p2p)，使用了已有的一个`p2p`库。

#### Storage
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
    BUTTON = 7;
}
```
用户可以根据自己的需要调整`region`列表。

具体实现上，就是已有存储系统的简单封装。当前实现有基于`sqlite`的[storage_sqlite](https://github.com/cita-cloud/storage_sqlite)和基于`tikv`的[storage_tikv](https://github.com/cita-cloud/storage_tikv)。

`region`的实现，基于`sqlite`时是用不同的`table`来实现；基于`tikv`时是直接把`region`序列化之后拼接在`key`的前面。


#### KMS
`KMS`微服务，主要提供私钥加密存储，以及相关的密码学服务。接口有：`GenerateKeyPair`，`HashData`，`VerifyDataHash`，`SignMessage`，`RecoverSignature`，形成——生成密钥对/哈希以及相关的验证/签名以及相关的验证——这么一个自洽的功能集合。可以认为是一个软件加密机加上一个密码学工具箱。

目前的实现[kms_eth](https://github.com/cita-cloud/kms_eth)和[kms_sm](https://github.com/cita-cloud/kms_sm)，分别实现了`secp256k1+keccak`和`sm2+sm3`两种密码学算法组合。

#### Executor
`Executor`微服务，主要提供根据交易改变链上状态以及查询链上状态的功能。接口有：`Exec`和`Call`，分别对应执行和查询。

至于执行的细节，比如采用何种`VM`；状态有哪些内容；状态如何组织和保存，都由具体实现来决定。

后续会给出一些兼容已有链的特定实现，比如[executor_chaincode](https://github.com/cita-cloud/executor_chaincode)就是兼容`Fabric`的实现。

也可以针对一些特定应用场景，提供特定的`VM`和智能合约编程语言。比如可信计算，隐私计算，数据格式转换等。

甚至可以不提供智能合约，直接针对具体应用实现，类似于原生合约。

#### Consensus
`Consensus`微服务，主要提供让提案在多个共识参与方之间达成一致的功能。接口包括：

1. 实现在`controller`中的：`GetProposal`本节点提交的提案；`CheckProposal`检查别的节点提交的提案；`CommitBlock`确认一个提案。
2. 实现在`Consensus`中的：`CheckBlock`检查同步自别的节点的已经确认的提案；`Reconfigure`处理系统配置变更。

单独这个微服务的功能，可以认为是一个分歧解决机。

实现上，目前尝试了`PoX`类型的[consensus_proof_of_sleep](https://github.com/cita-cloud/consensus_proof_of_sleep)，以及基于`raft`的[consensus_raft](https://github.com/cita-cloud/consensus_raft)。

#### Controller
`Controller`微服务在整个区块链中处于核心的位置，主导所有主要的流程，并给上层用户提供`RPC`接口。

接口除了前述的针对`Consensus`微服务的接口，就是针对上层用户的`RPC`接口。其中最重要的是`SendRawTransaction`发送交易接口，剩下的都是一些信息查询接口。

单独就这个微服务来说，可以认为是一个提案管理系统。用户通过发送交易接口，提交原始交易数据，`Controller`管理这些原始交易数据。通过计算原始交易数据的哈希，组装`CompactBlock`，以及再次哈希，形成`Consensus`需要的提案，管理这些提案。这里所说的管理，包括持久化，同步，以及验证其合法性。

`Blockchain.proto`文件中定义了一套交易和块的数据结构，但是前面所述的从原始交易数据如何产生最终`Consensus`需要的提案，并且这个过程还是要可验证的，这些都由具体实现决定。未来我们会提供一个框架，方便用户自定义整个流程，甚至是自定义交易和块等核心数据结构。

实现上，目前只有一个[controller_poc](https://github.com/cita-cloud/controller_poc)。主要模块还是以自己实现为主，只有同步模块使用了`Syncthing`。实现上的技术细节参见[项目Wiki](https://github.com/cita-cloud/controller_poc/wiki)。整个实现的代码量还是比较大，后续会考虑进一步细化设计，让尽量多的模块能够复用已有的软件或者库。
