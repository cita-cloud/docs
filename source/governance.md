# 治理
本章节的操作需要超级管理员权限，且都非常危险，请谨慎操作。

超级管理员账户在`运行CITA-Cloud`时指定，详情参见`快速入门`相关章节的内容。

在`cloud-cli`中登录超级管理员账户：
```
$ cldi account login admin
OK, now the default user is `admin`, account addr is 0xae069e1925a1dad2a1f4c7034d87258dfd9b6532
```

## 更新超级管理员

超级管理员可以指定下一任超级管理员。

操作示例：

1. 查看系统配置，确认当前超级管理员账户。
  ```
  $ cldi system-config 
  {
    "admin": "0xae069e1925a1dad2a1f4c7034d87258dfd9b6532",
    "block_interval": 6,
    "chain_id": "0x26b0b83e7281be3b117658b6f2636d0368cad3d74f22243428f5401a4b70897e",
    "validators": [
      "0xc35b3b7437a31b4d0a737041a17a8e181ae25ba5",
      "0xa5e75c8ed90c17d2cd0b637943c7ce83248dbf20",
      "0x32872cec919211f5d144f8464b45140f4a146002",
      "0x790f590a1ea9764bcc26154c3de868ccf7bdcad4"
    ],
    "version": 0
  }
  ```
2. 指定新的超级管理员
  ```
  $ cldi update-admin 0x9817ac046e0b6a2903352d05497564147ddc0a6f 
  tx_hash: 0x1f3b25887bca912b5809e4860cd507574682efe457c01d2d450aa2067c06e826
  ```
3. 等待几秒，待交易上链之后，查看系统配置确认`admin`变成了新的账户地址。
  ```
  $ cldi system-config 
  {
    "admin": "0x9817ac046e0b6a2903352d05497564147ddc0a6f",
    "block_interval": 6,
    "chain_id": "0x26b0b83e7281be3b117658b6f2636d0368cad3d74f22243428f5401a4b70897e",
    "validators": [
      "0xc35b3b7437a31b4d0a737041a17a8e181ae25ba5",
      "0xa5e75c8ed90c17d2cd0b637943c7ce83248dbf20",
      "0x32872cec919211f5d144f8464b45140f4a146002",
      "0x790f590a1ea9764bcc26154c3de868ccf7bdcad4"
    ],
    "version": 0
  }
  ```

## 共识节点管理

设置新的共识节点列表。

操作示例：

1. 查看系统配置，确认当前的验证人列表。
  ```
  $ cldi system-config                                                                         
  {
    "admin": "0xae069e1925a1dad2a1f4c7034d87258dfd9b6532",
    "block_interval": 3,
    "chain_id": "0x26b0b83e7281be3b117658b6f2636d0368cad3d74f22243428f5401a4b70897e",
    "validators": [
      "0x724f28ac8d069f150d541a07235ebc2363ae9b2a",
      "0x1e0427b2a2ba34dceda5788d98f59aef0eb92f8e",
      "0x868a116bd018c3ed0facb4a761829485f03c2f73"
    ],
    "version": 0
  }
  ```
2. 设置新的共识节点列表，新增地址为`0xc35b3b7437a31b4d0a737041a17a8e181ae25ba5`的共识节点。
  ```
  $ cldi update-validators 0x724f28ac8d069f150d541a07235ebc2363ae9b2a 0x1e0427b2a2ba34dceda5788d98f59aef0eb92f8e 0x868a116bd018c3ed0facb4a761829485f03c2f73 0xc35b3b7437a31b4d0a737041a17a8e181ae25ba5
  tx_hash: 0x54f9ebdb3d5e52b80cad92c7551a706a4ae20aecd7cde88c167f8659bec4a4b4
  ```
3. 等待几秒，待交易上链之后，查看系统配置确认`validators`变成了新的一组账户地址。
  ```
  $ cldi system-config
  {
    "admin": "0xae069e1925a1dad2a1f4c7034d87258dfd9b6532",
    "block_interval": 3,
    "chain_id": "0x26b0b83e7281be3b117658b6f2636d0368cad3d74f22243428f5401a4b70897e",
    "validators": [
      "0x724f28ac8d069f150d541a07235ebc2363ae9b2a",
      "0x1e0427b2a2ba34dceda5788d98f59aef0eb92f8e",
      "0x868a116bd018c3ed0facb4a761829485f03c2f73",
      "0xc35b3b7437a31b4d0a737041a17a8e181ae25ba5"
    ],
    "version": 0
  }
  ```

共识节点列表变更之后，若有共识节点被剔除，则被剔除共识节点的共识会停止；若有新的共识节点被添加，则链可能会停止出块，直到新加入的共识节点启动并完成同步之后才会继续共识并出块。

## 修改出块间隔

超级管理员可以设置新的出块间隔。

操作示例：

1. 查看当前的出块间隔。

  ```
  $ cldi system-config
  {
    "admin": "0x9817ac046e0b6a2903352d05497564147ddc0a6f",
    "block_interval": 6,  //出块间隔为6秒
    "chain_id": "0x26b0b83e7281be3b117658b6f2636d0368cad3d74f22243428f5401a4b70897e",
    "validators": [
      "0xc35b3b7437a31b4d0a737041a17a8e181ae25ba5",
      "0xa5e75c8ed90c17d2cd0b637943c7ce83248dbf20",
      "0x32872cec919211f5d144f8464b45140f4a146002",
      "0x790f590a1ea9764bcc26154c3de868ccf7bdcad4"
    ],
    "version": 0
  }
  ```

2. 修改出块间隔为3秒。

  ```
  $ cldi  cldi set-block-interval 3
  tx_hash: 0xa0cad608b1e0245f988d193380a56aea1056bca217f2a73dadb8dcec02e20cb9
  ```

3. 等待几秒，待交易上链之后，查看系统配置确认`blockInterval`变成了新的值。
  ```
  $ cldi system-config
  {
    "admin": "0x9817ac046e0b6a2903352d05497564147ddc0a6f",
    "block_interval": 3,
    "chain_id": "0x26b0b83e7281be3b117658b6f2636d0368cad3d74f22243428f5401a4b70897e",
    "validators": [
      "0xc35b3b7437a31b4d0a737041a17a8e181ae25ba5",
      "0xa5e75c8ed90c17d2cd0b637943c7ce83248dbf20",
      "0x32872cec919211f5d144f8464b45140f4a146002",
      "0x790f590a1ea9764bcc26154c3de868ccf7bdcad4"
    ],
    "version": 0
  }
  ```

## 紧急制动

超级管理员在极端情况下的维护手段，开启紧急制动模式后，链上只接收超级管理员发送的`治理`交易，其他交易全部拒绝。（`治理交易`为本章的管理员操作）

主要使用场景为进行一些升级，维护等操作时，不希望有其他交易干扰。

#### 开启
```
$  cldi emergency-brake on
tx_hash: 0x5b5aaa42b4312fb204a156d279e0f98805e62a98cdf9293c4d5f361fcefc3d88
```

等待几秒，待交易上链之后，将只有超级管理员可以发送治理交易，普通用户和普通交易会返回`forbidden`错误。

例如使用普通用户账户`test`创建合约：
```
$ cldi account login test
OK, now the default user is `test`, account addr is 0x415f568207900b6940477396fcd2c201efe49beb
$ cldi create 0x608060405234801561001057600080fd5b5060f58061001f6000396000f3006080604052600436106053576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806306661abd1460585780634f2be91f146080578063d826f88f146094575b600080fd5b348015606357600080fd5b50606a60a8565b6040518082815260200191505060405180910390f35b348015608b57600080fd5b50609260ae565b005b348015609f57600080fd5b5060a660c0565b005b60005481565b60016000808282540192505081905550565b600080819055505600a165627a7a72305820faa1d1f51d7b5ca2b200e0f6cdef4f2d7e44ee686209e300beb1146f40d32dee0029
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Status { code: InvalidArgument, message: "Expect error: forbidden", metadata: MetadataMap { headers: {"content-type": "application/grpc", "date": "Wed, 21 Jul 2021 09:05:37 GMT"} }, source: None }', src/client.rs:146:14
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

#### 关闭
```
$ cldi emergency-brake off
tx_hash: 0x347cf8a7ef89958e936344dc8318fd0ca7dd5eadaab60df74f094989549b7427
```
等待几秒，待交易上链之后，普通用户就可以再次正常发送交易了。
```
$ cldi account login test
OK, now the default user is `test`, account addr is 0x415f568207900b6940477396fcd2c201efe49beb
$ cldi create 0x608060405234801561001057600080fd5b5060f58061001f6000396000f3006080604052600436106053576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806306661abd1460585780634f2be91f146080578063d826f88f146094575b600080fd5b348015606357600080fd5b50606a60a8565b6040518082815260200191505060405180910390f35b348015608b57600080fd5b50609260ae565b005b348015609f57600080fd5b5060a660c0565b005b60005481565b60016000808282540192505081905550565b600080819055505600a165627a7a72305820faa1d1f51d7b5ca2b200e0f6cdef4f2d7e44ee686209e300beb1146f40d32dee0029
tx_hash: 0x7eeaefdf3d266a2b1a8b6350952b1353b230207bc4c089735a3d2e482dc1f77e
```
