# 版本发布
## 最新版本

### 6.7.0

主要更新内容如下：

#### 底链微服务

- 支持陆羽跨链插件的证明和验证
  - 完成controller、executor、cli、proto配套修改
- 性能优化
  - Proposal中使用CompactBlock，降低传输数据量
  - crypto用作库，不再作为独立微服务，降低调用耗时
- Controller
  - 代码重构，优化代码结构
  - 移除wal，上个版本修改：未执行成功的区块由consensus重新commit，controller中的wal不再需要
  - 同步缺失的交易，Proposal不再包含完整交易，因此缺失某些交易时要从其他节点获取
- Storage_opendal替换原Storage_rocksdb
  - 冷热数据分离。热数据存在本地，冷数据存到云存储，可以使得本地存储数据量较小且相对固定。第一层内存，第二层rocksdb，第三层云存储（可选），可以配置前两层保存的容量（单位：高度）以及向第三层备份的间隔。
  - 共享第三层。多个节点共享第三层云存储，新节点无需恢复大量的冷数据，而只需通过恢复热数据即可快速加入。
  - 支持更多类型的云存储，亚马逊s3、Azure blob、阿里云oss、华为云ods、腾讯云cos
  - 第三层启动检查，如果配置了第三层，启动时会检查其可用性
  - 备份优化，第二层高度为h，则第三层会备份到高度h-1，防止分叉块被备份到多节点共享的云存储中
- 调整grpc消息的大小限制，tonic升级后限制了消息的大小，导致发批量交易报错
- Cloud-config
  - 移除crypto相关配置，storage_opendal配置替换storage_rocksdb配置
  - 修复执行init_node错误，“.”文件夹导致，忽略“.”开头的文件
- consensus_raft
  - 修复持久化entry和snapshot失败，调用flush将缓冲区数据立即写入硬盘
- executor_evm
  - 更新cita-database
- cloud-cli
  - 修复单元测试偶尔失败
  - dockerfile添加pkg-config、libssl-dev依赖
- rollup
  - 缓存接入DAS，并添加适配器，适配多种DAS
  - 缓存validator配合修改
  - 缓存适配cita-cloud v6.7.0

#### 云原生

- 同步上游版本至v2.7.1
- 添加字段：DeleteConsensusData，用于恢复或消块时是否删除共识数据
- 相关Bugfix修复

#### 集成测试

- 补充完善测试用例

#### Controller

##### [Feature]

[feat] feat: support cross chain proof @Pencil-Yao 

##### [Fix]

[fix] set grpc server max_decoding_message_size to max @JLerxky

##### [Optim]

[optim] rmove wal @JLerxky 

[optim] change block in proposal to compact block @rink1969

[optim] sync tx when miss raw tx @rink1969 

[optim] use crypto as lib @rink1969 

[optim] optim: use simple method replace of get_full_block @Pencil-Yao 

##### [Chore]

[chore] add step comments @JLerxky 

[chore] rm crypto port @rink1969 

##### [Refactor]

[refactor] refactor: extract the grpc mod @JLerxky 

[refactor] refactor and optim @JLerxky 

#### network_zenoh

##### [Fix]

[fix] set grpc server max_decoding_message_size to max @JLerxky

#### consensus_overlord

无更新

#### consensus_raft

##### [Fix]

[fix] fix: flush file @Jayanring

#### executor_evm

##### [Feature]

[feature] [breaking]feat: support cross chain proof @Pencil-Yao

##### [Fix]

[fix] fix: re-enter use state_root not app_hash @Pencil-Yao

[fix] set grpc server max_decoding_message_size to max @JLerxky

[fix] fix: re-enter block receipt is null @Pencil-Yao

##### [Chore]

[chore] chore: update cita-database @Pencil-Yao

#### storage_opendal

##### [Feature]

[feature] public layer3 @Jayanring 

[feature] feat: support more cloud storage @Jayanring 

##### [Fix]

[fix] fix: test

[fix] fix layer 2 3 different height key @Jayanring

[fix] enable rocksdb dafault features @Jayanring 

[fix] set grpc server max_decoding_message_size to max @JLerxky 

##### [Optim]

[optim] optim: not store global hash @Jayanring 

[optim] optim: async store block data @Jayanring 

[optim] backup 1 height slower @Jayanring 

[optim] check layer3 availability when init @Jayanring 

##### [Chore]

[chore] replace panic hook @Jayanring 

[chore] chore: debug print error @Jayanring 

[chore] remove crypto dep @rink1969

[chore] update opendal @Jayanring 

[chore] remove crypto port @rink1969 

[chore] store data in root dir @rink1969 

[chore] public operator @Jayanring 

##### [Refactor]

[refactor] init @Jayanring 

#### cita_cloud_proto

##### [Feature]

[feat] feat: support cross chain @Pencil-Yao

##### [Optim]

[optim] change block in proposal to compact block @rink1969

#### cloud-common-rs

##### [Feature]

[feat] feat: support cross chain proof @Pencil-Yao 

##### [Optim]

[optim] change block in proposal to compact block @rink1969

[optim] set grpc client max_decoding_message_size to max @JLerxky

#### cloud-config

##### [Feature]

[feat] rm crypto and replace rocksdb with opendal @rink1969 

##### [Fix]

[fix] add features @rink1969 

[fix] fix infinity loop when call init-node @rink1969 

[fix] fix storage config @rink1969 

##### [Chore]

[chore] use crypto as lib @rink1969 

[chore] add cloud storage args @rink1969 

[chore] update readme @rink1969 

#### cloud-cli

##### [Feature]

[feat] feat: support cross chain proof @Pencil-Yao 

##### [Fix]

[fix] fix mock @rink1969

[fix] fix: dockerfile @Pencil-Yao 

##### [Optim]

[optim] optim error msg @JLerxky 

[optim] optim: raft cross chain proof verify @Pencil-Yao 

##### [Chore]

[chore] chore: add dependence for evm @Pencil-Yao

## 历史版本

### 6.6.5

主要更新内容如下：

#### 底链微服务

- 新增storage_opendal微服务
  - 对接多种存储后端
  - 冷热数据分层管理
- 新增executor_noop微服务
  - 适用只存证 layer 1 
- controller_crdt调研
  - crdt_macro
- consensus_raft
  - fix: get term error & fix: cover conflict entry
  - recommit entry && transfer leader timeout depend on block interval
- controller
  - optimize get_proposal
  - optimize finalize_block
  - optimize log
- cloud-config
  - 结合kustomize
- cldi
  - 在node status里增加init block number
- 其他
  - 修复共识与controller高度不一致的问题
  - Jaeger 使用远程采样配置
  - panic hook修复
  - 优化日志

#### 云原生

- cita-node-operator 同步上游

#### 集成测试

- 统计同步时间
- 添加namespace环境变量

#### Controller

##### [Feature]

[feat] add init height in node status @rink1969

##### [Fix]

[fix] fix skip reenter error when checking executor @JLerxky

[fix] fix update sc @JLerxky

##### [Optim]

[optim] optim: skip and log ReenterBlock @JLerxky

[optim] update auth and pool after block is persisted @JLerxky

[optim] optim: log @Jayanring

[optim] move update systemconfig after exec block; ignore update if empty blo… @rink1969

[optim] update auth even empty block @rink1969

[optim] optimize get_proposal @rink1969

##### [Chore]

[chore] update keepalive config @rink1969

[chore] replace panic hook @Jayanring

[chore] chore: debug print error @Jayanring

#### network_zenoh

##### [Chore]

[chore] replace panic hook @Jayanring]

#### consensus_overlord

##### [Chore]

[chore] replace panic hook @Jayanring

[chore] chore: debug print error @Jayanring

#### consensus_raft

##### [Feature]

[feat] recommit entry && transfer leader timeout depend on block interval  @Jayanring

##### [Fix]

[fix] fix: get term error & fix: cover conflict entry @Jayanring

[fix] replace panic hook @Jayanring

[fix] chore: debug print error @Jayanring

#### executor_evm

##### [Chore]

[chore] replace panic hook @Jayanring

#### crypto_eth

##### [Chore]

[chore] replace panic hook with common-rs @rink1969

[chore] get inner err info @rink1969

#### crypto_sm

##### [Chore]

[chore] replace panic hook with common-rs @rink1969

#### storage_rocksdb

##### [Optim]

[optim] optim: not store global hash @Jayanring

##### [Chore]

[chore] replace panic hook @Jayanring

[chore] chore: debug print error @Jayanring

#### cita_cloud_proto

##### [Feature]

[feat] add init height in node status @rink1969

#### cloud-common-rs

##### [Feature]

[feat] add jaeger sampler @Jayanring

[feat] add init height in node status @rink1969

[feat] add panic hook @rink1969

##### [Fix]

[fix] fix: get local offset @Jayanring

##### [Optim]

[optim] optim wal clear_file @JLerxky

#### cloud-config

##### [Feature]

[feat] gen kustomization @rink1969

##### [Fix]

[fix] fix debug @rink1969

##### [Chore]

[chore] remove /etc/timezone @rink1969

#### cloud-cli

##### [Feature]

[feat] add init height in node status @rink1969

### v6.6.4

主要更新内容如下：
#### 底链微服务
- bft微服务
  - 废弃该共识
- raft微服务
  - 修复consensus重启后raft会卡住的问题
- controller微服务
  - 添加danger模式
  - commit block耗时长的问题（wal的IO操作引起的）
  - check_proposal添加检查
  - get_node_status优化，整合network信息
- network微服务
  - 升级依赖的zenoh库
  - 修复network zid panic的问题
- cloud-config
  - 增加关闭健康检查的选项
  - 为链节点微服务添加work node时区映射
- cache组件
  - 输出性能测试报告
- cldi
  - 修复release问题
- console/provider组件
  - console组件与rivspace联调中
  - provider组件开发中
- 其他
  - 优化微服务输出的日志，让定位问题更加方便
  - 更新微服务镜像中的grpc-health-check至最新版本
  - 修复u64::from_be_bytes函数调用相关问题
  - 修复微服务帮助信息没有version的问题
  - 集成测试混沌测试用例加强
  
#### 云原生
- cita-node-operator支持新的备份恢复CRD(Duplicate和Recover)
- cita-node-operator与开源版operator相关问题修复与联调

#### SLA测试
- 正在迁移SLA环境

#### Controller
##### [Feature]
[feat] add danger mode for bft to overlord @rink1969

[feat] add is_danger in node_status@rink1969

[feat] tracing @JLerxky
##### [Fix]
[fix] Validator address @Jayanring

[fix] proposal missing pre_proof @Jayanring

[fix] update health probe binary @ rink1969

[fix] check timestamp @ rink1969

[fix] clap help info @Jayanring
##### [Optim]
[optim] process_network_msg log print @Jayanring

[optim] Async wal @JLerxky

[optim] check_proposal @Jayanring
##### [Chore]
[chore] check_proposal add warn @JLerxky
#### network_zenoh
##### [Feature]
[feat] use tracing @JLerxky
##### [Fix]
[fix] domain convert to zenoh_id inconsistent @JLerxky

[fix] reconnect if disconnected @JLerxky

[fix] set unicast max_links 4 @JLerxky

[fix] clap help info @JLerxky
##### [Optim]
[optim] health_check_msg and zenoh_id @JLerxky
##### [Chore]
[chore] Upgrade dependencies @JLerxky

[chore] update zenoh @JLerxky

[chore] update Dockerfile @rink1969
#### consensus_raft
##### [Fix]
[fix] upgrade grpc probe @rink1969

[fix] clap help info @rink1969
#### executor_evm
##### [Refactor]
[refactor] use tracing @rink1969
##### [Fix]
[fix] tracer init @JLerxky

[fix] upgrade grpc probe @rink1969

[fix] clap help info @Jayanring
#### crypto_eth
##### [Feat]
[feat] trace  @Jayanring
##### [Fix]
[fix] upgrade grpc probe @rink1969

[fix] clap help info @Jayanring
#### crypto_sm
##### [Fix]
[fix] Tracing @Jayanring

[fix] upgrade grpc probe @rink1969

[fix] clap help info @Jayanring
##### [Chore]
[chore] update deps @Jayanring
#### storage_rocksdb
##### [Feature]
[feat] trace @Jayanring
##### [Fix]
[fix] upgrade grpc probe @rink1969

[fix] clap help info @Jayanring
#### cita_cloud_proto
##### [Feature]
[feat] add is_danger in node status @rink1969
#### cloud-common-rs
##### [Feature]
[feat] Async wal @JLerxky

[feat] Tracer  @JLerxky 
##### [Refactor]
[refactor] metrics @Jayanring
#### cloud-config
##### [Fix]
[fix] Host aliases   @JLerxky 

[fix] search @JLerxky 

[fix] add disable-metrics in env-dev and env-k8s @Jayanring
##### [Feature]
[feat] tracing @JLerxky 

[feat] Add is danger  @rink1969
##### [Chore]
[chore] rm node-log @JLerxky 

[chore] mnt timezone file @Jayanring
#### cloud-cli
##### [Optim]
[optim] node_status @Jayanring
##### [Feature]
[feat] add lib @JLerxky 
##### [Chore]
[chore] remove bft @Jayanring

#### 兼容性
1. 与上一个版本数据兼容。
2. 因为网络微服务实现升级，不能与上个版本混用。

#### 已知问题
无

### v6.6.3

主要更新内容如下：
#### 底链微服务
- controller微服务
  - GetNodeStatus接口替换GetVersion/GetPeerCount/GetPeersInfo接口
- raft微服务
  - raft节点switch over后共识panic问题修复
- network微服务
  - network 添加 connected info
- cldi
  - cldi命令超时问题修复
  - cldi admin update-validators help命令添加
  - cldi bench send问题修复
- cloud-config
  - 修复import-account没有生成node_address
- 公共服务
  - 拆分cita_cloud_proto为proto文件和rust代码
  - rust代码部分与cloud-util，cloud-code合并到一个仓库（cloud-common-rs）
  - status-code添加至proto文件中
  
#### 云原生
- 支持轻量化部署
- cita-node-operator节点任务串行化
- cita-node-operator支持对象存储备份与恢复

#### SLA测试
- sla运维操作脚本

#### 其他
- 缓存与cldi压力测试对比
- 缓存文档添加

#### Controller
##### [Feature]
[feat] get node status @Jayanring
##### [Chore]
[chore] use common @JLerxky
#### network_zenoh
##### [Feature]
[feat] connected info and update origin by CHECK msg @JLerxky
##### [Chore]
[chore] Use common @JLerxky
#### consensus_bft
##### [Chore]
[chore] use common @JLerxky
#### consensus_raft
##### [Chore]
[chore] use common @JLerxky
#### executor_evm
##### [Chore]
[chore] use common @JLerxky
#### crypto_eth
##### [Chore]
[chore] use common @JLerxky
#### crypto_sm
##### [Chore]
[chore] use common @JLerxky
#### storage_rocksdb
##### [Chore]
[chore] use common @JLerxky
#### cita_cloud_proto
##### [Feature]
[feat] get node status @Jayanring
##### [Chore]
[chore] Simplified @JLerxky
#### cloud-common-rs
##### [Optimize]
[optim] metrics parse request info @Jayanring
#### cloud-config
##### [Optimize]
[optim]  import-account store node_address@Jayanring
#### cloud-cli
##### [Feature]
[feat] get node status@Jayanring

[feat] rpc request timeout@Jayanring
##### [Chore]
[chore] use cloud-common-rs@Jayanring
##### [Fix]
[fix] add Timeout @rink1969  
[fix] fix bench send error@NaughtyDogOfSchrodinger

#### 兼容性
1. 与上一个版本数据兼容。
2. 可以与上个版本混用。

#### 已知问题
1. cloud-config执行import-account命令导入账户，会缺少node_address，导致后续操作失败。影响范围：v6.3.0 - v6.6.2。
2. cloud-cli执行bench send命令进行性能压测时，如果发送交易时间超过100个区块的时间（默认情况下为5分钟），会导致发送交易失败。影响范围：v0.5.2之前的版本。

如果以上问题影响了正常使用，请尽快升级到最新版本。

### v6.6.2

主要更新内容如下：

1. network_zenoh升级上游zenoh版本。
2. network_zenoh在健康检查时增加打印消息时延信息。
3. 重构raft日志模块。
4. 定位raft删除单个节点日志后panic的问题。
5. cloud-op增加消块时是否删除共识日志的开关。
6. cldi文档增加升级报错的说明。
7. executor_evm增加估算quota的接口。
8. cldi增加估算quota的子命令。
9. 增加可靠性测试。
10. 集成测试增加运维操作相关的用例。
11. 优化集成测试的日志输出。
12. 修复operator备份和快照操作中的问题。
13. 实验性增加缓存中间件。

#### Controller

##### [Feature]

[feat] update deps @rink1969

##### [Fix]

[fix] fix wal load @rink1969

#### Network_zenoh

##### [Feature]

[feat] update_zenoh @JLerxky

[feat] print_latency @JLerxky

[feat] update_deps @rink1969

#### Consensus_bft

##### [Feature]

[feat] update_deps @rink1969

#### Consensus_overload

##### [Feature]

[feat] update_deps @rink1969

##### [Chore]

[chore] fix_check_block @rink1969

#### Consensus_raft

##### [Feature]

[feat] refactor storage @Jayanring 

[feat] update deps @rink1969

#### Executor_evm

##### [Feature]

[feat] expose estimate-quota @Jayanring 

[feat] update_deps @rink1969

#### Crypto_eth

##### [Feature]

[feat] update_deps @rink1969

#### Crypto_sm

##### [Feature]

[feat] update_deps @rink1969

#### Storage_rocksdb

##### [Feature]

[feat] update_deps @rink1969

#### Cita_cloud_proto

##### [Feature]

[feat] update deps @rink1969

[feat] estimate-quota @Jayanring

#### Cloud-util

##### [Feature]

[feat] update deps @rink1969

#### Cloud-config

##### [Feature]

[feat] update deps @rink1969

#### Cloud-cli

##### [Feature]

[feat] add_trouble_shooting @rink1969

[feat] update_deps @rink1969

[feat] add-estimate-quota @Jayanring

#### Cloud-op

##### [Feature]

[feat] add clear consensus data option @Jayanring

[feat] state recover add reserve consensus option @Jayanring

[feat] update_deps @rink1969

##### [Fix]

[fix] fix-lockid @Jayanring

#### Integration-test

##### [Feature]

[feat] add operator test @acechef

#### Cita-node-operator

##### [Fix]

[fix] fix backup @acechef

#### 兼容性
1. 与上一个版本数据兼容。
2. 不能与上个版本混用。因为网络微服务升级，新老版本之间网络不通。

#### 已知问题
1. cloud-op消块后没有配套回退system config中的quota_limite。导致消块后重新同步时，如果遇到修改quota_limit的交易，将会无法同步。影响范围：v6.5.0 - v6.6.1
2. 选择consensus_overlord的链重启后重新执行wal中记录的区块时，可能会报错，导致节点无法继续出块。影响范围：共识为overlord的链。

请使用以上版本的链尽快升级到最新版本。


### v6.6.1

主要更新内容如下：
1. 添加grpc metrics功能，能够实时统计Cita-Cloud运行时各微服务间调用的次数、耗时等数据，并在cloud-config中添加开关
2. 优雅退出
3. 将镜像仓库从docker hub迁移到harbor
4. 集成测试改进
    - 添加nft性能测试
    - 添加同步节点升级为共识节点测试
    - 添加raft共识节点更改测试
5. 支持获取指定高度system-config
6. cldi添加解析proof功能
7. 修复consensus_raft更改validators异常
    - 修复移除leader后异常
    - 支持增删多节点
    - 支持一次性更换一批节点
8. 反亲和性
9. 修复共识微服务单独重启问题
10. 守护节点

#### Controller

##### [Feature]

[feat] add metrics @Jayanring

[feat] feat: handle sigterm @JLerxky

[feat] get block add stateroot proof @Jayanring

[feat] get system-config by block-number @Jayanring

##### [Fix]

[fix] fix match_data panic when init @Jayanring

##### [Optimize]

[optim] optim: controller server single restart @Pencil-Yao

[optim] optim: misbehave handling logical @Pencil-Yao

[optim] optim: server not ready info print @Pencil-Yao

[optim] deny change of chain_id and block_limit @Jayanring

##### [Chore]

[chore] fix: change image registry @acechef

[chore] read node_address from file @Jayanring

[chore] fix clippy @Jayanring

#### Network_zenoh

##### [Feature]

[feat] add metrics @Jayanring

[feat] feat: add handle_signals @JLerxky

[feat] feat: send health check msg @JLerxky

##### [Chore]

[chore] fix: change image registry @acechef

[chore] read node_address from file @Jayanring

#### Consensus_bft

##### [Feature]

[feat] add metrics @Jayanring

[feat] feat: add handle_signals @JLerxky

##### [Optimize]
[optim] optim: consensus-server single restart @Pencil-Yao

[optim] optim: use timeout replace interval @Pencil-Yao

##### [Chore]

[chore] fix: change image registry @acechef

[chore] expose LeaderVote @Jayanring

[chore] read node_address from file @Jayanring

#### Consensus_overload

##### [Feature]

[feat] add metrics @Jayanring

[feat] feat: add handle_signals @JLerxky

##### [Optimize]

[optim] optim: consensus-server single restart @Pencil-Yao

##### [Chore]

[chore] fix: change image registry @acechef

[chore] remove unused config items @rink1969

#### Consensus_raft

##### [Feature]

[feat] add metrics @Jayanring

[feat] feat: add handle_signals @JLerxky

[feat] feat: support replacing all validators at one time @Jayanring

##### [Fix]

[fix] fix consensus stop when remove leader @Jayanring

##### [Optimize]

[optim] optim: consensus-server single restart @Pencil-Yao

##### [Chore]

[chore] fix: change image registry @acechef

[chore] read node_address from file @Jayanring

#### Executor_evm

##### [Feature]

[feat] add metrics @Jayanring

[feat] feat: add handle_signals @JLerxky

##### [Fix]

[fix] fix: receipt legacy block hash @Pencil-Yao

[fix] fix: docker image build @Pencil-Yao

[fix] fix: quota_uesd is 0 @Pencil-Yao

##### [Chore]

[chore] fix: change image registry @acechef

[chore] CloudBlock add stateroot @Jayanring

#### Crypto_eth

##### [Feature]

[feat] add metrics @Jayanring

[feat] feat: add handle_signals @JLerxky

##### [Chore]

[chore] fix: change image registry @acechef

#### Crypto_sm

##### [Feature]

[feat] add metrics @Jayanring

[feat] feat: add handle_signals @JLerxky

##### [Chore]

[chore] fix: change image registry @acechef

#### Storage_rocksdb

##### [Feature]

[feat] add metrics @Jayanring

[feat] feat: add handle_signals @JLerxky

##### [Chore]

[chore] fix: change image registry @acechef

[chore] get block add state_root proof @Jayanring

#### Cita_cloud_proto

##### [Feature]

[feat] get system-config by number @Jayanring

[feat] get block add stateroot proof @Jayanring

#### Cloud-util

##### [Feature]

[feat] add metrics @Jayanring

[feat] feat: add handle_signals @JLerxky

##### [Fix]

[fix] fix metrics AlreadyReg @Jayanring

[fix] fix duplicate register when restart @Jayanring

##### [Chore]

[chore] use regex @Jayanring

[chore] chore: modify exporter log info @Jayanring

#### Cloud-config

##### [Feature]

[feat] --init-node add --disable-metrics option @Jayanring

[feat] set pod_anti_affinity @rink1969

##### [Fix]

[fix] fix env_k8s metrics port @Jayanring

[fix] fix crypto log config file @rink1969

[fix] fix create-dev node_address path @Jayanring

[fix] add mount account cm @rink1969

[fix] fix: Permission denied @k4nzdroid

##### [Chore]

[chore] fix: change image registry @acechef

[chore] add common name @rink1969

[chore] use addr in account configmap @rink1969

#### Integration-test

##### [Feature]

[feat] add nft performance test @rink1969

[feat] set sync node to validator @JLerxky

[feat] add only one validator test to raft @JLerxky

##### [Fix]

[fix] fix remove last validator @JLerxky

[fix] fix pip install PyYaml @JLerxky

[fix] fix pyyaml load @JLerxky

[fix] fix: use validator_address @JLerxky

[fix] fix: use exec_bad when expect a bad result @JLerxky

[fix] fix watch sync node @JLerxky

[fix] fix: add kubectl timeout @acechef

[fix] fix get system-config not up to date @JLerxky

##### [Optimize]

[optim] update update-validators @JLerxky

[optim] optim: use retry exec @JLerxky

[optim] check quota used @JLerxky

[optim] change log level @NaughtyDogOfSchrodinger

##### [Chore]

[chore] fix: change image registry @acechef

[chore] use harbor image @JLerxky

[chore] fix: modify update_chain_config.sh @acechef

[chore] keep same args with nft bench @rink1969

[chore] remove set-block-limit.py @JLerxky

[chore] fix: update Dockerfile @acechef

[chore] update height_bad_result @JLerxky

[chore] update chain config @JLerxky

[chore] sort script @JLerxky

[chore] print bad receipt @JLerxky

[chore] update yaml @JLerxky

#### Cloud_cli

##### [Feature]

[feat] get system config by height @Jayanring

[feat] get block add state_root proof @Jayanring

[feat] parse proof @Jayanring

##### [Fix]

[fix] fix: build image error @acechef

##### [Optimize]

[optim] identify validators @Jayanring

[optim] remove admin set-block-kimit @Jayanring

[optim] optim watch @JLerxky

[optim] add get_compact_block_by_number @JLerxky

##### [Chore]

[chore] add non root user @miaojun

[chore] fix: change image registry @acechef

### v6.6.0

主要更新内容如下：
1. 将network_tls微服务废弃，启用network_zenoh微服务
2. consensus_overlord微服务完善
  - bft升级overlord
3. controller微服务
  - 优化同步过程
  - 调整proof not influence proposal_hash
  - 批量广播交易、优化tps
  - 适配network_zenoh
4. grpc封装高层client
  - 实现retry功能、增加keepalive设置
5. call支持指定块高
6. 健康检查
  - network检查一定时间内是否有已连接的节点
7. 依赖库升级
  - tonic升级到0.7、prost升级到0.10
  - 其它依赖升到当前最新版本

#### Controller

##### [Feature]

[feat] set tcp and http2 keep alive  @rink1969

[hard-fork!] feat: proof not influence proposal_hash @Pencil-Yao 

##### [Fix]

[fix] optim: set false sync state when height not grow @Pencil-Yao 

[fix] fix check proposal failed @rink1969

[fix] optim: set true sync state when send sync req @Pencil-Yao 

[fix] mv quota limit form controller config to system config @rink1969

##### [Refactor]

[refactor] switch to retryclient @Jayanring

[refactor] change origin @JLerxky

##### [Chore]

[chore] chore: broadcast increased status @Pencil-Yao

[chore] change multicast to broadcast @JLerxky

[chore] add healthcheck info @ rink1969

[chore] optim: tps! @Pencil-Yao

[chore] chore: upgrade tonic and prost @NaughtyDogOfSchrodinger

#### Network_zenoh

##### [new] Release v6.6.0

#### Consensus_bft

##### [Fix]

[fix] fix check proposal failed @rink1969

[fix] fix wrong lock round @rink1969

[fix] optim: add NewViewRes to optimize newview step @Pencil-Yao

[fix] fix: leader not send newview @Pencil-Yao

##### [Refactor]

[refactor] switch to retryclient @Jayanring

##### [Chore]

[chore] chore: optim newview process @Pencil-Yao

[chore] chore: upgrade tonic and prost @NaughtyDogOfSchrodinger

#### Consensus_overload

##### [new] Release v6.6.0

#### Consensus_raft

##### [Refactor]

[refactor] refactor: handle internal error @NaughtyDogOfSchrodinger

[refactor] switch to retryclient @Jayanring

##### [Chore]

[chore] chore: upgrade tonic and prost @NaughtyDogOfSchrodinger

#### Executor_evm

##### [Feature]

[feat] add height in call request @NaughtyDogOfSchrodinger

##### [Chore]

[chore] chore: upgrade tonic and prost @NaughtyDogOfSchrodinger

#### Crypto_eth

##### [Refactor]

[refactor] update tonic and prost @Jayanring

#### Crypto_sm

##### [Refactor]

[refactor] update tonic and prost @Jayanring

#### Storage_rocksdb

##### [Feature]

[feat] switch to retryclient @Jayanring

##### [Chore]

[chore] chore: upgrade tonic and prost @NaughtyDogOfSchrodinger

#### Network_tls[Archived]

#### Cita_cloud_proto

##### [Feature]

[feat] add height in CallRequest @NaughtyDogOfSchrodinger

##### [Refactor]

[refactor] add client retry @rink1969

[refactor] add retry for NetworkMsgHandlerServiceClient @rink1969

[refactor] connect lazy @rink1969

[refactor] add evm client @rink1969

##### [Chore]

[chore] chore: upgrade tonic and prost @NaughtyDogOfSchrodinger

#### Cloud-util

##### [Refactor]

[refactor] switch to retryclient @Jayanring

##### [Chore]

[chore] chore: upgrade tonic and prost @NaughtyDogOfSchrodinger

#### Cloud-cli

##### [Refactor]

[refactor] refactor: adapt for call with height @NaughtyDogOfSchrodinger

[refactor] remove p2p tls related code @Jayanring

[refactor] display timestamp when get block @Jayanring

[refactor] ignore empty net-info when getting peers-info @JLerxky

##### [Chore]

[chore] chore: upgrae tonic and prost @NaughtyDogOfSchrodinger

##### [Fix]

[fix] fix: watch_cmd is pending in bench send @JLerxky

[fix] fix send default quota @Jayanring

#### Cloud-config

##### [Fix]

[fix] fix default quota limit in init-node @rink1969

##### [Refactor]

[refactor] support network_zenoh @JLerxky

[refactor] change svc clusterip; rename yamls @rink1969

[refactor] mv quota limit from node config to chain config @rink1969

##### [Chore]

[chore] env_dev add overlord config @JLerxky

[chore] judge the is_stdout in the write_log4rs @JLerxky

[chore] remove redundant moduleconfig @Jayanring

[chore] remove p2p tls related code @Jayanring

[chore] add debug info for release @rink1969

[chore] change default consensus from raft to bft @rink1969

[chore] set default output to stdout @rink1969

[chore] remove sh -c in command @rink1969

#### Integration-test

##### [Feature]

[feat] check tx in pool @rink1969

[feat] feat: add build image action @acechef

[feat] enhance chaos test @rink1969

[feat] add sync node @JLerxky

##### [Refactor]

[refactor] change 'timestamp' to 'time' @JLerxky

[refactor] remove tls type test chain @JLerxky

##### [Chore]

[chore] increase delay after stop node @rink1969

[chore] update get_peers_info @JLerxky

[chore] add test chain type @JLerxky

### v6.5.0

主要更新内容如下：
1. 将kms微服务替换成crypto微服务
2. 提供交易池内交易查询的功能
3. cldi 执行 get_block 返回 proof 信息
4. 增加对于单个交易quota限制和block的quota总量限制
5. 微服务常量统一管理
6. controller微服务的health_check增加出块检查
7. 修复WAL文件损坏的问题
8. 修复cloud-config添加新节点后原有节点配置未更新的问题

#### Controller

##### [Feature]

[feat] kms to crypto @rink1969

[feat] health check block number increase @rink1969

[feat] get pool tx @JLerxky

##### [Refactor]

[refactor] add quota limit @NaughtyDogOfSchrodinger

[refactor] fix dead lock @NaughtyDogOfSchrodinger

[refactor] Uniform constant @NaughtyDogOfSchrodinger

##### [Chore]

[chore] optiom quota limit & code type @Pencil-Yao

#### Consensus_bft

##### [Feature]

[feat] kms to crypto @rink1969

#### Storage_rocksdb

##### [Feature]

[feat] kms to crypto @rink1969

[feat] add panic hook and health check load/store @rink1969

#### Cloud-cli

##### [Feature]

[feat] get pool tx @JLerxky

[feat] get block proof  @JLerxky

##### [Refactor]
[refactor] add quota limit  @NaughtyDogOfSchrodinger


#### Cloud-config

##### [Feature]

[feat]  kms to crypto  @rink1969

##### [Fix]

[fix] refresh chain config for old nodes @rink1969


##### [Refactor]

[refactor] adapt for add quota limit @NaughtyDogOfSchrodinger

#### 兼容性问题

1. 升级到该版本会导致crypto服务起不来，需要导出私钥，并修改配置文件

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

#### Consensus_bft

##### [Fix]

[fix] fix commit_block proof inconsistent  @JLerxky

#### Cloud-config

##### [Feature]

[feat] support consensus_overlord  @rink1969

[feat] adapt for libsm @NaughtyDogOfSchrodinger

#### Executor_evm

##### [Optimization]

[optim] add debug info in release  @rink1969  

#### Storage_rocksdb

##### [Feature]

[feat] adapt for libsm @NaughtyDogOfSchrodinger

#### Kms_sm

##### [Fix]

[fix] adapt for fix panic in libsm @NaughtyDogOfSchrodinger

#### Cloud-cli

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

#### Consensus_raft

##### [Feature]

[feat] add health check  @rink1969

#### Consensus_bft

##### [Feature]

[feat] add health check  @rink1969

##### [Optimization]

[optim] optimize handle commit entries @NaughtyDogOfSchrodinger

#### Cloud-config

##### [Feature]

[feat] add livness probe  @rink1969  

[feat] Support gitops  @rink1969  

[feat] add container resource requirements  @rink1969

##### [Fix]

[fix] panic when create-dev  @rink1969  

[fix] update node, protect rewrite file in used  @rink1969

#### Executor_evm

##### [Feature]

[feat] add health check  @rink1969  

[feat]  add base_fee opcode  @JLerxky  

[feat] support switch between full mode and archive mode  @Pencil-Yao

#### Storage_rocksdb

##### [Feature]

[feat] add health check  @rink1969

#### Kms_sm

##### [Feature]

[feat] add health check  @rink1969

#### Network_tls

##### [Feature]

[feat] add health check  @rink1969

##### [Optimization]

[optim] update dependence tokio-rustls and unit-test   @Pencil-Yao

#### Network_p2p

##### [Feature]

[feat] add health check  @rink1969

#### kms_eth

##### [Feature]

[feat] add health check  @rink1969

#### Cloud-cli

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

#### Consensus_raft

##### [Feature]

[feat] update readme  @rink1969 @NaughtyDogOfSchrodinger

[feat] support recover  @Pencil-Yao

#### Consensus_bft

##### [Feature]

[feat] update readme  @NaughtyDogOfSchrodinger

[feat] Support cloud-op  @Pencil-Yao  @JLerxky

##### [Optimization]

[optim] use cloud_util::wal  @JLerxky

[optim] run -c & -l  @JLerxky

#### Cloud-config

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

#### Executor_evm

##### [Feature]

[feat] update readme  @rink1969

[feat] Support cloud-op  @Pencil-Yao  @JLerxky

##### [Optimization]

[optim] run -c & -l  @JLerxky

#### Storage_rocksdb

##### [Feature]

[feat] update readme  @rink1969

[feat] Support cloud-op  @Pencil-Yao  @JLerxky

##### [Optimization]

[optim] run -c & -l  @JLerxky

#### Kms_sm

##### [Feature]

[feat] remove create subcmd; update example and readme  @rink1969

##### [Optimization]

[optim] run -c & -l  @JLerxky

#### Network_tls

##### [Feature]

[feat] update readme  @rink1969

[feat] support hot update + peer-count + origin  @Pencil-Yao  @Jayanring

##### [Optimization]

[optim] run -c  @JLerxky

#### Network_p2p

##### [Feature]

[feat] update readme  @rink1969

##### [Optimization]

[optim] run -c & -l  @JLerxky

#### Kms_eth

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
