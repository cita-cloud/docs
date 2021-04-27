# 快速入门

## 环境准备

#### 硬件配置建议

* CPU：2核或以上
* 内存：4GB或以上
* 硬盘：30G或以上

#### 操作系统

常见的`Linux`发行版本均可，例如：`CentOS`，`Debian`，`Ubuntu`等等。

#### 软件依赖

* docker 安装方法参见[官方文档](https://docs.docker.com/engine/install/)。

#### k8s集群

`k8s`集群的搭建非常复杂，为了快速体验，我们推荐使用`minikube`，可以在本地快速搭建一个单节点的`k8s`集群。

安装运行`minikube`参见[链接](https://minikube.sigs.k8s.io/docs/start/)。

国内需要设置一些镜像参数，参考：

```
minikube start --registry-mirror=https://hub-mirror.c.163.com --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers --vm-driver=docker --alsologtostderr -v=8 --base-image registry.cn-hangzhou.aliyuncs.com/google_containers/kicbase:v0.0.10
```

安装`k8s`集群命令行工具`kubectl`，参考[官方文档](https://kubernetes.io/zh/docs/tasks/tools/install-kubectl/)。


## 生成链的配置

#### 软件依赖

* python3

#### 生成配置

1. 下载配置生成工具。
   
    ```
    $ wget https://github.com/cita-cloud/runner_k8s/archive/refs/heads/master.zip
    $ unzip master.zip
    $ cd runner_k8s-master
    $ ls
    config.xml         create_k8s_config.py  create_syncthing_config.py  requirements.txt
    create_account.py  create_pvc.py         README.md                   service-config.toml
    ```
   
2. 安装依赖包。
   
    ```
    pip install -r requirements.txt
    ```

3. 创建`PVC`。
   
    ```
    $ minikube ssh
    docker@minikube:~$ mkdir cita-cloud-datadir
    docker@minikube:~$ exit
    $ ./create_pvc.py local_pvc
    $ ls
    local-pvc.yaml
    $ kubectl apply -f local-pvc.yaml
    ```

4. 生成配置。
   
    ```
    ./create_k8s_config.py local_cluster --kms_password 123456 --peers_count 3 --pvc_name local-pvc
    $ ls
    cita-cloud  test-chain.yaml
    ```

生成的cita-cloud目录结构如下：

```
$ tree cita-cloud
cita-cloud
└── test-chain
    ├── node0
    ├── node1
    ├── node2
```

最外层是`cita-cloud`；第二层是链的名称，这里采用默认值`test-chain`；最里面是各个节点的文件夹。

`node0`，`node1`, `node2`是三个节点文件夹，里面有相应节点的配置文件。

`test-chain.yaml`用于将链部署到`k8s`集群，里面声明了相关的`secret/deployment/service`，文件名跟链的名称保持一致。

针对更复杂的情况，有更多可设置的参数，参见[runner_k8s文档](https://github.com/cita-cloud/runner_k8s/blob/master/README.md)。


## 运行链

#### 部署

```
$ scp -i ~/.minikube/machines/minikube/id_rsa -r cita-cloud docker@`minikube ip`:~/cita-cloud-datadir/
$ kubectl apply -f test-chain.yaml
```

#### 查看运行情况

查看节点是否运行正常：

```
$ kubectl get deployments.apps
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
test-chain-0              1/1     1            1           71s
test-chain-1              1/1     1            1           71s
test-chain-2              1/1     1            1           70s
```

查看日志：

```
$ minikube ssh
docker@minikube:~$ tail -10f cita-cloud-datadir/cita-cloud/test-chain/node0/logs/controller-service.log
2020-08-27T07:42:43.280172163+00:00 INFO controller::chain - 1 blocks finalized
2020-08-27T07:42:43.282871996+00:00 INFO controller::chain - executed block 1397 hash: 0x 16469..e061
2020-08-27T07:42:43.354375501+00:00 INFO controller::pool - before update len of pool 0, will update 0 tx
2020-08-27T07:42:43.354447445+00:00 INFO controller::pool - after update len of pool 0
2020-08-27T07:42:43.354467947+00:00 INFO controller::pool - low_bound before update: 0
2020-08-27T07:42:43.354484999+00:00 INFO controller::pool - low_bound after update: 0
2020-08-27T07:42:43.385062636+00:00 INFO controller::controller - get block from network
2020-08-27T07:42:43.385154627+00:00 INFO controller::controller - add block
2020-08-27T07:42:43.386148650+00:00 INFO controller::chain - add block 0x 11cf7..cad9
2020-08-27T07:42:46.319058783+00:00 INFO controller - reconfigure consensus!
```

## 基本操作

#### 软件依赖

* grpcurl [下载链接](https://github.com/fullstorydev/grpcurl/releases)

#### 下载cita_cloud_proto

```
$ wget https://github.com/cita-cloud/cita_cloud_proto/archive/refs/heads/master.zip
$ unzip master.zip
$ cd cita_cloud_proto-master
$ ls
build.rs  Cargo.toml  protos  README.md  src
```

#### 查询操作

`cita-cloud`的`RPC`接口采用的是`gRPC`，因此需要使用`grpcurl`工具才能在命令行访问。

同时需要相应的`protobuf`定义文件。

查看块高:

```
$ ./grpcurl -emit-defaults -plaintext -d '{"flag": false}' \
-proto ~/cita_cloud_proto-master/protos/controller.proto \
-import-path ~/cita_cloud_proto-master/protos \
`minikube ip`:30004 controller.RPCService/GetBlockNumber
{
"blockNumber": "320"
}
```

查看系统配置：

```
$ ./grpcurl -emit-defaults -plaintext -d '{"flag": false}' \
-proto ~/cita_cloud_proto-master/protos/controller.proto \
-import-path ~/cita_cloud_proto-master/protos \
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

注意：部分参数实际类型为`bytes`，`grpcurl`工具对其进行了`Base64`编码，输出显示为字符串。
