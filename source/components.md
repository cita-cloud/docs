# 组件

组件是各个微服务的实现。

仓库名称以微服务名称开头，下划线后接用于标识不同实现的名称。

比如`storage_rocksdb`，是基于`rocksdb`实现的`storage`微服务。

组件分为两类：

* 原厂组件，即`CITA-Cloud`自带的组件。
* 第三方组件。

组件的成熟度：1-5，1表示仅实现必要的功能的最小实现，5表示非常成熟的实现。

组件状态：开发中，维护中，废弃。

## 原厂组件

### controller

### consensus_bft

### consensus_raft

### network_p2p

### network_tls

### network_quic

### kms_eth

### kms_sm

### storage_rocksdb

### storage_sqlite

### storage_tikv

### executor_evm

## 第三方组件

### executor_chaincode

### kms_sdsm

