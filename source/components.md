# 组件

组件是各个微服务的实现。

每个组件单独一个代码仓库，仓库名称以微服务名称开头，下划线后接用于标识不同实现的名称。

构建物使用微服务名称，以便于相互替换。

比如`storage_rocksdb`，是基于`rocksdb`实现的`Storage`微服务，其构建物为可执行文件`storage`。

组件分为两类：

* 原厂组件，即`CITA-Cloud`自带的组件。
* 第三方组件。

组件还有以下一些指标：
1. 组件的成熟度：1-5，1表示仅实现必要的功能的最小实现，5表示非常成熟的实现。
2. 组件的状态：开发中，维护中，废弃。
3. 组件的授权状态：商业，或者开源。

## 原厂组件

### network_zenoh

介绍： 基于网络库[zenoh](https://zenoh.io/)实现。

特点： 
* 基于[QUIC](https://zhuanlan.zhihu.com/p/32553477)网络协议，弱网络下更稳定，延迟更低。
* 支持`TLS`，通信加密保证安全。
* 支持`Pub/Sub`模式，提供更高层的接口，提供与底层网络连接无关的节点标识。
* 支持消息送达保证，更加稳定可靠。

[代码仓库](https://github.com/cita-cloud/network_zenoh)

[镜像仓库](https://hub.docker.com/r/citacloud/network_zenoh/tags)

成熟度： 4

状态： 维护中

授权： 开源，`Apache-2.0 License`

### storage_rocksdb

介绍： 基于[rocksdb](https://zhuanlan.zhihu.com/p/51285080)的实现。

特点：
* 高效，`KV`数据库，读写效率高。
* 可靠，多数区块链项目都使用`rocksdb`作为存储引擎，稳定性好。

[代码仓库](https://github.com/cita-cloud/storage_rocksdb)

[镜像仓库](https://hub.docker.com/r/citacloud/storage_rocksdb/tags)

成熟度： 4

状态： 维护中

授权： 开源，`Apache-2.0 License`

### crypto_sm

介绍：国密算法的实现，使用`sm2`签名算法和`sm3`哈希算法。

特点：
* 符合中国国家密码标准。
* 高效，纯`Rust`实现，采用多种优化技术。

[代码仓库](https://github.com/cita-cloud/crypto_sm)

[镜像仓库](https://hub.docker.com/r/citacloud/crypto_sm/tags)

成熟度： 4

状态： 维护中

授权： 开源，`Apache-2.0 License`

### crypto_eth

介绍： 兼容以太坊算法的实现，使用`secp256k1`签名算法和`keccak`哈希算法。

特点：
* 兼容以太坊。

[代码仓库](https://github.com/cita-cloud/crypto_eth)

[镜像仓库](https://hub.docker.com/r/citacloud/crypto_eth/tags)

成熟度： 4

状态： 维护中

授权： 开源，`Apache-2.0 License`

### executor_evm

介绍： 基于以太坊的`EVM`实现。

特点：
* 兼容以太坊的智能合约生态。

[代码仓库](https://github.com/cita-cloud/executor_evm)

[镜像仓库](https://hub.docker.com/r/citacloud/executor_evm/tags)

成熟度： 4

状态： 维护中

授权： 开源，`Apache-2.0 License`

### consensus_bft

介绍： 基于[CITA-BFT](https://docs.citahub.com/zh-CN/cita/architecture/cons)实现。

特点：
* 拜占庭容错。
* 线性消息复杂度。

[代码仓库](https://github.com/cita-cloud/consensus_bft)

[镜像仓库](https://hub.docker.com/r/citacloud/consensus_bft/tags)

成熟度： 4

状态： 维护中

授权： 开源，`Apache-2.0 License`

### consensus_raft

介绍： 基于[Raft](https://github.com/tikv/raft-rs)实现。

特点：
* 非拜占庭容错。
* 成熟实现，稳定可靠。

[代码仓库](https://github.com/cita-cloud/consensus_raft)

[镜像仓库](https://hub.docker.com/r/citacloud/consensus_raft/tags)

成熟度： 4

状态： 维护中

授权： 开源，`Apache-2.0 License`

### consensus_overlord

介绍： 基于[overlord](https://github.com/nervosnetwork/overlord)实现。

特点：
* 拜占庭容错。
* 成熟实现。
* 线性消息复杂度。

[代码仓库](https://github.com/cita-cloud/consensus_overlord)

[镜像仓库](https://hub.docker.com/r/citacloud/consensus_overlord/tags)

成熟度： 4

状态： 维护中

授权： 开源，`Apache-2.0 License`

### controller

介绍： 目前唯一的`Controller`实现。

特点：
* 先共识后执行。
* 高性能，流水线式并行。
* `utxo`模型的系统配置管理。
* 丰富的治理功能。

[代码仓库](https://github.com/cita-cloud/controller)

[镜像仓库](https://hub.docker.com/r/citacloud/controller/tags)

成熟度： 4

状态： 维护中

授权： 开源，`Apache-2.0 License`


### 废弃组件

#### network_direct

介绍： 基于`tokio`网络库的实现。

特点：
* 网络直连，简单可靠
* 无通信加密

[代码仓库](https://github.com/cita-cloud/network_direct)

[镜像仓库](https://hub.docker.com/r/citacloud/network_direct/tags)

成熟度： 3

状态： 废弃

授权： 开源，`Apache-2.0 License`

废弃原因：被`network_tls`替代。因为区块链的去中心化属性，网络通信加密是比较基础的需求。

#### network_p2p

介绍： 基于网络库[tentacle](https://github.com/nervosnetwork/tentacle)实现。

特点： 
* 支持[secio](https://github.com/libp2p/specs/blob/master/secio/README.md)，通信加密保证安全。
* 支持多路复用(`yamux`)，可以自定义协议。
* 支持节点发现，节点之间会自动交换连接的节点信息。

[代码仓库](https://github.com/cita-cloud/network_p2p)

[镜像仓库](https://hub.docker.com/r/citacloud/network_p2p/tags)

成熟度： 4

状态： 废弃

授权： 开源，`Apache-2.0 License`

废弃原因：被`network_zenoh`替代。

#### network_tls

介绍： 基于[tokio-rustls](https://crates.io/crates/tokio-rustls)实现。

特点：
* 支持`TLS1.3`，通信加密保证安全。
* 使用标准的`x509`证书，方便复用已有的基础设施。
* 支持白名单，便于权限管理。

[代码仓库](https://github.com/cita-cloud/network_tls)

[镜像仓库](https://hub.docker.com/r/citacloud/network_tls/tags)

成熟度： 4

状态： 废弃

授权： 开源，`Apache-2.0 License`

废弃原因：被`network_zenoh`替代。

#### network_quic

介绍： 基于[QUIC](https://zhuanlan.zhihu.com/p/32553477)网络协议的实现。

特点：
* 高效，基于`UDP`协议，开销更小。
* 安全，默认支持`TLS`，通信加密。
* 可靠，弱网络下效果更高。

[代码仓库](https://github.com/cita-cloud/network_quic)

[镜像仓库](https://hub.docker.com/r/citacloud/network_quic/tags)

成熟度： 2

状态： 废弃

授权： 开源，`Apache-2.0 License`

废弃原因：被`network_zenoh`替代。

#### storage_sqlite

介绍： 基于[sqlite](https://sql50.readthedocs.io/zh_CN/latest/sqlite_intro.html)的实现。

特点：
* 轻量，嵌入式数据库，开销小。
* 功能丰富，完整支持`SQL`。

[代码仓库](https://github.com/cita-cloud/storage_sqlite)

[镜像仓库](https://hub.docker.com/r/citacloud/storage_sqlite/tags)

成熟度： 3

状态： 废弃

授权： 开源，`Apache-2.0 License`

废弃原因：被`storage_rocksdb`替代。目前区块链的实现中主要还是以`KV`存储为主，`SQL`数据库的优势发挥不出来，反而性能上不如`KV`数据库。也许将来对链上数据分析有更多需求的时候可以切换至`SQL`数据库。

#### storage_tikv

介绍： 基于[tikv](https://docs.pingcap.com/zh/tidb/stable/tikv-overview)的实现。

特点：
* 扩展能力强，分布式`KV`数据库。
* 稳定可靠，支持分布式事务操作，得到广泛应用。

[代码仓库](https://github.com/cita-cloud/storage_tikv)

[镜像仓库](https://hub.docker.com/r/citacloud/storage_tikv/tags)

成熟度： 2

状态： 废弃

授权： 开源，`Apache-2.0 License`

废弃原因：这个组件主要就是验证使用分布式`KV`数据库的可行性。但是目前数据量还没有到这个程度，所以暂时搁置。

#### executor_chaincode

介绍： 实验性兼容`Fabric Chaincode`实现。

特点：
* 兼容`Fabric`的智能合约生态。

[代码仓库](https://github.com/cita-cloud/executor_chaincode)

[镜像仓库](https://hub.docker.com/r/citacloud/executor_chaincode/tags)

成熟度： 1

状态： 废弃

授权： 开源，`Apache-2.0 License`

废弃原因：本身就是实验性质的组件，为了验证框架有足够的灵活性。后期有相关需求之后完善之后成为`executor_chaincode_ext`。


## 第三方组件


### 废弃组件

#### executor_chaincode_ext

介绍： 增强型兼容`Fabric Chaincode`实现。

特点：
* 兼容`chaincode`合约。
* 支持了`CouchDB`。
* 增加了`chaincode`事件相关功能。

代码仓库： 无
镜像仓库： 无

成熟度： 3

状态： 废弃

授权： 商业

废弃原因：专门为某个商业项目定制，后续没有类似的需求。

#### kms_sdibc

介绍： 基于高性能国密算法实现。

特点：
* 性能好。

代码仓库： 无
镜像仓库： 无

成熟度： 4

状态： 废弃

授权： 商业

废弃原因：专门为某个商业项目定制，后续没有类似的需求。
