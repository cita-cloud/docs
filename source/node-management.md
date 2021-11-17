# 节点管理

## 节点分类

节点分为两大类：

* 共识节点：共识节点具有出块和投票权限，交易由共识节点排序并打包成块，共识完成后即被确认为合法区块。链创建时生成的节点默认为共识节点，后续管理员可修改共识节点成员。
* 普通节点：普通节点没有出块和投票权限，其他方面和共识节点相同。可以同步和验证链上所有的原始数据，接受交易数据并向其他节点广播。链创建之后添加的节点默认为普通节点。

公有链没有节点准入机制，意味着任何节点都可以接入链并同步其全部的数据，在满足一定的条件下都可以参加共识。而 `cita-cloud`对于共识节点和普通节点都进行了准入管理。对于身份验证失败的节点，即使该节点能够在网络层与其他节点连通，这些节点也会拒绝与之建立通讯会话，如此可避免信息泄漏。

## 共识节点管理
参见[治理-共识节点管理](https://cita-cloud-docs.readthedocs.io/zh_CN/latest/governance.html#id3)

## 普通节点管理

普通节点的管理，是指普通节点的添加与删除。

### 创建链
此时生成的三个节点默认是共识节点。

```
$ helm install init cita-cloud/cita-cloud-config --set config.chainName=test-chain --set pvcName=local-pvc --set 'config.arguments={--kms_password,123456,--nodes,192.168.12.2:30000\,192.168.12.3:30000\,192.168.12.4:30000,--is_bft,true,--is_stdout,true}' --wait && helm uninstall init
NAME: init
LAST DEPLOYED: Thu Jul 29 23:29:52 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
release "init" uninstalled
```

#### 生成文件

```
$ cd /tmp/hostpath-provisioner/default/local-pvc
$ ls
test-chain  test-chain-0  test-chain-1  test-chain-2  test-chain.config
```

### 添加普通节点

```
$ helm install increase cita-cloud/cita-cloud-config --set config.action=increase --set config.chainName=test-chain --set pvcName=local-pvc --set 'config.arguments={--kms_password,123456,--node,192.168.12.5:30000}' --wait && helm uninstall increase
NAME: increase
LAST DEPLOYED: Thu Jul 29 23:38:26 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
release "increase" uninstalled
```

#### 生成文件

```
$ cd /tmp/hostpath-provisioner/default/local-pvc
$ ls
test-chain  test-chain-0  test-chain-1  test-chain-2  test-chain-3  test-chain.config
```

多了节点文件夹`test-chain-3`。

```
$ ls test-chain-3
blocks                  controller-log4rs.yaml  key_id               network_key          tx_infos
consensus-config.toml   executor-log4rs.yaml    kms-log4rs.yaml      node_address         txs
consensus-log4rs.yaml   genesis.toml            network-config.toml  node_key
controller-config.toml  init_sys_config.toml    network-log4rs.yaml  storage-log4rs.yaml
```

### 删除普通节点

```
$ helm install decrease cita-cloud/cita-cloud-config --set config.action=decrease --set config.chainName=test-chain --set pvcName=local-pvc --wait && helm uninstall decrease
NAME: decrease
LAST DEPLOYED: Thu Jul 29 23:42:38 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
release "decrease" uninstalled
```

#### 文件变化

```
$ cd /tmp/hostpath-provisioner/default/local-pvc/
$ ls
test-chain  test-chain-0  test-chain-1  test-chain-2 test-chain.config
```

少了节点文件夹`test-chain-3`。

### 删除链

```
$ helm install clean cita-cloud/cita-cloud-config --set config.action=clean --set config.chainName=test-chain --set pvcName=local-pvc --wait && helm uninstall clean
Location: /home/mid/.kube/config
NAME: clean
LAST DEPLOYED: Thu Jul 29 23:47:11 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
release "clean" uninstalled
```

#### 文件变化

```
$ cd /tmp/hostpath-provisioner/default/local-pvc/
/tmp/hostpath-provisioner/default/local-pvc$ ls
/tmp/hostpath-provisioner/default/local-pvc$
```

节点有关文件夹全部被删除。
