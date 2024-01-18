# RPC

`CITA-Cloud`为用户提供功能丰富的`RPC`接口，并在不断更新完善中。这些`RPC`接口主要分为两部分，分别由`Controller`和`Executor`提供，`Controller`主要提供区块链信息数据查询、发送交易、添加节点相关接口，`Executor`主要提供合约、账户、交易执行相关接口。

示例中使用`grpcurl`调用这些接口，`bytes`类型的参数需要传入`base64`编码后的值，返回结果中的`bytes`类型也是`base64`编码后的值。

## grpc reflection

`Controller`和`Executor`两个微服务都支持了`reflection`（反射），在使用时可以查询服务提供的接口和消息的结构，依此构造`grpc`请求，而不用指定`proto`文件。示例如下：

```
# 查看提供的服务
$ grpcurl -plaintext 127.0.0.1:50004 list
controller.Consensus2ControllerService
controller.RPCService
evm.RPCService
executor.ExecutorService
grpc.reflection.v1alpha.ServerReflection

# 查看服务中的接口
$ grpcurl -plaintext 127.0.0.1:50004 describe controller.RPCService
controller.RPCService is a service:
service RPCService {
  // add new node
  rpc AddNode ( .common.NodeNetInfo ) returns ( .common.StatusCode );
  rpc GetBlockByHash ( .common.Hash ) returns ( .blockchain.CompactBlock );
  rpc GetBlockByNumber ( .controller.BlockNumber ) returns ( .blockchain.CompactBlock );
  rpc GetBlockDetailByNumber ( .controller.BlockNumber ) returns ( .blockchain.Block );
  rpc GetBlockHash ( .controller.BlockNumber ) returns ( .common.Hash );
  // flag means latest or pending.
  // true means pending, false means latest.
  rpc GetBlockNumber ( .controller.Flag ) returns ( .controller.BlockNumber );
  rpc GetCrossChainProof ( .common.Hash ) returns ( .controller.CrossChainProof );
  rpc GetHeightByHash ( .common.Hash ) returns ( .controller.BlockNumber );
  rpc GetNodeStatus ( .common.Empty ) returns ( .common.NodeStatus );
  rpc GetProofByNumber ( .controller.BlockNumber ) returns ( .common.Proof );
  rpc GetStateRootByNumber ( .controller.BlockNumber ) returns ( .common.StateRoot );
  rpc GetSystemConfig ( .common.Empty ) returns ( .controller.SystemConfig );
  rpc GetSystemConfigByNumber ( .controller.BlockNumber ) returns ( .controller.SystemConfig );
  rpc GetTransaction ( .common.Hash ) returns ( .blockchain.RawTransaction );
  rpc GetTransactionBlockNumber ( .common.Hash ) returns ( .controller.BlockNumber );
  rpc GetTransactionIndex ( .common.Hash ) returns ( .controller.TransactionIndex );
  rpc SendRawTransaction ( .blockchain.RawTransaction ) returns ( .common.Hash );
  rpc SendRawTransactions ( .blockchain.RawTransactions ) returns ( .common.Hashes );
}

# 以GetBlockByNumber接口为例，查看它的参数BlockNumber的结构
$ grpcurl -plaintext 127.0.0.1:50004 describe .controller.BlockNumber
controller.BlockNumber is a message:
message BlockNumber {
  uint64 block_number = 1;
}

# 依此构造grpc请求，不用指定proto文件
$ grpcurl -emit-defaults -plaintext -d '{"block_number": "100"}' 127.0.0.1:50004 controller.RPCService/GetBlockByNumber
{
  "version": 0,
  "header": {
    "prevhash": "ShGkXUKqJ9HZPmQo5Z94o7bPhWeBXNoE7AYr0I8m/6w=",
    "timestamp": "1687337523507",
    "height": "100",
    "transactionsRoot": "GrIdg1XPoX+OYRlIMegajyK+yMco/vt0ftA161CCqis=",
    "proposer": "x+nzQBPukPmVH7VqMNFqh+F03Cg="
  },
  "body": {
    "txHashes": [
      
    ]
  }
}
```


## Controller RPC

### GetBlockNumber

查询当前区块高度

* 接口
  
  rpc GetBlockNumber(Flag) returns (BlockNumber);

* 参数

  `bool flag`: `true`表示获取`Pending`状态的块高；`false`表示获取`latest`状态的块高

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"flag": "false"}' \
127.0.0.1:50004 controller.RPCService/GetBlockNumber
{
  "blockNumber": "1632"
}
```

* * *

### GetNodeStatus

查询当前节点状态

* 接口
  
  rpc GetNodeStatus(common.Empty) returns (common.NodeStatus);

* 参数

  无

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '' \
127.0.0.1:50004 controller.RPCService/GetNodeStatus
{
  "isDanger": false,
  "isSync": false,
  "waiting_block": 0,
  "version": "6.6.3",
  "selfStatus": {
    "height": "72",
    "address": "mOhkarWP6rvHflW7WTotEq+jRYM=",
    "nodeNetInfo": null
  },
  "peersCount": "3",
  "peersStatus": [
    {
      "height": "72",
      "address": "OHQt7esHjdYaN++ada8HozVrRq4=",
      "nodeNetInfo": {
        "multiAddress": "/dns4/127.0.0.1/tcp/40002/tls/test-chain-2",
        "origin": "4067926863296040406"
      }
    },
    {
      "height": "72",
      "address": "4t8sZtTXWW171+g/ww+UG4EdWZM=",
      "nodeNetInfo": {
        "multiAddress": "/dns4/127.0.0.1/tcp/40001/tls/test-chain-1",
        "origin": "16347833992547359085"
      }
    },
    {
      "height": "72",
      "address": "+Nh2xYHoOKtJQfCcYj1d+9pUVWs=",
      "nodeNetInfo": {
        "multiAddress": "/dns4/127.0.0.1/tcp/40003/tls/test-chain-3",
        "origin": "17931212507035744427"
      }
    }
  ]
}
```

* * *

### GetSystemConfig / GetSystemConfigByNumber

查询当前系统配置/查询指定高度的系统配置

* 接口
  
  rpc GetSystemConfig(common.Empty) returns (SystemConfig)
  
  rpc GetSystemConfigByNumber(BlockNumber) returns (SystemConfig);

* 参数

  无 / `uint64 block_number`: 指定高度

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"block_number": "100"}' \
127.0.0.1:50004 controller.RPCService/GetSystemConfigByNumber
{
  "version": 0,
  "chainId": "Y1hqPAJV8zfHend/9U8AQLjDiNoE8j7O5r/UlTplErQ=",
  "admin": "eYA2BKam4PwAKR6Lnh7z8grxr1k=",
  "blockInterval": 3,
  "validators": [
    "eYA2BKam4PwAKR6Lnh7z8grxr1k=",
    "6WOv57ByszRv0OzM39kH8JhJQq8=",
    "SnxP0ScLDh7xkW/Ld2ZueTNYMnw=",
    "b0IurgflE6JTuUf/IaIE/k0CF58="
  ],
  "emergencyBrake": false,
  "versionPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "chainIdPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "adminPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "blockIntervalPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "validatorsPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "emergencyBrakePreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "quotaLimit": 1073741824,
  "quotaLimitPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "blockLimit": 100,
  "blockLimitPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"
}
```

* * *

### GetBlockByNumber / GetBlockDetailByNumber

查询指定高度的区块。`GetBlockByNumber`的内容只包括`version`、`header`、`body`（其中是该区块包含的交易哈希），`GetBlockDetailByNumber`能获取区块的更多信息，包括`version`、`header`、`body`（其中是该区块包含的完整交易）

* 接口
  
  rpc GetBlockByNumber(BlockNumber) returns (blockchain.CompactBlock);
  
  rpc GetBlockDetailByNumber(BlockNumber) returns (blockchain.Block);

* 参数

  `uint64 block_number`: 指定高度

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"block_number": "100"}' \
127.0.0.1:50004 controller.RPCService/GetBlockByNumber
{
  "version": 0,
  "header": {
    "prevhash": "jo2yzstN8kK4yUrzGjwewIILMC8qNaKX2QHsYZJ7n20=",
    "timestamp": "1673418837346",
    "height": "100",
    "transactionsRoot": "GrIdg1XPoX+OYRlIMegajyK+yMco/vt0ftA161CCqis=",
    "proposer": "eYA2BKam4PwAKR6Lnh7z8grxr1k="
  },
  "body": {
    "txHashes": [
      
    ]
  }
}

$ grpcurl -emit-defaults -plaintext -d '{"block_number": "75"}' \
127.0.0.1:50004 controller.RPCService/GetBlockDetailByNumber
{
  "version": 0,
  "header": {
    "prevhash": "jo2yzstN8kK4yUrzGjwewIILMC8qNaKX2QHsYZJ7n20=",
    "timestamp": "1673418837346",
    "height": "100",
    "transactionsRoot": "GrIdg1XPoX+OYRlIMegajyK+yMco/vt0ftA161CCqis=",
    "proposer": "eYA2BKam4PwAKR6Lnh7z8grxr1k="
  },
  "body": {
    "body": [
      
    ]
  },
  "proof": "",
  "stateRoot": "VugfFxvMVab/g0XmksD4bltI4BuZbK3AAWIvteNjtCE="
}
```

* * *

### GetBlockHash 

查询指定高度区块的哈希值

* 接口
  
  rpc GetBlockHash(BlockNumber) returns (common.Hash);

* 参数

  `uint64 block_number`: 指定高度

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"block_number": "100"}' \
127.0.0.1:50004 controller.RPCService/GetBlockHash
{
  "hash": "xs6WmcxhL7apHujKixeYw7Yk+B2W3dd9o86OFVavbPQ="
}
```

* * *

### GetStateRootByNumber 

查询指定高度区块的`StateRoot`（状态树的根哈希值）

* 接口
  
  rpc GetStateRootByNumber (BlockNumber) returns (common.StateRoot);

* 参数

  `uint64 block_number`: 指定高度

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"block_number": "100"}' \
127.0.0.1:50004 controller.RPCService/GetStateRootByNumber
{
  "stateRoot": "VugfFxvMVab/g0XmksD4bltI4BuZbK3AAWIvteNjtCE="
}
```

* * *

### GetProofByNumber 

查询指定高度区块的`Proof`（区块合法证明）

* 接口
  
  rpc GetProofByNumber (BlockNumber) returns (common.Proof);

* 参数

  `uint64 block_number`: 指定高度

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"block_number": "10"}' \
127.0.0.1:50004 controller.RPCService/GetProofByNumber
{
  "proof": "CgAAAAAAAAAAAAAAAAAAAAQAAAABQgAAAAAAAAAweDY4ZWYyMzg3OTVhYTVkNjAxMTRiYTgyMTgyYTI1MTU5MGRhODA4MGE4M2Y1MGRjZTg3NzcxNjNlZTI1YmFmMjMDAAAAAAAAAAoAAAAAAAAAAAAAAAAAAAAEAAAAAUIAAAAAAAAAMHg2OGVmMjM4Nzk1YWE1ZDYwMTE0YmE4MjE4MmEyNTE1OTBkYTgwODBhODNmNTBkY2U4Nzc3MTYzZWUyNWJhZjIzgAAAAAAAAAAPrGoG3fM9XdW+EWKyye0aPDd0d6YQoolm1fkp7yYXYyFFjg//gnDWBrseMP2iSsiP/KCFrCuOTgUWv8hPrkLZbG7UtF7JzUS5ywCrsN55sGwK6MJaeQYWZ67yai/yKPwZM6GTIQZmKHkWUfCVqw/NapocKA5Uum6x1IoDNoP99goAAAAAAAAAAAAAAAAAAAAEAAAAAUIAAAAAAAAAMHg2OGVmMjM4Nzk1YWE1ZDYwMTE0YmE4MjE4MmEyNTE1OTBkYTgwODBhODNmNTBkY2U4Nzc3MTYzZWUyNWJhZjIzgAAAAAAAAAAKTsiHpNgw3lX5sf0Tb8c9SJoLHWUky/0vD6dOg2SwxLN8Ecv6VA1C4vIAjnkKuZz8ObSl3bgFw8PB2js81JfMYEGfyOw5OjD9wOBLbib1HonOXv9hS+u9OIw9/BKNsh95RLKgh7GljqKS7imASytRgNNC/Po6dH3Ld1GWaH3uDAoAAAAAAAAAAAAAAAAAAAAEAAAAAUIAAAAAAAAAMHg2OGVmMjM4Nzk1YWE1ZDYwMTE0YmE4MjE4MmEyNTE1OTBkYTgwODBhODNmNTBkY2U4Nzc3MTYzZWUyNWJhZjIzgAAAAAAAAADY7vMi70yCTq1oYT5e9vJO411Db5rWbpO1C485qo+4tYCtEgwehM2jXPM79TUc9+AxMi0EfY4BDh8vyfieiIVoMcmPN9rePduqGOuh5wKzbsMLpv3fO69MLhpjV257si1b4CGWrge5OhPkAWK3J3DiHWQYLVhAV4scesNpEB5QCQ=="
}
```

* * *

### GetHeightByHash 

根据区块哈希值查询对应区块的高度

* 接口
  
  rpc GetHeightByHash(common.Hash) returns (BlockNumber);

* 参数

  `bytes hash`: 区块哈希值

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"hash": "xs6WmcxhL7apHujKixeYw7Yk+B2W3dd9o86OFVavbPQ="}' \
127.0.0.1:50004 controller.RPCService/GetHeightByHash
{
  "blockNumber": "100"
}
```

* * *

### GetBlockByHash 

根据区块哈希值查询对应的区块

* 接口
  
  rpc GetBlockByHash(common.Hash) returns (blockchain.CompactBlock);

* 参数

  `bytes hash`: 区块哈希值

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"hash": "xs6WmcxhL7apHujKixeYw7Yk+B2W3dd9o86OFVavbPQ="}' \
127.0.0.1:50004 controller.RPCService/GetBlockByHash
{
  "version": 0,
  "header": {
    "prevhash": "4H/+Gc9wP2oKWpcXhchu2PucnXzxzTH85XJcd/JeXF8=",
    "timestamp": "1673431776734",
    "height": "100",
    "transactionsRoot": "GrIdg1XPoX+OYRlIMegajyK+yMco/vt0ftA161CCqis=",
    "proposer": "mOhkarWP6rvHflW7WTotEq+jRYM="
  },
  "body": {
    "txHashes": [
      
    ]
  }
}
```

* * *

### SendRawTransaction / SendRawTransactions

发送交易

`RawTransaction`有两种类型：普通交易`UnverifiedTransaction normal_tx`和治理交易`UnverifiedUtxoTransaction utxo_tx`。

* 接口
  
  rpc SendRawTransaction(blockchain.RawTransaction) returns (common.Hash);

* 参数

  ```
  message RawTransaction {
      oneof tx {
          UnverifiedTransaction normal_tx = 1;
          UnverifiedUtxoTransaction utxo_tx = 2;
      }
  }
  ```

  `oneof`类型表示`normal_tx`和`utxo_tx`字段只能有一个有值。
  `RawTransaction`详见[cita_cloud_proto](https://github.com/cita-cloud/cita_cloud_proto/blob/master/protos/blockchain.proto#L75)。

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"normal_tx": {"transaction": { "version": 0, "to": "a2WNQSBYfQH91o2+oFKuwqHBPew=", "nonce": "6797892813804568799", "quota": "200000", "validUntilBlock": "169", "data": "BmYavQ==", "value": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=", "chainId": "Y1hqPAJV8zfHend/9U8AQLjDiNoE8j7O5r/UlTplErQ=" }, "transactionHash": "2+3XKgIA+DFzKULl23LTC6JjBXFjShffE5gznUWbSrw=", "witness": { "signature": "qG8g7+F2BxJt9QKggM/hMOwMNDmM9995kjV3UrtxY4mi4rJZV6s8t9cNgTL0LGzE7gsFqntwnv38uIYcdY3ZyPambVwKOl9K85PjwALji59M9LORT4W6A51eybh1Ag+khGDkf5wnEr71kYvFdH8ROIx/YYtys+k6PjV3vvczLfs=", "sender": "a2WNQSBYfQH91o2+oFKuwqHBPew=" }}}' \
127.0.0.1:50004 controller.RPCService/SendRawTransaction
{
  "hash": "2+3XKgIA+DFzKULl23LTC6JjBXFjShffE5gznUWbSrw="
}
```

* * *

### GetTransaction 

根据交易哈希值查询对应的交易

* 接口
  
  rpc GetTransaction(common.Hash) returns (blockchain.RawTransaction);

* 参数

  `bytes hash`: 交易哈希值

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"hash": "2+3XKgIA+DFzKULl23LTC6JjBXFjShffE5gznUWbSrw="}' \
127.0.0.1:50004 controller.RPCService/GetTransaction
{
  "normalTx": {
    "transaction": {
      "version": 0,
      "to": "a2WNQSBYfQH91o2+oFKuwqHBPew=",
      "nonce": "6797892813804568799",
      "quota": "200000",
      "validUntilBlock": "169",
      "data": "BmYavQ==",
      "value": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=",
      "chainId": "Y1hqPAJV8zfHend/9U8AQLjDiNoE8j7O5r/UlTplErQ="
    },
    "transactionHash": "2+3XKgIA+DFzKULl23LTC6JjBXFjShffE5gznUWbSrw=",
    "witness": {
      "signature": "qG8g7+F2BxJt9QKggM/hMOwMNDmM9995kjV3UrtxY4mi4rJZV6s8t9cNgTL0LGzE7gsFqntwnv38uIYcdY3ZyPambVwKOl9K85PjwALji59M9LORT4W6A51eybh1Ag+khGDkf5wnEr71kYvFdH8ROIx/YYtys+k6PjV3vvczLfs=",
      "sender": "a2WNQSBYfQH91o2+oFKuwqHBPew="
    }
  }
}
```

* * *

### GetTransactionBlockNumber 

根据交易哈希值查询对应交易所在的块高

* 接口
  
  rpc GetTransactionBlockNumber(common.Hash) returns (BlockNumber);

* 参数

  `bytes hash`: 交易哈希值

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"hash": "2+3XKgIA+DFzKULl23LTC6JjBXFjShffE5gznUWbSrw="}' \
127.0.0.1:50004 controller.RPCService/GetTransactionBlockNumber
{
  "blockNumber": "107"
}
```

* * *

### GetTransactionIndex 

根据交易哈希值查询对应交易在所在区块中的序号

* 接口
  
  rpc GetTransactionIndex(common.Hash) returns (TransactionIndex);

* 参数

  `bytes hash`: 交易哈希值

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"hash": "2+3XKgIA+DFzKULl23LTC6JjBXFjShffE5gznUWbSrw="}' \
127.0.0.1:50004 controller.RPCService/GetTransactionIndex
{
  "txIndex": "0"
}
```

* * *

### AddNode 

添加节点

* 接口
  
  rpc AddNode(common.NodeNetInfo) returns (common.StatusCode);

* 参数

  ```
  message NodeNetInfo {
    string multi_address = 1;
    uint64 origin = 2;
  }
  ```

  `multi_address`是添加节点的网络地址信息，`origin`是节点地址的前8个字节转换为64位无符号整型的结果

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{ "multiAddress": "/dns4/127.0.0.1/tcp/40001/tls/test-chain-1", "origin": "16347833992547359085" }' \
127.0.0.1:50004 controller.RPCService/AddNode
{
  "code": 0
}
```

* * *

### GetCrossChainProof 

获取跨链证明

* 接口
  
  rpc GetCrossChainProof(common.Hash) returns (CrossChainProof);

* 参数

  `bytes hash`: 交易哈希值

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"hash": "2+3XKgIA+DFzKULl23LTC6JjBXFjShffE5gznUWbSrw="}' \
127.0.0.1:50004 controller.RPCService/GetCrossChainProof
{
  "version": 0,
  "chainId": "Y1hqPAJV8zfHend/9U8AQLjDiNoE8j7O5r/UlTplErQ=",
  "proposal": {
    "preStateRoot": "z7OLj3I0Wge52dht/Wehgj6L4hbArs+GY9/QsHi8rZI=",
    "proposal": {
      "version": 0,
      "header": {
        "prevhash": "YLJ/O9MiaUoRVB5j1V+4fMgyTvn0MCIWqmB4JiUoGIc=",
        "timestamp": "1687334197360",
        "height": "50",
        "transactionsRoot": "O1JV/1UTJGnyxPrnRbHxx5YajCBlizbCBLwDN9LOOak=",
        "proposer": "x+nzQBPukPmVH7VqMNFqh+F03Cg="
      },
      "body": {
        "txHashes": [
          "do38caZae78ZfQqz0IDHdbG8oeIxrFRylRv3+svh9eg="
        ]
      }
    }
  },
  "receiptProof": {
    "receipt": "+QEqglIIuQEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMDAgKB2jfxxplp7vxl9CrPQgMd1sbyh4jGsVHKVG/f6y+H16A==",
    "receiptProof": "wA==",
    "rootsInfo": {
      "height": "50",
      "stateRoot": "bXg4C9xF4cXv2V8a9YwPgq0WjCdytJkkTFqbuZg6toI=",
      "receiptRoot": "tO1g3jO9tjU/XimtDqtgodzV3V3b2DWpfmumE8m1kI4="
    }
  },
  "proof": "",
  "stateRoot": "2ZVY1e/4V/DQh3a3+ydvI3HDUXqpbKyNMjE9qlGPJgw="
}
```

* * *

## Executor RPC

### GetTransactionReceipt 

根据交易哈希值查询对应交易的收据（执行信息）

* 接口
  
  rpc GetTransactionReceipt(common.Hash) returns (Receipt);

* 参数

  `bytes hash`: 交易哈希值

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"hash": "2+3XKgIA+DFzKULl23LTC6JjBXFjShffE5gznUWbSrw="}' \
127.0.0.1:50002 evm.RPCService/GetTransactionReceipt
{
  "transactionHash": "2+3XKgIA+DFzKULl23LTC6JjBXFjShffE5gznUWbSrw=",
  "transactionIndex": "0",
  "blockHash": "GNtzrxx6h+K0SojFLrd8BsdhQiHCbgNdT4jvr0iKj0A=",
  "blockNumber": "107",
  "cumulativeQuotaUsed": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAUgg=",
  "quotaUsed": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAUgg=",
  "contractAddress": "AAAAAAAAAAAAAAAAAAAAAAAAAAA=",
  "logs": [
    
  ],
  "stateRoot": "bXg4C9xF4cXv2V8a9YwPgq0WjCdytJkkTFqbuZg6toI=",
  "logsBloom": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA==",
  "errorMessage": ""
}
```

* * *

### GetBalance 

查询账户地址余额

* 接口
  
  rpc GetBalance(GetBalanceRequest) returns (Balance);

* 参数

  ```
  message GetBalanceRequest {
    common.Address address = 1;
    BlockNumber block_number = 2;
  }
  ```

  `address`: 账户地址

  `block_number`: 查询的区块高度，如果缺失，默认当前最新高度。


* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"address": "a2WNQSBYfQH91o2+oFKuwqHBPew="}' \
127.0.0.1:50002 evm.RPCService/GetBalance
{
  "value": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA="
}
```

* * *

### GetTransactionCount 

查询账户地址发送的交易数量

* 接口
  
  rpc GetTransactionCount(GetTransactionCountRequest) returns (Nonce);

* 参数

  ```
  message GetTransactionCountRequest {
    common.Address address = 1;
    BlockNumber block_number = 2;
  }
  ```

  `address`: 账户地址

  `block_number`: 查询的区块高度，如果缺失，默认当前最新高度。

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"address": "a2WNQSBYfQH91o2+oFKuwqHBPew="}' \
127.0.0.1:50002 evm.RPCService/GetTransactionCount
{
  "nonce": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAE="
}
```

* * *

### GetCode 

根据合约地址查询对应合约的字节码

* 接口
  
  rpc GetCode(GetCodeRequest) returns (ByteCode);

* 参数

  ```
  message GetCodeRequest {
    common.Address address = 1;
    BlockNumber block_number = 2;
  }
  ```

  `address`: 合约地址

  `block_number`: 查询的区块高度，如果缺失，默认当前最新高度。

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"address": "Req3cucyj4Vkugo9aVQW1VLDpfM="}' \
127.0.0.1:50002 evm.RPCService/GetCode
{
  "byteCode": "YIBgQFJgBDYQYFNXYAA1fAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAkARj/////xaAYwZmGr0UYFhXgGNPK+kfFGCAV4Bj2Cb4jxRglFdbYACA/Vs0gBVgY1dgAID9W1BgamCoVltgQFGAgoFSYCABkVBQYEBRgJEDkPNbNIAVYItXYACA/VtQYJJgrlZbAFs0gBVgn1dgAID9W1BgpmDAVlsAW2AAVIFWW2ABYACAgoJUAZJQUIGQVVBWW2AAgIGQVVBWAKFlYnp6cjBYIPqh0fUde1yisgDg9s3vTy1+RO5oYgnjAL6xFG9A0y3uACk="
}
```

* * *

### GetAbi 

根据合约地址查询保存的合约`Abi`。在此操作之前需要发送一个特殊交易保存合约地址对应的`Abi`。

* 接口
  
  rpc GetAbi(GetAbiRequest) returns (ByteAbi);

* 参数

  ```
  message GetAbiRequest {
    common.Address address = 1;
    BlockNumber block_number = 2;
  }
  ```

  `address`: 合约地址

  `block_number`: 查询的区块高度，如果缺失，默认当前最新高度。

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"address": "Req3cucyj4Vkugo9aVQW1VLDpfM="}' \
127.0.0.1:50002 evm.RPCService/GetAbi
{
  "bytesAbi": "WwoJewoJCWNvbnN0YW50OiB0cnVlLAoJCWlucHV0czogW10sCgkJbmFtZTogY291bnQsCgkJb3V0cHV0czogWwoJCQl7CgkJCQluYW1lOiAsCgkJCQl0eXBlOiB1aW50MjU2CgkJCX0KCQldLAoJCXBheWFibGU6IGZhbHNlLAoJCXN0YXRlTXV0YWJpbGl0eTogdmlldywKCQl0eXBlOiBmdW5jdGlvbgoJfSwKCXsKCQljb25zdGFudDogZmFsc2UsCgkJaW5wdXRzOiBbXSwKCQluYW1lOiBhZGQsCgkJb3V0cHV0czogW10sCgkJcGF5YWJsZTogZmFsc2UsCgkJc3RhdGVNdXRhYmlsaXR5OiBub25wYXlhYmxlLAoJCXR5cGU6IGZ1bmN0aW9uCgl9LAoJewoJCWNvbnN0YW50OiBmYWxzZSwKCQlpbnB1dHM6IFtdLAoJCW5hbWU6IHJlc2V0LAoJCW91dHB1dHM6IFtdLAoJCXBheWFibGU6IGZhbHNlLAoJCXN0YXRlTXV0YWJpbGl0eTogbm9ucGF5YWJsZSwKCQl0eXBlOiBmdW5jdGlvbgoJfQpd"
}
```

* * *

### EstimateQuota 

估算执行交易需要消耗的`quota`

* 接口
  
  rpc EstimateQuota(executor.CallRequest) returns (ByteQuota);

* 参数

  ```
  message CallRequest {
    bytes to = 1;
    bytes from = 2;
    bytes method = 3;
    repeated bytes args = 4;
    uint64 height = 5;
  }
  ```

  `EstimateQuota`接口实际上只需要`to`、`from`和`method`三个参数，其他参数用默认值即可，不影响计算。`from`是交易发送者的地址，用户调用该接口时可以随意指定，交易发送者会影响交易的执行过程。如果是调用合约交易，则需要则`to`是合约地址，`method`是要调用的合约方法；如果是创建合约交易，则`to`是20字节的0，`method`是合约字节码。

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"to": "Req3cucyj4Vkugo9aVQW1VLDpfM=", "from": "a2WNQSBYfQH91o2+oFKuwqHBPew=", "method": "TyvpHw=="}' \
127.0.0.1:50002 evm.RPCService/EstimateQuota
{
  "bytesQuota": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAoRY="
}
```

* * *

### GetReceiptProof

获取交易的`ReceiptProof`

* 接口
  
  rpc GetReceiptProof(common.Hash) returns (ReceiptProof);

* 参数

  `bytes hash`: 交易哈希值

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"hash": "do38caZae78ZfQqz0IDHdbG8oeIxrFRylRv3+svh9eg="}' \
127.0.0.1:50002 evm.RPCService/GetReceiptProof
{
  "receipt": "+QEqglIIuQEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMDAgKB2jfxxplp7vxl9CrPQgMd1sbyh4jGsVHKVG/f6y+H16A==",
  "receiptProof": "wA==",
  "rootsInfo": {
    "height": "50",
    "stateRoot": "bXg4C9xF4cXv2V8a9YwPgq0WjCdytJkkTFqbuZg6toI=",
    "receiptRoot": "tO1g3jO9tjU/XimtDqtgodzV3V3b2DWpfmumE8m1kI4="
  }
}
```

* * *

### GetRootsInfo

获取指定高度的`RootsInfo`

* 接口
  
  rpc GetRootsInfo(BlockNumber) returns (RootsInfo);

* 参数

  `uint64 block_number`: 指定高度

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"block_number": "50"}' \
127.0.0.1:50002 evm.RPCService/GetRootsInfo
{
  "height": "50",
  "stateRoot": "bXg4C9xF4cXv2V8a9YwPgq0WjCdytJkkTFqbuZg6toI=",
  "receiptRoot": "tO1g3jO9tjU/XimtDqtgodzV3V3b2DWpfmumE8m1kI4="
}
```

* * *

### Call 

查询链上数据

* 接口
  
  rpc Call(CallRequest) returns (CallResponse);

* 参数

  ```
  message CallRequest {
    bytes to = 1;
    bytes from = 2;
    bytes method = 3;
    repeated bytes args = 4;
    uint64 height = 5;
  }
  ```

  `to` 合约地址
  
  `from` 交易发送者的地址，用户调用该接口时可以随意指定
  
  `method` 要查询的方法；
  
  `height` 查询该数据在指定高度的值，0(默认值)代表当前高度。

* 示例

```bash
$ grpcurl -emit-defaults -plaintext -d '{"to": "13TPTHVrw0WmEi/XdSVt5Wa4S4E=", "from": "a2WNQSBYfQH91o2+oFKuwqHBPew=", "method": "BmYavQ=="}' \
127.0.0.1:50002 executor.ExecutorService/Call
{
  "value": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAE="
}
```

* * *

