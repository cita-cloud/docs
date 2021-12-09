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
$ helm install init-multi cita-cloud/cita-cloud-config --set config.action.type=initMulti --set config.chainName=test-chain --set config.action.initMulti.superAdmin=8f81961f263f45f88230375623394c9301c033e7 --set config.action.initMulti.kmsPasswordList="123456\,123456\,123456" --set config.action.initMulti.nodeList="192.168.10.123:40000:node0\,192.168.10.134:40000:node1\,192.168.10.135:40000:node2" --set pvcName=local-pvc
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
test-chain  test-chain-node0  test-chain-node1  test-chain-node2
```

### 添加普通节点

```
$ helm uninstall init-multi 
release "init-multi" uninstalled
$ helm install increase-multi cita-cloud/cita-cloud-config --set config.action.type=increaseMulti --set config.chainName=test-chain --set config.action.increaseMulti.kmsPassword=123456 --set config.action.increaseMulti.node="192.168.10.136:40000:node3" --set pvcName=local-pvc
NAME: increase-multi
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
test-chain  test-chain-node0  test-chain-node1  test-chain-node2  test-chain-node3
```

多了节点文件夹`test-chain-3`。

```
$ ls test-chain-node3
config.toml             executor-log4rs.yaml  kms-log4rs.yaml      node_config.toml
controller-log4rs.yaml  kms.db                network-log4rs.yaml  storage-log4rs.yaml
```

### 删除普通节点

```
$ helm uninstall increase-multi
release "increase-multi" uninstalled
$ helm install decrease-multi cita-cloud/cita-cloud-config --set config.action.type=decreaseMulti --set config.chainName=test-chain --set config.action.decreaseMulti.domain=node3
NAME: decrease-multi
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
test-chain  test-chain-node0  test-chain-node1  test-chain-node2
```

少了节点文件夹`test-chain-node3`。

### 删除链

```
$ helm uninstall decrease-multi 
release "decrease-multi" uninstalled
$ helm install clean cita-cloud/cita-cloud-config --set config.action.type=clean
Location: /home/mid/.kube/config
NAME: clean
LAST DEPLOYED: Thu Jul 29 23:47:11 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
release "clean" uninstalled
$ helm uninstall clean 
release "clean" uninstalled
```

#### 文件变化

```
$ cd /tmp/hostpath-provisioner/default/local-pvc/
/tmp/hostpath-provisioner/default/local-pvc$ ls
/tmp/hostpath-provisioner/default/local-pvc$
```

节点有关文件夹全部被删除。
