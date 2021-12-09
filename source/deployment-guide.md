# 部署指南

部署一条`CITA-Cloud`链，除了准备好`k8s`集群外，还需要实现进行持久化存储和网络的设置。

## 持久化存储

链的节点是有状态的服务，需要挂载持久化存储来保存数据。

为了方便对接不同的存储服务，我们使用了`k8s`的`PV/PVC`对存储进行了抽象。

建议由运维人员配置`StorageClass`，对`PV/PVC`进行动态绑定。

例如`快速入门`中就使用了`minikube`自带的名为`standard`的`StorageClass`。

```
$ helm install local-pvc cita-cloud/cita-cloud-pvc --set scName=standard
```

用户需要根据自己的环境以及所使用的存储服务来设置`scName`参数。

`local-pvc`为创建的`PVC`的名字，用户也可以随意设置。

## 网络

网络方面，需要节点之间可以通过网络相互连接。

集群内部可以通过`k8s`的`SVC`来暴露节点的网络端口。

如果是跨集群的情况，则需要使用`LoaderBalancer`服务对外暴露节点的网络端口。

如使用公有云环境，请咨询当前使用的云服务商。

## 部署模式

部署模式可分为单集群和多集群。

#### 单集群

链的所有节点都在同一个`k8s`集群中。

我们提供了一个`Chart`工程来实现该部署模式。在此之前需先执行`快速入门`-`运行CITA-Cloud`中的`添加Charts仓库`和`创建PVC`。

```
$ helm install test-chain cita-cloud/cita-cloud-local-cluster --set config.superAdmin=0xae069e1925a1dad2a1f4c7034d87258dfd9b6532 --set pvcName=local-pvc
```

* `test-chain`为要创建的链的名字。
* `pvcName`参数指定了`PVC`的名字。
* `superAdmin`要修改为自己的管理员地址。
* `CITA-Cloud`的各个微服务都有多种实现，用户可以通过`xxx.imageName`和`xxx.imageTag`参数来选择要使用的实现。
* 更多参数参见[链接](https://github.com/cita-cloud/charts/tree/main/cita-cloud-local-cluster)。
* 部署上采用`statefulset`，链的每个节点对应一个`pod`。
* 网络使用了`headless service`，使得节点直接可以相互访问。

#### 多集群

链的节点分布在多个`k8s`集群中。

步骤如下：
1. 规划节点所属的`k8s`集群。
2. 提前设置好节点网络端口对外的`ip`和端口。
3. 集中生成配置。
4. 将生成好的节点文件夹，分别下发到所属的`k8s`集群。
5. 在多个`k8s`集群中分别运行节点。

第2步对外暴露节点网络端口。

如果使用[porterLB](https://openelb.github.io/)，可以使用如下命令创建`eip`。
```
$ helm install cita-cloud-eip cita-cloud/cita-cloud-eip
```
* `cita-cloud-eip`为`eip`的名字。
* 更多参数参见[链接](https://github.com/cita-cloud/charts/tree/main/cita-cloud-eip)。

然后使用如下命令创建`SVC`。
```
$ helm install test-chain-0-lb cita-cloud/cita-cloud-porter-lb
```
* `test-chain-0-lb`为`SVC`的名字。
* 更多参数参见[链接](https://github.com/cita-cloud/charts/tree/main/cita-cloud-porter-lb)。

第3步生成配置。

可以使用`Chart`工程：
```
$ helm install init-multi cita-cloud/cita-cloud-config --set config.action.type=initMulti --set config.chainName=test-chain --set config.action.initMulti.superAdmin=8f81961f263f45f88230375623394c9301c033e7 --set config.action.initMulti.kmsPasswordList="123456\,123456\,123456" --set config.action.initMulti.nodeList="192.168.10.123:40000:node0\,192.168.10.134:40000:node1\,192.168.10.135:40000:node2" --set pvcName=local-pvc
```
更多参数参见[链接](https://github.com/cita-cloud/charts/tree/main/cita-cloud-config)。

也可以直接使用[cita-cloud-config](https://github.com/cita-cloud/cita_cloud_config)。

第5步运行节点。

可以使用`Chart`工程：
```
$ helm install test-chain-node0 cita-cloud/cita-cloud-multi-cluster-node --set config.chainName=test-chain --set config.domain=node0
```
* `chainName`和`kmsPassword`参数要与第3步保持一致。
* `CITA-Cloud`的各个微服务都有多种实现，用户可以通过`xxx.imageName`和`xxx.imageTag`参数来选择要使用的实现。
* 更多参数参见[链接](https://github.com/cita-cloud/charts/tree/main/cita-cloud-multi-cluster-node)。

针对阿里云场景，可以使用[cita_cloud_operator](https://github.com/cita-cloud/operator)。`CITA-Cloud`的各个微服务都有多种实现，用户可以通过`service-config.toml`配置文件来选择要使用的实现。
