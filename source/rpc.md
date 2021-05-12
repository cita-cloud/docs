# RPC列表

### GetPeerCount

当前节点连接数。

* 参数: 

无。

* 返回值

`uint64` - 本节点连接节点个数。

* 示例

```
$ ./grpcurl -emit-defaults -plaintext -d '' \
    -proto ~/cita_cloud_proto-5.0.0/protos/controller.proto \
    -import-path ~/cita_cloud_proto-5.0.0/protos \
    `minikube ip`:30004 controller.RPCService/GetPeerCount
{
  "peerCount": "2"
}
```

### GetBlockNumber

返回当前块高度。

* 参数

`bool` - `true` 表示获取`Pending`状态的块高；`false`表示获取`latest`状态的块高。

* 返回值

`uint64` - 当前块高度。

* 示例

```
$ ./grpcurl -emit-defaults -plaintext -d '{"flag": false}' \
    -proto ~/cita_cloud_proto-5.0.0/protos/controller.proto \
    -import-path ~/cita_cloud_proto-5.0.0/protos \
    `minikube ip`:30004 controller.RPCService/GetBlockNumber
{
  "blockNumber": "320"
}
```


### GetTransaction

根据交易哈希查询交易。

* 参数

`bytes` - 交易哈希值。

* 返回值

`RawTransaction` - 交易结构体。

* 示例

```
$ ./grpcurl -emit-defaults -plaintext -d '{"hash": "n7T5mKGARwowz5DFLV5tpanV+XCxnQ4P4YAuSjDTmdA="}' \
    -proto ~/cita_cloud_proto-5.0.0/protos/controller.proto \
    -import-path ~/cita_cloud_proto-5.0.0/protos \
    `minikube ip`:30004 controller.RPCService/GetTransaction
{
  "normalTx": {
    "transaction": {
      "version": 0,
      "to": "AQEBAQEBAQEBAQEBAQEBAQEBAQEB",
      "nonce": "test",
      "quota": "300000",
      "validUntilBlock": "249",
      "data": "QKarrbmLQYf9k279T9lf0NsMpdDNO3Q7N29j6sofdSM=",
      "value": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=",
      "chainId": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAE="
    },
    "transactionHash": "kN+0q0M5vGiwOFBVE7FgCEQThkQ+BlaPFpOmrgwPpQs=",
    "witness": {
      "signature": "sQ0XtpH3n0On2RDQZxzz/pLAIon0O8+iMqnkz1pCqbzftwka0rcVlcmq+w8iKGnFi8WiBjDQLjScwokdpb+Q9q5v9f2ltUTNtXO1M6Gn9uZDD+/thY57zec+rNDOpyD2yh4eeh8J/EXcad8SS98PTzx8MrNbBYH2x50bPDcNu4I=",
      "sender": "Hx2Xykv9907bllhHBXjAsAr+dbY="
    }
  }
}
```

### GetSystemConfig

查询链上系统配置数据。

* 参数

无。

* 返回值

`SystemConfig` - 系统配置结构体。

* 示例

```
$ ./grpcurl -emit-defaults -plaintext -d '' \
    -proto ~/cita_cloud_proto-5.0.0/protos/controller.proto \
    -import-path ~/cita_cloud_proto-5.0.0/protos \
    `minikube ip`:30004 controller.RPCService/GetSystemConfig
{
  "version": 0,
  "chainId": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAE=",
  "admin": "CU1ivX7bcvYbAEKIbHfPDhR/bEI=",
  "blockInterval": 6,
  "validators": [
    "eO1A2A+j/eHvUnvLAny4ycq/i5U=",
    "78dGY0xDCXTQq1dmcRFmUdBdpek=",
    "FqpkYXH8c3JMWST4/kKMLpGBEMk="
  ],
  "versionPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "chainIdPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "adminPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "blockIntervalPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "validatorsPreHash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"
}
```

### GetVersion

获取当前软件的版本号。

* 参数

无。

* 返回值

`String` - 软件版本号。

* 示例

```
$ ./grpcurl -emit-defaults -plaintext -d '' \
    -proto ~/cita_cloud_proto-5.0.0/protos/controller.proto \
    -import-path ~/cita_cloud_proto-5.0.0/protos \
    `minikube ip`:30004 controller.RPCService/GetVersion
{
  "version": "4.0.0"
}
```

### GetBlockHash

获取指定块高的块哈希值。

* 参数

`uint64` - 块高度。

* 返回值

`bytes` - 块哈希值。

* 示例

```
$ ./grpcurl -emit-defaults -plaintext -d '{"block_number": 10}' \
    -proto ~/cita_cloud_proto-5.0.0/protos/controller.proto \
    -import-path ~/cita_cloud_proto-5.0.0/protos \
    `minikube ip`:30004 controller.RPCService/GetBlockHash
{
  "hash": "zt8g46kl/bZraBM2nb+6Jao7bu0IAUb83U+IPsgs3Hk="
}
```

### GetTransactionBlockNumber

获取指定交易所在的块高度。

* 参数

`bytes` - 交易哈希值。

* 返回值

`uint64` - 块高度。

* 示例

```
$ ./grpcurl -emit-defaults -plaintext -d '{"hash": "fJkDwCaMi7mOnHTVQ/IcGNr83aoUxnj5kAkDDhRkya0="}' \
   -proto ~/cita_cloud_proto-5.0.0/protos/controller.proto \
   -import-path ~/cita_cloud_proto-5.0.0/protos \
   `minikube ip`:30004 controller.RPCService/GetTransactionBlockNumber
{
  "blockNumber": "22"
}
```

### GetTransactionIndex

获取指定交易在块中的序号。

* 参数

`bytes` - 交易哈希值。

* 返回值

`uint64` - 序号。

* 示例

```
$ ./grpcurl -emit-defaults -plaintext -d '{"hash": "fJkDwCaMi7mOnHTVQ/IcGNr83aoUxnj5kAkDDhRkya0="}' \
    -proto ~/cita_cloud_proto-5.0.0/protos/controller.proto \
    -import-path ~/cita_cloud_proto-5.0.0/protos \
    `minikube ip`:30004 controller.RPCService/GetTransactionIndex
{
  "txIndex": "2"
}
```

### GetBlockByHash

根据块哈希值查询块。

* 参数

`bytes` - 块哈希值。

* 返回值

`CompactBlock` - 块结构体。

* 示例

```
$ ./grpcurl -emit-defaults -plaintext -d '{"hash": "CeTdeke5A8WeTexqWBxl5elFb8vaHFkLYcTkPo5JAMs="}' \
    -proto ~/cita_cloud_proto-5.0.0/protos/controller.proto \
    -import-path ~/cita_cloud_proto-5.0.0/protos \
    `minikube ip`:30004 controller.RPCService/GetBlockByHash
{
  "version": 0,
  "header": {
    "prevhash": "6r28BwIkV6qTbLD9//WKcGWK3zZNbarD4cVIzSy+M2E=",
    "timestamp": "1616058361343",
    "height": "10",
    "transactionsRoot": "GrIdg1XPoX+OYRlIMegajyK+yMco/vt0ftA161CCqis=",
    "proposer": "eO1A2A+j/eHvUnvLAny4ycq/i5U="
  },
  "body": {
    "txHashes": [
      
    ]
  }
}
```

### GetBlockByNumber

根据块高度查询块。

* 参数

`uint64` - 块高度。

* 返回值

`CompactBlock` - 块结构体。

* 示例

```
$ ./grpcurl -emit-defaults -plaintext -d '{"block_number": 10}' \
    -proto ~/cita_cloud_proto-5.0.0/protos/controller.proto \
    -import-path ~/cita_cloud_proto-5.0.0/protos \
    `minikube ip`:30004 controller.RPCService/GetBlockByNumber
{
  "version": 0,
  "header": {
    "prevhash": "6r28BwIkV6qTbLD9//WKcGWK3zZNbarD4cVIzSy+M2E=",
    "timestamp": "1616058361343",
    "height": "10",
    "transactionsRoot": "GrIdg1XPoX+OYRlIMegajyK+yMco/vt0ftA161CCqis=",
    "proposer": "eO1A2A+j/eHvUnvLAny4ycq/i5U="
  },
  "body": {
    "txHashes": [
      
    ]
  }
}
```

### SendRawTransaction

发送交易。

* 参数

`RawTransaction` - 交易结构体。

* 返回值

`bytes` - 交易哈希值。
