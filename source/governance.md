# 治理
本章节的操作需要超级管理员权限，且都非常危险，请谨慎操作。

## 安装工具

目前工具还比较原始，且没有提供编译好的二进制，需要用户自行下载源码编译安装。

参见[仓库地址](https://github.com/cita-cloud/tools)。

## 超级管理员

超级管理员可以指定下一任超级管理员。

操作示例：

在链的配置过程中，会生成存放超级管理员私钥的数据库文件`kms.db`和账户在数据库中的序号`key_id`。

使用跟链匹配的`kms`组件和前述`kms.db`来启动`kms`服务。

通过`-i`传递超级管理员账户的`key_id`，`-a`参数传递新的`super admin`账户地址。

```
$ ./update_admin update -a 0x380fc667f4ccd2e91366331f76d4030d27427185 -i 1 -c `minikube ip`:30004
2021-04-28T16:06:16.609927914+08:00 INFO update_admin - waiting for a while...
2021-04-28T16:06:25.677037921+08:00 INFO update_admin - OK!
```

查询`SystemConfig`确认`admin`变成了新的账户地址。

```
$ ./grpcurl -emit-defaults -plaintext -d '' \
    -proto ~/cita_cloud_proto-master/protos/controller.proto \
    -import-path ~/cita_cloud_proto-master/protos \
    `minikube ip`:30004 \
    controller.RPCService/GetSystemConfig
{
  ...
  "admin": "OA/GZ/TM0ukTZjMfdtQDDSdCcYU=",
  ...
  "adminPreHash": "2bq+B1JoDUUFggNM7tjhFI/Y6MQQIGwS4EkaoiG9a6E=",
  ...
}
$ echo OA/GZ/TM0ukTZjMfdtQDDSdCcYU= | base64 -d | hexdump -C
00000000  38 0f c6 67 f4 cc d2 e9  13 66 33 1f 76 d4 03 0d  |8..g.....f3.v...|
00000010  27 42 71 85                                       |'Bq.|
00000014
```

## 共识节点管理

设置新的共识节点列表。

操作示例：

在链的配置过程中，会生成存放超级管理员私钥的数据库文件`kms.db`和账户在数据库中的序号`key_id`。

使用跟链匹配的`kms`组件和前述`kms.db`来启动`kms`服务。

通过`-i`传递超级管理员账户的`key_id`，`-v`参数传递新的共识节点列表，用逗号隔开的一组账户地址。

首先查看当前的验证人列表:

```
$ ./grpcurl -emit-defaults -plaintext -d '' \
    -proto ~/cita_cloud_proto-master/protos/controller.proto \
    -import-path ~/cita_cloud_proto-master/protos \
    `minikube ip`:30004 \
    controller.RPCService/GetSystemConfig
{
  ...
  "validators": [
    "G9qFsSUTToPVoFgkwkoWhXU02X4=",
    "jQbZ2EUmojWMbMnu7t8m+ImhtTw=",
    "cp68Y+9FPCRkRqU/hAuwAuy0KjA=",
    "EknAFRpaXJeTdza63Qe6eIA4Vvw="
  ],
  ...
  "validatorsPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  ...
}
```

设置新的共识节点列表为`0x9b87c3739321b4753aee1ef012d4ec5d7b26dd4d,0x2a783705d63870f83d22520d93bee754e32f04c5,0xffe7019074ddc13d3eab1454fa3445458dc0f0d0`。

```
$ ./update_validators update -i 1 -c `minikube ip`:30004 -v 0x9b87c3739321b4753aee1ef012d4ec5d7b26dd4d,0x2a783705d63870f83d22520d93bee754e32f04c5,0xffe7019074ddc13d3eab1454fa3445458dc0f0d0
2021-04-29T12:01:00.012615335+08:00 INFO update_validators - waiting for a while...
2021-04-29T12:01:15.036363788+08:00 INFO update_validators - OK!
```

查询`SystemConfig`确认`validators`变成了新的一组账户地址。

```
$ ./grpcurl -emit-defaults -plaintext -d '' \
    -proto ~/cita_cloud_proto-master/protos/controller.proto \
    -import-path ~/cita_cloud_proto-master/protos \
    `minikube ip`:30004 \
    controller.RPCService/GetSystemConfig
{
  ...
  "validators": [
    "m4fDc5MhtHU67h7wEtTsXXsm3U0=",
    "Kng3BdY4cPg9IlINk77nVOMvBMU=",
    "/+cBkHTdwT0+qxRU+jRFRY3A8NA="
  ],
  ...
  "validatorsPreHash": "odfZw+wlze5TaSAdETqrdTClDIeLUz0ahoPn6ewLLN8=",
  ...
}

$ echo m4fDc5MhtHU67h7wEtTsXXsm3U0= | base64 -d | hexdump -C
00000000  9b 87 c3 73 93 21 b4 75  3a ee 1e f0 12 d4 ec 5d  |...s.!.u:......]|
00000010  7b 26 dd 4d                                       |{&.M|
00000014
$ echo Kng3BdY4cPg9IlINk77nVOMvBMU= | base64 -d | hexdump -C
00000000  2a 78 37 05 d6 38 70 f8  3d 22 52 0d 93 be e7 54  |*x7..8p.="R....T|
00000010  e3 2f 04 c5                                       |./..|
00000014
$ echo /+cBkHTdwT0+qxRU+jRFRY3A8NA= | base64 -d | hexdump -C
00000000  ff e7 01 90 74 dd c1 3d  3e ab 14 54 fa 34 45 45  |....t..=>..T.4EE|
00000010  8d c0 f0 d0                                       |....|
00000014
```

共识节点列表变更之后，原有共识节点的共识会停掉，链可能会停止出块。

直到使用新的共节点账户的共识节点启动并完成同步之后才会继续共识并出块。

## 修改出块间隔

设置新的出块间隔。

操作示例：

在链的配置过程中，会生成存放超级管理员私钥的数据库文件`kms.db`和账户在数据库中的序号`key_id`。

使用跟链匹配的`kms`组件和前述`kms.db`来启动`kms`服务。

通过`-i`传递超级管理员账户的`key_id`，`-a`参数传递新的`super admin`账户地址。

首先查看当前的出块间隔：

```
$ ./grpcurl -emit-defaults -plaintext -d '' \
    -proto ~/cita_cloud_proto-master/protos/controller.proto \
    -import-path ~/cita_cloud_proto-master/protos \
    `minikube ip`:30004 \
    controller.RPCService/GetSystemConfig
{
  ...
  "blockInterval": 6,
  ...
  "blockIntervalPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  ...
}
```

修改出块间隔为3秒：

```
$ ./set_block_interval set -i 1 -c `minikube ip`:30004 -b 3
2021-04-29T15:12:19.397725063+08:00 INFO set_block_interval - waiting for a while...
2021-04-29T15:12:28.412780267+08:00 INFO set_block_interval - OK!
```

查询`SystemConfig`确认`blockInterval`变成了新的值。

```
$ ./grpcurl -emit-defaults -plaintext -d '' \
    -proto ~/cita_cloud_proto-master/protos/controller.proto \
    -import-path ~/cita_cloud_proto-master/protos \
    `minikube ip`:30004 \
    controller.RPCService/GetSystemConfig
{
  ...
  "blockInterval": 3,
  ...
  "blockIntervalPreHash": "OT4KJp/CcUb+d3KQJK8VYHZvVyL0RaGUR4mWE96s5+U=",
  ...
}
```

## 紧急制动

超级管理员在极端情况下的维护手段，开启紧急制动模式后，链上只接收超级管理员发送的交易，其他交易全部拒绝。

主要使用场景为进行一些升级，维护等操作时，不希望有其他交易干扰。

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
    `minikube ip`:30004 \
    controller.RPCService/GetSystemConfig
{
  ...
  "emergencyBrake": true,
  ...
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
    `minikube ip`:30004 \
    controller.RPCService/GetSystemConfig
{
  ...
  "emergencyBrake": false,
  ...
  "emergencyBrakePreHash": "F0nyI2Tqr/yVbw5b6mAwc38mRp3zAotrq+Em+MO+6d8="
}
```

此时，就可以正常发送普通交易了。
