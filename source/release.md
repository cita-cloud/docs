# 版本发布
## 最新版本

### v6.4.1

本次版本更新主要在系统稳定性方面做了优化，并修复了一些`bug`。

主要更新内容如下：
1. 业务监控指标对接监控平台 
2. check_proposal_proof bug修复
3. libsm错误处理问题修复
4. 实验性支持consensus_overlord

#### Controller
##### [Feature]
[feat] support consensus_overlord @rink1969

#### consensus_bft
##### [Fix]
[fix] fix commit_block proof inconsistent  @JLerxky

#### cloud-config
##### [Feature]
[feat] support consensus_overlord  @rink1969
[feat] adapt for libsm @NaughtyDogOfSchrodinger

#### executor_evm
##### [Optimization]
[optim] add debug info in release  @rink1969  

#### storage_rocksdb
##### [Feature]
[feat] adapt for libsm @NaughtyDogOfSchrodinger

#### kms_sm
##### [Fix]
[fix] adapt for fix panic in libsm @NaughtyDogOfSchrodinger

#### cloud-cli
##### [Feature]
[feat] upgrade libsm; support overlord @rink1969

#### 兼容性问题
1. 数据兼容
    v6.4.1 与 v6.4.0 数据完全兼容。旧有的 v6.3.3 以及 v6.3.4 链跑出来的数据，只需要停链然后使用新的 v6.4.0 镜像启动，就可以正常出块。

2. 版本兼容
    v6.4.1 与 v6.4.0 版本的节点可以混合组成网络，并正常出块

3. 配置变更
    v6.4.1 与 v6.4.0  的配置未发生修改或减少

#### 遗留问题

1. `WAL`损坏问题。
2. 增加删除节点会报错。

## 历史版本

### v6.4.0

更新概览：本次版本在稳定性和运维方面做了逐多优化，如果你想要多方共同参与创建联盟链，同时又不想暴露自己的私钥，那么你可以关心gitops这一新功能；如果你担心自己的机器哪天会爆炸，那么你可以关注cloud-op新的快照功能；如果你想要想要节省硬盘运维开销，那么你可以关注execuotr-evm的full mode模式，这比过去的archive mode在硬盘占用上有不少的提升。
主要更新内容如下：
1. 集成测试的搭建
2. cita-cloud集成gitops
3. 运维工具cloud-op提供快照功能
4. 提供输出完整区块的RPC接口，同时cloud-cli完成支持
5. execuotr-evm支持full mode和archive mode切换
6. charts增加微服务资源限制
7. controller正确处理节点重连和关机情况
8. network-tls的TLS证书支持多级证书体系
9. cita-cloud增加了健康检查(livness check)
10. executor_evm的evm升级到London

#### Controller

##### [Feature]
[feat] add inner block growth check @Pencil-Yao  
[feat] add rpc get_block_detail_by_number @JLerxky  
[feat] add health check @Pencil-Yao  
[feat] add health check  @rink1969
##### [Fix]
[fix] repeat node set @Pencil-Yao  
##### [Optimization]
[optim] handle repeat & idle node @Pencil-Yao  
[optim] hanle the same origin node  @Pencil-Yao  
[optim] record package limit in utxo db  @NaughtyDogOfSchrodinger

#### consensus_raft
##### [Feature]
[feat] add health check  @rink1969

#### consensus_bft
##### [Feature]
[feat] add health check  @rink1969
##### [Optimization]
[optim] optimize handle commit entries @NaughtyDogOfSchrodinger

#### cloud-config
##### [Feature]
[feat] add livness probe  @rink1969  
[feat] Support gitops  @rink1969  
[feat] add container resource requirements  @rink1969
##### [Fix]
[fix] panic when create-dev  @rink1969  
[fix] update node, protect rewrite file in used  @rink1969

#### executor_evm
##### [Feature]
[feat] add health check  @rink1969  
[feat]  add base_fee opcode  @JLerxky  
[feat] support switch between full mode and archive mode  @Pencil-Yao

#### storage_rocksdb
##### [Feature]
[feat] add health check  @rink1969

#### kms_sm
##### [Feature]
[feat] add health check  @rink1969

#### network_tls
##### [Feature]
[feat] add health check  @rink1969
##### [Optimization]
[optim] update dependence tokio-rustls and unit-test   @Pencil-Yao

#### network_p2p
##### [Feature]
[feat] add health check  @rink1969
kms_eth
##### [Feature]
[feat] add health check  @rink1969

#### cloud-cli
##### [Feature]
[feat] add get_block_detail @JLerxky  
[feat] add set-package-limit and set-block-limit sub-command @NaughtyDogOfSchrodinger
##### [Optimization]
[optim] use cloud proto @JLerxky

#### 兼容性问题
1. 数据兼容
   v6.4.0 与 v6.3.3 以及 v6.3.4 数据完全兼容。旧有的 v6.3.3 以及 v6.3.4 链跑出来的数据，只需要停链然后使用新的 v6.4.0 镜像启动，就可以正常出块。

2. 版本兼容
   v6.4.0 与 v6.3.3 以及 v6.3.4 版本的节点可以混合组成网络，并正常出块

3. 配置变更
   v6.4.0 与 v6.3.3 以及 v6.3.4 的配置未发生修改或减少

#### 遗留问题

1. `WAL`损坏问题。
2. 增加删除节点会报错。

### v6.3.3

本次版本在稳定性和运维方面做了优化，主要更新内容如下：

1. 修复上个版本 `WAL` 功能引入的问题。
2. 支持消块工具。可以修复因为意外导致区块链节点数据不一致的问题。
3. 梳理微服务的命令行参数。参数统一从配置文件传递，配置更清晰，更统一。也为将来支持`gitops`打下基础。
4. `network`支持配置文件热更新。增加删除节点之后，已有节点可以自动感知到网络的变化。
5. `CLI`重构。新增交互式模式，优化用户体验。参见[文档](https://cita-cloud.github.io/cloud-cli/)。
6. 更新文档。新的文档更加面向链的定制开发者。 参见[文档](https://cita-cloud-docs.readthedocs.io/zh_CN/latest/)。
7. `k8s Operator`实验性支持。参见[项目](https://github.com/cita-cloud/operator-proxy)。

修改详情：

#### Controller

##### [Feature]

[feat] update readme  @rink1969  @JLerxky

[feat] Support cloud-op  @Pencil-Yao  @JLerxky

##### [Fix]

[fix] Add the judgment of not redoing wal  @JLerxky

[fix] save current_block_hash before saving current_block_height  @Pencil-Yao 

##### [Optimization]

[optim] handle add existed peer  @Jayanring

[optim] use cloud_util::wal  @JLerxky

[optim] run -c & -l  @JLerxky

#### consensus_raft

##### [Feature]

[feat] update readme  @rink1969 @NaughtyDogOfSchrodinger

[feat] support recover  @Pencil-Yao

#### consensus_bft

##### [Feature]

[feat] update readme  @NaughtyDogOfSchrodinger

[feat] Support cloud-op  @Pencil-Yao  @JLerxky

##### [Optimization]

[optim] use cloud_util::wal  @JLerxky

[optim] run -c & -l  @JLerxky

#### cloud-config

##### [Feature]

[feat] Update k8s subcmd  @NaughtyDogOfSchrodinger

[feat] output stdout and log file at same time  @rink1969

##### [Fix]

[fix] fix typo  @rink1969

[fix] add license and copyright info  @rink1969

[fix] update images  @rink1969

[fix] fix update node, protect rewrite file in used  @rink1969

##### [Optimization]

[optim] github ci & fix: fmt @Pencil-Yao  @Jayanring

[chore] update clap  @JLerxky

#### executor_evm

##### [Feature]

[feat] update readme  @rink1969

[feat] Support cloud-op  @Pencil-Yao  @JLerxky

##### [Optimization]

[optim] run -c & -l  @JLerxky

#### storage_rocksdb

##### [Feature]

[feat] update readme  @rink1969

[feat] Support cloud-op  @Pencil-Yao  @JLerxky

##### [Optimization]

[optim] run -c & -l  @JLerxky

#### kms_sm

##### [Feature]

[feat] remove create subcmd; update example and readme  @rink1969

##### [Optimization]

[optim] run -c & -l  @JLerxky

#### network_tls

##### [Feature]

[feat] update readme  @rink1969

[feat] support hot update + peer-count + origin  @Pencil-Yao  @Jayanring

##### [Optimization]

[optim] run -c  @JLerxky

#### network_p2p

##### [Feature]

[feat] update readme  @rink1969

##### [Optimization]

[optim] run -c & -l  @JLerxky

#### kms_eth

##### [Feature]

[feat] remove create subcmd; update example and readme  @rink1969

##### [Fix]

[fix] check invalid msg  @rink1969

##### [Optimization]

[optim] run -c & -l  @JLerxky

#### 兼容性问题
1. 数据兼容
   
    v6.3.3 与 v6.3.2 数据完全兼容。旧有的 v6.3.2 链跑出来的数据，只需要停链然后使用新的 v6.3.3 镜像启动，就可以正常出块。

2. 版本兼容
   
    v6.3.3 与 v6.3.2 版本的节点可以混合组成网络，并正常出块

3. 配置变更
   
    微服务的参数有变化，但是采用默认值的情况下，行为与v6.3.2一致。

    如果有自行设置参数的情况，可能需要进行修改。

#### 遗留问题

1. `WAL`损坏问题。

### v6.3.2

本次版本更新对稳定性和运维作了优化，稳定性方面链能够在高负载的情况下更稳定的运行，主要更新内容如下：

1. 优化了 `consensus_bft` 由于网络时延问题导致的无法正常出块的问题；
2. `controller` 具备 WAL 的能力，可以处理进程突然被杀掉而引发的存储问题。
3. 升级的微服务添加了 apache license

修改详情：

#### [Controller](https://github.com/cita-cloud/controller/tree/v6.3.2-alpha.1)

##### [Feature]

[feat] add wal [@JLerxky @Pencil-Yao @Jayanring]

[feat] support retransmit chain_status_init regularly [@Pencil-Yao ]

[feat] store genesis system config [@JLerxky @Jayanring]

##### [Fix]

[fix] executor init [@Pencil-Yao]

[fix] in csi mode, can't send block request [@Pencil-Yao]

[fix] dup tx in busy environment [@Pencil-Yao]

##### [Optimization]

[optim] confirm executor when init chain [@Jayanring]

[optim] process earlystatus logic [@Pencil-Yao ]

[optim] chain lock [@Pencil-Yao ]

#### [Consensus_bft](https://github.com/cita-cloud/consensus_bft/tree/v6.3.2-alpha.1)

##### [Fix]

[fix] handle leader_commit error with dup-tx [@Pencil-Yao ]

#### [Executor_evm](https://github.com/cita-cloud/executor_evm/tree/v6.3.2-alpha.1)

##### [Feature]

[feat] hanle same or invalid block re-enter [@Pencil-Yao ]

##### [Optimization]

[optim] use reenter-invalid block code [@Pencil-Yao ]

#### [Cloud-config](https://github.com/cita-cloud/cloud-config/tree/v6.3.0)

##### [Feature]

[feat] rewrite! from cita_cloud_config [@NaughtyDogOfSchrodinger, @Pencil-Yao ]

[feat] support config [@NaughtyDogOfSchrodinger ]

##### [BugFix]

[fix] fix series 6.3.0 adaption problem [@Pencil-Yao, @NaughtyDogOfSchrodinger ]

#### 兼容性问题

1. 数据兼容

    v6.3.2 与 v6.3.1 数据完全兼容。旧有的 v6.3.1 链跑出来的数据，只需要停链然后使用新的 v6.3.2 镜像启动，就可以正常出块。

2. 版本兼容

    v6.3.2 与 v6.3.1 版本的节点可以混合组成网络，并正常出块


### v6.3.0
[版本发布说明](https://github.com/cita-cloud/operator/releases/tag/6.3.0)

### v6.2.0
[版本发布说明](https://github.com/cita-cloud/operator/releases/tag/6.2.0)

### v6.1.0
[版本发布说明](https://github.com/cita-cloud/operator/releases/tag/v6.1.0)

### v6.0.0
[版本发布说明](https://github.com/cita-cloud/operator/releases/tag/v6.0.0)

### v5.0.0
[版本发布说明](https://github.com/cita-cloud/runner_k8s/releases/tag/v5.0.0)

### v4.0.0
[版本发布说明](https://github.com/cita-cloud/runner_k8s/releases/tag/v4.0.0)

### v3.0.0
[版本发布说明](https://github.com/cita-cloud/runner_k8s/releases/tag/v3.0.0)
