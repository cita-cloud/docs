﻿# 快速入门

本章介绍的是使用minikube的快速入门操作方法，CITA-Cloud提供本地运行版本runner_local，参阅`本地运行`一节

## 环境准备

#### 硬件配置建议

* CPU：2核或以上
* 内存：4GB或以上
* 硬盘：30G或以上

#### 操作系统

常见的`Linux`发行版本均可，例如：`CentOS`，`Debian`，`Ubuntu`等等。

#### 软件依赖

* `docker` 安装方法参见[官方文档](https://docs.docker.com/engine/install/)。

* `helm` 安装方法参见[helm文档](https://helm.sh/docs/intro/install/)。Helm是Kubernetes的包管理器。Helm是寻找、共享和使用为Kubernetes构建的软件的最佳方式。

#### k8s集群

`k8s`集群的搭建非常复杂，在`快速入门`中，我们推荐使用`minikube`，可以在本地快速搭建一个单节点的`k8s`集群。

* `minikube` 安装方法参见[官方文档](https://minikube.sigs.k8s.io/docs/start/)。

安装完成后用下面的命令启动`minikube`，国内需要在启动`minikube`时设置一些镜像参数，注意不能在根权限下启动`minikube`。

```
minikube start --registry-mirror=https://hub-mirror.c.163.com --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers --vm-driver=docker --alsologtostderr -v=8 --base-image registry.cn-hangzhou.aliyuncs.com/google_containers/kicbase:v0.0.17
```
耐心等待，看到以下信息代表启动成功。

```
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

* `k8s`集群命令行工具`kubectl`安装方法参见[官方文档](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)。kubectl是Kubernetes集群的命令行工具,通过kubectl能够对集群本身进行管理,并能够在集群上进行容器化应用的安装部署。

#### cloud-cli

该工具为`CITA-Cloud`链的命令行客户端，可以方便的对链进行常用的操作。

```
$ wget https://github.com/cita-cloud/cloud-cli/releases/download/v0.1.1/cldi
$ chmod +x cldi
$ sudo mv ./cldi /usr/local/bin/
$ cldi -h
cloud-cli 0.1.0
The command line interface to interact with `CITA-Cloud`.

USAGE:
    cldi [OPTIONS] [SUBCOMMAND]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -e, --executor_addr <executor_addr>    executor address
    -r, --rpc_addr <rpc_addr>              rpc(controller) address
    -u, --user <user>                      the user(account) to send tx

SUBCOMMANDS:
    account               Manage account
    bench                 Send multiple txs with random content
    block-hash            Get block hash by block number(height)
    block-number          Get block number
    call                  Executor call
    completions           Generate completions for current shell
    create                Create contract
    emergency-brake       Send emergency brake cmd to chain
    get-abi               Get specific contract abi
    get-balance           Get balance by account address
    get-block             Get block by number or hash
    get-code              Get code by contract address
    get-tx                Get transaction by hash
    help                  Prints this message or the help of the given subcommand(s)
    peer-count            Get peer count
    receipt               Get receipt by tx_hash
    send                  Send transaction
    set-block-interval    Set block interval
    store-abi             Store abi
    system-config         Get system config
    update-admin          Update admin of the chain
    update-validators     Update validators of the chain
```

## 运行CITA-Cloud


#### 添加Charts仓库

```
$ helm repo add cita-cloud https://registry.devops.rivtower.com/chartrepo/cita-cloud
$ helm repo update
$ helm search repo cita-cloud
NAME                                            CHART VERSION   APP VERSION     DESCRIPTION                                       
cita-cloud/cita-cloud-config                    6.0.0           6.0.0           Create a job to change config of CITA-Cloud blo...
cita-cloud/cita-cloud-eip                       6.0.0           6.0.0           Create EIP for CITA-Cloud                         
cita-cloud/cita-cloud-local-cluster             6.1.0           6.1.0           Setup CITA-Cloud blockchain in one k8s cluster    
cita-cloud/cita-cloud-multi-cluster-node        6.1.0           6.1.0           Setup CITA-Cloud node in multi k8s cluster        
cita-cloud/cita-cloud-porter-lb                 6.0.0           6.0.0           Setup porter Loadbalancer for CITA-Cloud node     
cita-cloud/cita-cloud-pvc                       6.0.0           6.0.0           Create PVC for CITA-Cloud 
```

#### 创建PVC
PersistentVolumeClaim (PVC)是对PV的申请(Claim)。PVC通常由普通用户创建 和维护。需要为Pod分配存储资源时，用户可以创建一个PVC,指明存储资源的容量大小 和访问模式(比如只读)等信息，Kubemetes会查找并提供满足条件的PV。

```
$ helm install local-pvc cita-cloud/cita-cloud-pvc --set scName=standard
$ kubectl get pvc
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
local-pvc   Bound    pvc-fd3eaebd-3413-4205-b88a-dbc6cee9a057   10Gi       RWO            standard        18m
```

对应的路径在`minikube`虚拟机内的 `/tmp/hostpath-provisioner/default/local-pvc`。

注意：如果`minikube`版本为 `v1.20.0`，这里会有一个bug。详细情况和解决方法参见[链接](https://tonybai.com/2021/05/14/a-bug-of-minikube-1-20/)。

#### 生成超级管理员账户

```
$ cldi account create admin
user: `admin`
account_addr: 0xae069e1925a1dad2a1f4c7034d87258dfd9b6532
```


#### 运行CITA-Cloud

```
$ helm install test-chain cita-cloud/cita-cloud-local-cluster --set config.superAdmin=0xae069e1925a1dad2a1f4c7034d87258dfd9b6532
NAME: test-chain
LAST DEPLOYED: Wed Jul 14 23:09:37 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
```

该命令会创建一条有3个节点，链名为`test-chain`的链。

注意：`superAdmin`参数必须设置为自己生成的账户地址，切勿使用默认值。

#### 查看运行情况

```
$ kubectl get pod
NAME                      READY      STATUS       RESTARTS     AGE
test-chain-0              7/7        Running      1            3m24s
test-chain-1              7/7        Running      1            3m24s
test-chain-2              7/7        Running      1            3m24s
```

查看日志：

```
$ minikube ssh
docker@minikube:~$ tail -10f /tmp/hostpath-provisioner/default/local-pvc/test-chain-0/logs/controller-service.log
2021-07-21T08:12:07.242086786+00:00 INFO controller::node_manager - update node: 0xc3469...ebaeb
2021-07-21T08:12:07.242251607+00:00 INFO controller::node_manager - update node: 0xa6a1...208f9
2021-07-21T08:12:07.284207745+00:00 INFO controller::controller - add remote proposal(0x7b28...5cc64) through check_proposal
2021-07-21T08:12:10.228046821+00:00 INFO controller::controller - add remote proposal(0x26e7...fcf91) through check_proposal
2021-07-21T08:12:10.251289942+00:00 INFO controller::util - height: 126 hash 0x26e7...fcf91
2021-07-21T08:12:10.253093372+00:00 INFO controller::chain - finalize_block: 126, block_hash: 0x26e7...fcf91
2021-07-21T08:12:10.254012416+00:00 INFO controller::node_manager - update node: 0xc346...ebaeb
2021-07-21T08:12:10.254931221+00:00 INFO controller::node_manager - update node: 0x766a...7a954
2021-07-21T08:12:10.273834898+00:00 WARN controller - rpc: process_network_msg failed: Receive early status from same node
2021-07-21T08:12:10.296086993+00:00 INFO controller::controller - add remote proposal(0xe5f4...973cd) through check_proposal
2021-07-21T08:12:13.216728462+00:00 INFO controller::chain - main_chain_tx_hash len 0
2021-07-21T08:12:13.216755352+00:00 INFO controller::chain - tx poll len 0
2021-07-21T08:12:13.216766922+00:00 INFO controller::chain - proposal 127 prevhash 0x26e7...fcf91
2021-07-21T08:12:13.216778762+00:00 INFO controller::chain - proposal 127 block_hash 0x6a14...1e4e6
2021-07-21T08:12:13.256383088+00:00 INFO controller::util - height: 127 hash 0x6a14...1e4e6
```

## 基本操作

#### 指定链的RPC端口

可以通过设置环境变量的方式，为`cloud-cli`工具指定链的`RPC`端口：

```
$ export CITA_CLOUD_RPC_ADDR=`minikube ip`:30004 CITA_CLOUD_EXECUTOR_ADDR=`minikube ip`:30005
```

注意：这里minikube可能出现svc端口映射问题 使用以下命令解决

```
$ kubectl port-forward pod/test-chain-0 50002:50002 50004:50004
$ export CITA_CLOUD_RPC_ADDR=localhost:30004 CITA_CLOUD_EXECUTOR_ADDR=localhost:30005
```

#### 查看块高

```
$ cldi block-number
block_number: 74
```

#### 查看系统配置

```
$ cldi system-config
{
  "admin": "0xae069e1925a1dad2a1f4c7034d87258dfd9b6532",
  "block_interval": 3,
  "chain_id": "0x26b0b83e7281be3b117658b6f2636d0368cad3d74f22243428f5401a4b70897e",
  "validators": [
    "0x67611c4afee6a50c56d3a81c733260c2e1ca35ab",
    "0x97e05af22f5870c67b7f98bc6c7ebbba0273376b",
    "0x3275a8e90a92a29496edd9a7a5853a3fd3d51451",
    "0x148d37bd89ba2a8c7b5e4784df51a355439166b9"
  ],
  "version": 0
}
```

#### 停止链

```
$ helm uninstall test-chain
release "test-chain" uninstalled
```

#### 删除链

```
$ helm install clean cita-cloud/cita-cloud-config --set config.action=clean --set pvcName=local-pvc
NAME: clean
LAST DEPLOYED: Wed Jul 14 20:20:37 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

注意：该命令将永久性的删除链的所有数据，请谨慎操作。

## 账户操作

#### 创建账户

```
$ cldi account create test
user: `test`
account_addr: 0x415f568207900b6940477396fcd2c201efe49beb
```

#### 查看账户信息

创建账户后通过命令查看该账户的账户地址、私钥以及公钥：

```
$ cldi account export test
{
  "account_addr": "0x415f568207900b6940477396fcd2c201efe49beb",
  "private_key": "0x4f894fc00e6c71c7d0dde511eef64824b64fce87fd6f43492808cb99e9f22a57",
  "public_key": "0x6dd968546f2af3053be1d31aed723b58bf5380884ec5f2d41e8156dcc17c1317456c2cc9fb28290d7da0e606267ec1b00bfe54bb214ba5d6c2831c8211e9f343"
}
```

#### 登录账户

登录刚刚创建的`test`账户

```
$ cldi account login test
OK, now the default user is `test`, account addr is 0x415f568207900b6940477396fcd2c201efe49beb
```

登录账户之后，可以以该账户的身份发送交易。

## 发送交易

#### 编译合约

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

#### 创建合约

```
$ cldi create 0x608060405234801561001057600080fd5b5060f58061001f6000396000f3006080604052600436106053576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806306661abd1460585780634f2be91f146080578063d826f88f146094575b600080fd5b348015606357600080fd5b50606a60a8565b6040518082815260200191505060405180910390f35b348015608b57600080fd5b50609260ae565b005b348015609f57600080fd5b5060a660c0565b005b60005481565b60016000808282540192505081905550565b600080819055505600a165627a7a72305820faa1d1f51d7b5ca2b200e0f6cdef4f2d7e44ee686209e300beb1146f40d32dee0029
tx_hash: 0xb8fdb0a82d0403921fd0c11c7846500bdb5c032ad2e615dd1e8f42439f38f516
```

创建合约的参数是编译合约输出的二进制字节码，注意前面要增加`0x`前缀。

返回值为这笔创建合约交易的交易哈希。

#### 查看交易回执

```
$ cldi receipt 0xb8fdb0a82d0403921fd0c11c7846500bdb5c032ad2e615dd1e8f42439f38f516
{
  "block_number": 1111,
  "contract_addr": "0x138e2c0098ab9a5c38a7bc4a16a186f58fc4058a",
  "cumulative_quota_used": "0x0000000000000000000000000000000000000000000000000000000000018ed3",
  "error_msg": "",
  "legacy_cita_block_hash": "0x1b99b5e81feaef86dca9a309644ddbaaa7377733712b850ce50aff6c01ab2d4e",
  "logs": [],
  "logs_bloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "quota_used": "0x0000000000000000000000000000000000000000000000000000000000018ed3",
  "state_root": "0x9584cba462dec67dafaf5ae1210c8e3b3e58991ca31cafdbc395884b7d880364",
  "tx_hash": "0xb8fdb0a82d0403921fd0c11c7846500bdb5c032ad2e615dd1e8f42439f38f516",
  "tx_index": 0
}
```

参数为前一步操作返回的交易哈希。

返回值为该笔交易的执行结果信息，其中`contract_addr`字段为刚才部署的合约的地址。

得到合约地址后，就可以调用其中的方法。

##### 查询合约状态

查询合约中的`count`值：

```
$ cldi call -t 0x138e2c0098ab9a5c38a7bc4a16a186f58fc4058a 0x06661abd
result: 0x0000000000000000000000000000000000000000000000000000000000000000
```

`-t`参数为合约地址。

第二个参数为要调用的`count()`方法的函数签名。

返回值为`count`的当前值。

#### 发送交易

向合约发送交易，调用`add`方法改变`count`的值：

```
$ cldi send -t 0x138e2c0098ab9a5c38a7bc4a16a186f58fc4058a 0x4f2be91f
```

`-t`参数为合约地址。

第二个参数为要调用的`add()`方法的函数签名。

等待交易上链之后，再次查询就可以发现合约状态有了变化：

```
$ cldi call -t 0x138e2c0098ab9a5c38a7bc4a16a186f58fc4058a 0x06661abd
result: 0x0000000000000000000000000000000000000000000000000000000000000001
```

