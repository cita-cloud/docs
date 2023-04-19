# 快速入门

本章介绍的是使用`k3s`作为运行环境，快速搭建一条链的操作方法。

关于更加深入的`定制`操作，请参阅`定制`章节。

## 环境准备

### 硬件配置建议

* CPU：4核或以上
* 内存：8GB或以上
* 硬盘：30G或以上

### 软件依赖

#### 操作系统

常见的`Linux`发行版本均可，例如：`CentOS`，`Debian`，`Ubuntu`等等。

#### docker

安装方法参见[官方文档](https://docs.docker.com/engine/install/)。

#### k3s

安装方法参见[Rancher官方文档](https://docs.rancher.cn/docs/k3s/quick-start/_index/#%E5%AE%89%E8%A3%85%E8%84%9A%E6%9C%AC)。

完成之后设置环境变量：
```
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

#### cloud-cli

该工具为`CITA-Cloud`链的命令行客户端，可以方便的对链进行常用的操作。

使用方法参见[文档](https://cita-cloud.github.io/cloud-cli/)。

```
$ wget https://github.com/cita-cloud/cloud-cli/releases/download/v0.5.4/cldi-x86_64-unknown-linux-musl.tar.gz
$ tar zxvf cldi-x86_64-unknown-linux-musl.tar.gz
$ sudo mv ./cldi /usr/local/bin/
$ cldi -h
cldi 0.5.4
Rivtower Technologies <contact@rivtower.com>
The command line interface to interact with CITA-Cloud

Usage: cldi [OPTIONS] [COMMAND]

Commands:
  get          Get data from chain
  send         Send transaction
  call         Call executor
  create       create an EVM contract
  context      Context commands
  account      Account commands
  admin        The admin commands for managing chain
  rpc          Other RPC commands
  ethabi       Ethereum ABI coder.
  bench        Simple benchmarks
  watch        Watch blocks
  completions  Generate completions for current shell. Add the output script to `.profile` or `.bashrc` etc. to make it effective.
  help         Print this message or the help of the given subcommand(s)

Options:
  -c, --context <context>           context setting
  -r <controller-addr>              controller address
  -e <executor-addr>                executor address
  -u <account-name>                 account name
  -p <password>                     password to unlock the account
      --crypto <crypto-type>        The crypto type of the target chain [possible values: SM, ETH]
      --consensus <consensus-type>  The consensus type of the target chain [possible values: BFT, OVERLORD, RAFT]
  -h, --help                        Print help information
  -V, --version                     Print version information
```

## 运行链

### 生成超级管理员账户

使用 `cldi` 创建账户
```
$ cldi account generate -h
generate a new account

USAGE:
    cldi account generate [OPTIONS]

OPTIONS:
        --name <name>             The name for the new generated account, default to account address
    -p, --password <password>     The password to encrypt the account
        --crypto <crypto-type>    The crypto type for the generated account. [default: <current-
                                  context-crypto-type>] [possible values: SM, ETH]
    -h, --help                    Print help information
```

为了演示方便，这里不设置密码，加密算法也使用默认值。

```
$ cldi account generate --name admin
{
  "crypto_type": "SM",
  "address": "0xc8ca9cc77a7f822fdd0baef7a7740f9dba493455",
  "public_key": "0x0d9edfd3889ec752e92fb1aa53fdfc26512c6a0ea39deb12510e7ac4d0915c4d4f0a18a3c1cf2a5950319d429af38b13483e2ccc9bafd698f5ce2c7ef558ddf6",
  "secret_key": "0x50dc0c1655419938d83d924a3c3b4cbbd57de5df901ce4772272445605a52d43"
}
```

### 生成链的配置

设置环境变量：

```bash
# 设置docker镜像仓库
export DOCKER_REGISTRY=docker.io
export DOCKER_REPO=citacloud

# 设置链的版本
export RELEASE_VERSION=v6.6.5

# 设置链的类型和名称
export CHIAN_TYPE=overlord
# export CHIAN_TYPE=raft
export CHAIN_NAME=test-$CHIAN_TYPE

# 设置基础环境的Storage Class，这里使用k3s自带的local-path
export SC=local-path

# 设置链运行的命名空间
export NAME_SPACE=cita
```

生成链的配置文件：

```bash
# 生成初始的4个共识节点配置
# 注意：`--admin`参数必须设置为自己生成的账户地址，此处仅为演示，切勿在正式环境中使用演示值。
docker run -it --rm -v $(pwd):/data -w /data $DOCKER_REGISTRY/$DOCKER_REPO/cloud-config:$RELEASE_VERSION cloud-config create-k8s --chain-name $CHAIN_NAME --admin 0xc8ca9cc77a7f822fdd0baef7a7740f9dba493455 --nodelist localhost:40000:node0:k8s,localhost:40001:node1:k8s,localhost:40002:node2:k8s,localhost:40003:node3:k8s --controller_tag $RELEASE_VERSION --consensus_image consensus_$CHIAN_TYPE --consensus_tag $RELEASE_VERSION --crypto_tag $RELEASE_VERSION --network_tag $RELEASE_VERSION --storage_tag $RELEASE_VERSION --executor_tag $RELEASE_VERSION


# 生成所有节点配置的yaml文件
docker run -it --rm -v $(pwd):/data -w /data $DOCKER_REGISTRY/$DOCKER_REPO/cloud-config:$RELEASE_VERSION cloud-config update-yaml --chain-name $CHAIN_NAME --storage-class $SC --docker-registry $DOCKER_REGISTRY --docker-repo $DOCKER_REPO --domain node0
docker run -it --rm -v $(pwd):/data -w /data $DOCKER_REGISTRY/$DOCKER_REPO/cloud-config:$RELEASE_VERSION cloud-config update-yaml --chain-name $CHAIN_NAME --storage-class $SC --docker-registry $DOCKER_REGISTRY --docker-repo $DOCKER_REPO --domain node1
docker run -it --rm -v $(pwd):/data -w /data $DOCKER_REGISTRY/$DOCKER_REPO/cloud-config:$RELEASE_VERSION cloud-config update-yaml --chain-name $CHAIN_NAME --storage-class $SC --docker-registry $DOCKER_REGISTRY --docker-repo $DOCKER_REPO --domain node2
docker run -it --rm -v $(pwd):/data -w /data $DOCKER_REGISTRY/$DOCKER_REPO/cloud-config:$RELEASE_VERSION cloud-config update-yaml --chain-name $CHAIN_NAME --storage-class $SC --docker-registry $DOCKER_REGISTRY --docker-repo $DOCKER_REPO --domain node3
```

### 启动链

```bash
# 创建命名空间
kubectl create ns $NAME_SPACE

# 部署所有节点
kubectl apply -f $CHAIN_NAME-node0/yamls/ -n $NAME_SPACE
kubectl apply -f $CHAIN_NAME-node1/yamls/ -n $NAME_SPACE
kubectl apply -f $CHAIN_NAME-node2/yamls/ -n $NAME_SPACE
kubectl apply -f $CHAIN_NAME-node3/yamls/ -n $NAME_SPACE
```

### 查看运行情况

```bash
$ kubectl get pod -n $NAME_SPACE
NAME                                                    READY   STATUS    RESTARTS   AGE
test-overlord-node0-0                                   6/6     Running   0          8m3s
test-overlord-node1-0                                   6/6     Running   0          8m3s
test-overlord-node2-0                                   6/6     Running   0          8m3s
test-overlord-node3-0                                   6/6     Running   0          8m3s
```

查看日志：

```bash
$ kubectl logs -f $CHAIN_NAME-node0-0 -c controller -n $NAME_SPACE
2022-11-16T07:47:40.586458114+00:00 INFO controller::chain - finalize_block height: 23363
2022-11-16T07:47:41.615000522+00:00 INFO controller::health_check - healthcheck entry!
2022-11-16T07:47:41.615016195+00:00 INFO controller::health_check - healthcheck: block increase 23359 23363 1668584861615
2022-11-16T07:47:43.072274837+00:00 INFO controller::chain - add_proposal: tx poll len 0
2022-11-16T07:47:43.073818092+00:00 INFO controller::chain - proposal 23364 block_hash 0x98b1ed689464622d6b48ce8aaf26b3429d3e1dbbba9c6699ca2d0ea1d710f66b prevhash 0x6be134f00d78f7129e29b6db9b7a55a387472acc929ec7809645319c5c0455a5
2022-11-16T07:47:45.931514828+00:00 INFO controller::controller - chain_check_proposal: add remote proposal(0x4fe40496e71bf7a1d3344c6778c2cd9bea547763772fd276ab086ea3ceafe95c)
2022-11-16T07:47:45.935420225+00:00 INFO controller::controller - chain_check_proposal: finished
2022-11-16T07:47:46.014057182+00:00 INFO controller::node_manager - update node: NodeAddress(0a423460911494e1)
2022-11-16T07:47:46.014086372+00:00 INFO controller::controller - update global status node(NodeAddress(0a423460911494e1)) height(23364)
2022-11-16T07:47:46.015283211+00:00 INFO controller::chain - commit_block height: 23364 hash 0x4fe40496e71bf7a1d3344c6778c2cd9bea547763772fd276ab086ea3ceafe95c
2022-11-16T07:47:46.020220810+00:00 INFO controller::chain - exec_block(23364): status: Success, state_root: 0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421
2022-11-16T07:47:46.023409938+00:00 INFO controller::chain - finalize_block height: 23364
```

## 基本操作

### 指定链的RPC端口

链有两个`rpc`地址，分别是`controller`和`executor`微服务。

我们可以通过`-r`和`-e`来告诉`cldi`如何访问链：

使用如下命令映射节点0的`rpc`端口到本地。

```bash
$ kubectl port-forward -n $NAME_SPACE pod/$CHAIN_NAME-node0-0 50002:50002 50004:50004
```

对应的`cli`参数为：

```
-r localhost:50004 -e localhost:50002
```

### 查看块高

```
$ cldi -r localhost:50004 -e localhost:50002 get block-number
24806
```

### 查看系统配置

```
$ cldi -r localhost:50004 -e localhost:50002 get system-config
{
  "admin": "0xc8ca9cc77a7f822fdd0baef7a7740f9dba493455",
  "admin_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000",
  "block_interval": 3,
  "block_interval_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000",
  "block_limit": 100,
  "block_limit_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000",
  "chain_id": "0x63586a3c0255f337c77a777ff54f0040b8c388da04f23ecee6bfd4953a6512b4",
  "chain_id_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000",
  "emergency_brake": false,
  "emergency_brake_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000",
  "quota_limit": 1073741824,
  "quota_limit_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000",
  "validators": [
    "0x99ac6b222e64af1231dec421dbc22af377f178e070c62dfe272dd28ab2f69fd319fad5240068658e9f57831438bad431",
    "0xb594f9095578711c21f809e0f0cf3c5a2cb93b9b0f260075cea25af5aec2bea4aabe5e302fb31e289f550412fabe923b"
    "0xad93329676dfe026a591fdd6243d630cb28d2cd0e12a7ac2b2e52e1263709112e7e8aed5bfcb010ff207cf6ff58e1143",
    "0x95efa4fdd194a376885c30339a84c0636cf625254e9e7ee71b60b4a51ade4cb5741973f2839a08ff1d9197cdfff0dfd3"
  ],
  "validators_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000",
  "version": 0,
  "version_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000"
}
```

### 停止链

```bash
kubectl delete -f $CHAIN_NAME-node0/yamls/ -n $NAME_SPACE
kubectl delete -f $CHAIN_NAME-node1/yamls/ -n $NAME_SPACE
kubectl delete -f $CHAIN_NAME-node2/yamls/ -n $NAME_SPACE
kubectl delete -f $CHAIN_NAME-node3/yamls/ -n $NAME_SPACE
```

## 账户操作

### 创建账户

```
$ cldi account generate --name user
{
  "crypto_type": "SM",
  "address": "0xf4b80a27b7d526028183e705604b865c1458c838",
  "public_key": "0x71ebc3701780b4ac6a6ae0817e2fa402fb26082e6eade00f117015a1063a5c3af6e1f74c26c01f87af5ddea20fb28ff6d1a20f5438416afe95e9247ab2e517ce",
  "secret_key": "0x2757242a6138e9617b30ec63f6a240b880d25851d86d9d15849f2d45061d3288"
}
```

### 选择操作使用的账户

```
$ cldi -u user -r localhost:50004 -e localhost:50002
cldi> 
```

`-u`选择使用该账户，后续将以该账户的身份发送交易。

进入`cldi`的交互式模式。


## 发送交易

### 编译合约

这里以`Counter`合约为例:

```
$ cat Counter.sol 
pragma solidity ^0.4.24;

contract Counter {
    uint public count;
    
    function add() public {
        count += 1;
    }
    
    function reset() public {
        count = 0;
    }
}

$ curl -o solc -L https://github.com/ethereum/solidity/releases/download/v0.4.24/solc-static-linux
$ chmod +x solc
$ sudo mv ./solc /usr/local/bin/
$ solc --hashes --bin Counter.sol 

======= Counter.sol:Counter =======
Binary: 
608060405234801561001057600080fd5b5060f58061001f6000396000f3006080604052600436106053576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806306661abd1460585780634f2be91f146080578063d826f88f146094575b600080fd5b348015606357600080fd5b50606a60a8565b6040518082815260200191505060405180910390f35b348015608b57600080fd5b50609260ae565b005b348015609f57600080fd5b5060a660c0565b005b60005481565b60016000808282540192505081905550565b600080819055505600a165627a7a72305820a841f5848c8c68bc957103089b41e192a79aed7ac2aebaf35ae1e36469bd44d90029
Function signatures: 
4f2be91f: add()
06661abd: count()
d826f88f: reset()
```

### 创建合约

```
cldi> create 0x608060405234801561001057600080fd5b5060f58061001f6000396000f3006080604052600436106053576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806306661abd1460585780634f2be91f146080578063d826f88f146094575b600080fd5b348015606357600080fd5b50606a60a8565b6040518082815260200191505060405180910390f35b348015608b57600080fd5b50609260ae565b005b348015609f57600080fd5b5060a660c0565b005b60005481565b60016000808282540192505081905550565b600080819055505600a165627a7a72305820faa1d1f51d7b5ca2b200e0f6cdef4f2d7e44ee686209e300beb1146f40d32dee0029
0xf18153579e86b4d81617bf9d3b34e1ca2d0433296927f4b04d5c5219d2d82d46
```

创建合约的参数是编译合约输出的二进制字节码，注意前面要增加`0x`前缀。

返回值为这笔创建合约交易的交易哈希。

### 查看交易回执

```
cldi> get receipt 0xf18153579e86b4d81617bf9d3b34e1ca2d0433296927f4b04d5c5219d2d82d46
{
  "block_number": 454,
  "contract_addr": "0xa4582f4966bdef3a2839e2f256a714426508ddb7",
  "cumulative_quota_used": "0x0000000000000000000000000000000000000000000000000000000000018ed3",
  "error_msg": "",
  "legacy_cita_block_hash": "0x09b2e445d5b3a5e118bb0aad8d812d237ff24f2aa9e37da75e5c68d43ddcdbba",
  "logs": [],
  "logs_bloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "quota_used": "0x0000000000000000000000000000000000000000000000000000000000018ed3",
  "state_root": "0x28913fc520eaf50c72fad929d0f57c94ad487bfd63278a57cb40ef370a7697a0",
  "tx_hash": "0xf18153579e86b4d81617bf9d3b34e1ca2d0433296927f4b04d5c5219d2d82d46",
  "tx_index": 0
}
```

参数为前一步操作返回的交易哈希。

返回值为该笔交易的执行结果信息，其中`contract_addr`字段为刚才部署的合约的地址。

得到合约地址后，就可以调用其中的方法。

### 查询合约状态

查询合约中的`count`值：

```
cldi> call 0xa4582f4966bdef3a2839e2f256a714426508ddb7 0x06661abd
0x0000000000000000000000000000000000000000000000000000000000000000
```

第一个参数为合约地址。

第二个参数为要调用的`count()`方法的函数签名。

返回值为`count`的当前值。

### 发送交易

向合约发送交易，调用`add`方法改变`count`的值：

```
cldi> send 0xa4582f4966bdef3a2839e2f256a714426508ddb7 0x4f2be91f
0x24f0bc60340c49e875a8701e80849725a0a10e1bf47990e8c9cb399e31acea05
```

第一个参数为合约地址。

第二个参数为要调用的`add()`方法的函数签名。

等待交易上链之后，再次查询就可以发现合约状态有了变化：

```
cldi> call 0xa4582f4966bdef3a2839e2f256a714426508ddb7 0x06661abd
0x0000000000000000000000000000000000000000000000000000000000000001
```
