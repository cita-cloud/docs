# 版本发布
## 最新版本

### v6.3.2

本次版本更新对稳定性和运维作了优化，稳定性方面链能够在高负载的情况下更稳定的运行，主要更新内容如下：

1. 优化了 `consensus_bft` 由于网络时延问题导致的无法正常出块的问题；
2. `controller` 具备 WAL 的能力，可以处理进程突然被杀掉而引发的存储问题。
3. 升级的微服务添加了 apache license

<details>

<summary>修改详情</summary>

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

</details>

## 历史版本

### v6.3.0
[版本发布说明](https://github.com/cita-cloud/operator/releases/tag/v6.2.0)

### v6.2.0
[版本发布说明](https://github.com/cita-cloud/operator/releases/tag/v6.2.0)

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
