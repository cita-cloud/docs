# 快速入门

本章介绍的是使用`minikube`作为运行环境，使用默认推荐的组件，快速搭建一条链的操作方法。

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

#### Helm

`Helm`是`Kubernetes`的包管理器，是寻找、共享和使用为`Kubernetes`构建的软件的最佳方式。

安装方法参见[文档](https://helm.sh/docs/intro/install/)。

#### minikube

安装方法参见[官方文档](https://minikube.sigs.k8s.io/docs/start/)。

安装完成后用下面的命令启动`minikube`，国内需要在启动`minikube`时设置一些镜像参数。

注意不能使用`root`权限启动`minikube`。

```
minikube start --registry-mirror=https://hub-mirror.c.163.com --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers --vm-driver=docker --alsologtostderr -v=8 --base-image registry.cn-hangzhou.aliyuncs.com/google_containers/kicbase:v0.0.17
```
耐心等待，看到以下信息代表启动成功。

```
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

#### kubectl

`kubectl`是`Kubernetes`集群的命令行工具，通过`kubectl`能够对集群本身进行管理，并能够在集群上进行容器化应用的安装部署。

安装方法参见[官方文档](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)。

#### cloud-cli

该工具为`CITA-Cloud`链的命令行客户端，可以方便的对链进行常用的操作。

使用方法参见[文档](https://cita-cloud.github.io/cloud-cli/)。

```
$ wget https://github.com/cita-cloud/cloud-cli/releases/download/v0.5.0/cldi-x86_64-unknown-linux-musl.tar.gz
$ tar zxvf cldi-x86_64-unknown-linux-musl.tar.gz
$ sudo mv ./cldi /usr/local/bin/
$ cldi -h
cldi 0.5.0
Rivtower Technologies <contact@rivtower.com>
The command line interface to interact with CITA-Cloud

USAGE:
    cldi [OPTIONS] [SUBCOMMAND]

OPTIONS:
    -c, --context <context>       context setting
    -r <controller-addr>          controller address
    -e <executor-addr>            executor address
    -u <account-name>             account name
        --crypto <crypto-type>    The crypto type of the target chain [possible values: SM, ETH]
    -h, --help                    Print help information
    -V, --version                 Print version information

SUBCOMMANDS:
    get            Get data from chain
    send           Send transaction
    call           Call executor
    create         create an EVM contract
    context        Context commands
    account        Account commands
    admin          The admin commands for managing chain
    rpc            Other RPC commands
    ethabi         Ethereum ABI coder.
    bench          Simple benchmarks
    watch          Watch blocks
    completions    Generate completions for current shell. Add the output script to `.profile`
                       or `.bashrc` etc. to make it effective.
    help           Print this message or the help of the given subcommand(s)
```

## 运行链

### 添加Charts仓库

```
$ helm repo add cita-cloud https://cita-cloud.github.io/charts
$ helm repo update
$ helm search repo cita-cloud/
NAME                                            CHART VERSION   APP VERSION     DESCRIPTION
cita-cloud/cita-cloud-local-cluster             6.6.1           6.6.1           Setup CITA-Cloud blockchain in one k8s cluster
cita-cloud/cita-cloud-pvc                       6.6.1           6.6.1           Create PVC for CITA-Cloud
```

### 创建PVC
`PVC`(`PersistentVolumeClaim`)是对`PV`的申请(`Claim`)。`PVC`通常由普通用户创建和维护。需要为`Pod`分配存储资源时，用户可以创建一个`PVC`，指明使用的`StorageClass`，`Kubemetes`会动态创建并绑定相关的`PV`。

这里使用的是`minikube`自带的名为`standard`的`StorageClass`。

```
$ helm install local-pvc cita-cloud/cita-cloud-pvc --set scName=standard
$ kubectl get pvc
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
local-pvc   Bound    pvc-fd3eaebd-3413-4205-b88a-dbc6cee9a057   10Gi       RWO            standard        18m
```

对应的路径在`minikube`虚拟机内的`/tmp/hostpath-provisioner/default/local-pvc`。

注意：如果`minikube`版本为 `v1.20.0`，这里会有一个bug。详细情况和解决方法参见[链接](https://tonybai.com/2021/05/14/a-bug-of-minikube-1-20/)。

### 生成超级管理员账户

使用 `cldi` 创建账户
```
$ cldi account generate -h
cldi-account-generate
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

### 启动链

```
$ helm install test-chain cita-cloud/cita-cloud-local-cluster --set config.superAdmin=0xc8ca9cc77a7f822fdd0baef7a7740f9dba493455
NAME: test-chain
LAST DEPLOYED: Thu Mar 24 02:48:52 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
```

该命令会创建一条有4个节点，名为`test-chain`的链。

注意：`superAdmin`参数必须设置为自己生成的账户地址，此处仅为演示，切勿在正式环境中使用演示值。

### 查看运行情况

```
$ kubectl get pod
NAME                                               READY   STATUS    RESTARTS   AGE
test-chain-0                                       7/7     Running   0          8m3s
test-chain-1                                       7/7     Running   0          8m3s
test-chain-2                                       7/7     Running   0          8m3s
test-chain-3                                       7/7     Running   0          8m3s
```

查看日志：

```
$ minikube ssh
docker@minikube:~$ tail -10f /tmp/hostpath-provisioner/default/local-pvc/test-chain-0/logs/controller-service.log
2022-03-24T02:57:41.562867997+00:00 INFO controller::node_manager - update node: 0x6028b1113a9ac5f79d2fdfb37ca135812d675691
2022-03-24T02:57:43.863863944+00:00 INFO controller::controller - chain_check_proposal: add remote proposal(0x90543745dee079d9a37d2b5bd1e026ad092089c1f3fd88ebbc16b10b3d1926f3)
2022-03-24T02:57:43.864261069+00:00 INFO controller::controller - chain_check_proposal: finished
2022-03-24T02:57:44.441848936+00:00 INFO controller::chain - height: 103 hash 0x90543745dee079d9a37d2b5bd1e026ad092089c1f3fd88ebbc16b10b3d1926f3
2022-03-24T02:57:44.672342679+00:00 INFO controller::node_manager - update node: 0x6028b1113a9ac5f79d2fdfb37ca135812d675691
2022-03-24T02:57:44.672366020+00:00 INFO controller::controller - update global status node(0x6028b1113a9ac5f79d2fdfb37ca135812d675691) height(103)
2022-03-24T02:57:44.673242652+00:00 INFO controller::chain - exec_block(103): status: Success, executed_block_hash: 0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421
2022-03-24T02:57:44.783682991+00:00 INFO controller::chain - finalize_block: 103, block_hash: 0x90543745dee079d9a37d2b5bd1e026ad092089c1f3fd88ebbc16b10b3d1926f3
```

## 基本操作

### 指定链的RPC端口

链有两个`rpc`地址，分别是`controller`和`executor`微服务。

我们可以通过`-r`和`-e`来告诉`cldi`如何访问链：

使用如下命令映射节点0的`rpc`端口到本地。

```
$ kubectl port-forward pod/test-chain-0 50002:50002 50004:50004
```

对应的`cli`参数为：

```
-r localhost:50004 -e localhost:50002
```

### 查看块高

```
$ cldi -r localhost:50004 -e localhost:50002 get block-number
246
```

### 查看系统配置

```
$ cldi -r localhost:50004 -e localhost:50002 get system-config
{
  "admin": "0xc8ca9cc77a7f822fdd0baef7a7740f9dba493455",
  "admin_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000",
  "block_interval": 3,
  "block_interval_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000",
  "chain_id": "0x63586a3c0255f337c77a777ff54f0040b8c388da04f23ecee6bfd4953a6512b4",
  "chain_id_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000",
  "emergency_brake": false,
  "emergency_brake_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000",
  "validators": [
    "0x40283e85bfbe778a0ef0ee80688861ccc2c557a2",
    "0x7b9e332f91fe894da0260fe2207f021d2d24008c",
    "0x014146185e3b32e64f0d06021aa6fff8639d134a",
    "0x6028b1113a9ac5f79d2fdfb37ca135812d675691"
  ],
  "validators_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000",
  "version": 0,
  "version_pre_hash": "0x000000000000000000000000000000000000000000000000000000000000000000"
}
```

### 停止链

```
$ helm uninstall test-chain
release "test-chain" uninstalled
```

### 删除链

```
$ helm install clean cita-cloud/cita-cloud-config --set config.action.type=clean
NAME: clean
LAST DEPLOYED: Thu Mar 24 02:47:45 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

注意：该命令将永久性的删除链的所有数据，请谨慎操作。

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
