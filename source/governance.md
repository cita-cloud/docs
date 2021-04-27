# 治理

## 超级管理员

只有超级管理员账户才有权限进行治理操作。

超级管理员可以指定下一任超级管理员。

## 共识节点管理

设置新的共识节点列表，此操作只有超级管理员可以进行。

## 紧急制动

超级管理员在极端情况下的维护手段，开启紧急制动模式后，链上只接收超级管理员发送的交易，其他交易全部拒绝。

主要使用场景为进行一些升级，维护等操作时，不希望有其他交易干扰。

#### 安装工具

目前工具还比较原始，且没有提供编译好的二进制，需要用户自行下载源码编译安装。

参见[仓库地址](https://github.com/cita-cloud/tools)。

#### 开启

在链的配置过程中，会生成存放超级管理员私钥的数据库文件`kms.db`和账户在数据库中的序号`key_id`。

使用跟链匹配的`kms`组件和前述`kms.db`来启动`kms`服务。

通过`-i`传递超级管理员账户的`key_id`。

```
$ ./emergency_brake enable -i 1
...
2021-04-27T10:58:38.048269659+08:00 INFO emergency_brake - waiting for a while...
2021-04-27T10:58:44.061257726+08:00 INFO emergency_brake - OK!
```

查询`SystemConfig`确认`emergencyBrake`变成了`True`。

```
$ ./grpcurl -emit-defaults -plaintext -d '' \
    -proto ~/cita_cloud_proto-master/protos/controller.proto \
    -import-path ~/cita_cloud_proto-master/protos \
    127.0.0.1:50004 \
    controller.RPCService/GetSystemConfig

{
  "version": 0,
  "chainId": "JrC4PnKBvjsRdli28mNtA2jK09dPIiQ0KPVAGktwiX4=",
  "admin": "qGbgIj3glv4VoR8ckFC/MVXnxwY=",
  "blockInterval": 6,
  "validators": [
    "HdJMtGuVtszDfbIkcUh9b+XAy18=",
    "Fw3fXniH6UyXZLnH/WPS0ffGT20=",
    "Q0IjQU2GuSfWeakoQS9/TVXsEfo=",
    "+DEdoQP2CYSEGkb7PlpRvT4H9ko="
  ],
  "emergencyBrake": true,
  "versionPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "chainIdPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "adminPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "blockIntervalPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "validatorsPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "emergencyBrakePreHash": "W1qqQrQxL7IEoVbSeeD5iAXmKpjN+Sk8TV82H878PYg="
}
```

此时，发送普通交易将会返回`forbidden`错误信息。

#### 关闭

准备条件以及参数跟开启时相同。

```
$ ./emergency_brake disable -i 1
...
2021-04-27T10:58:38.048269659+08:00 INFO emergency_brake - waiting for a while...
2021-04-27T10:58:44.061257726+08:00 INFO emergency_brake - OK!
```

查询`SystemConfig`确认`emergencyBrake`变成了`False`。

```
$ ./grpcurl -emit-defaults -plaintext -d '' \
    -proto ~/cita_cloud_proto-master/protos/controller.proto \
    -import-path ~/cita_cloud_proto-master/protos \
    127.0.0.1:50004 \
    controller.RPCService/GetSystemConfig

{
  "version": 0,
  "chainId": "JrC4PnKBvjsRdli28mNtA2jK09dPIiQ0KPVAGktwiX4=",
  "admin": "qGbgIj3glv4VoR8ckFC/MVXnxwY=",
  "blockInterval": 6,
  "validators": [
    "HdJMtGuVtszDfbIkcUh9b+XAy18=",
    "Fw3fXniH6UyXZLnH/WPS0ffGT20=",
    "Q0IjQU2GuSfWeakoQS9/TVXsEfo=",
    "+DEdoQP2CYSEGkb7PlpRvT4H9ko="
  ],
  "emergencyBrake": false,
  "versionPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "chainIdPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "adminPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "blockIntervalPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "validatorsPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "emergencyBrakePreHash": "F0nyI2Tqr/yVbw5b6mAwc38mRp3zAotrq+Em+MO+6d8="
}
```

此时，就可以正常发送普通交易了。
