# 运维

## 日志管理

日志在系统调试，问题定位，甚至业务运行方面都有着重要作用。CITA-Cloud每个微服务的日志信息都会被记录到一个单独的日志文件。

### 日志位置

CITA-Cloud的日志文件位于各个节点文件夹下的`log`文件中，每个微服务单独一个日志文件。

```
$ ls test-chain-0/logs
consensus-service.log   executor-service.log  network-service.log
controller-service.log  kms-service.log       storage-service.log
```

### 日志等级

日志等级定义如下：

- Warn: 警告信息
- Info: 默认信息等级
- Debug: 调试信息
- Trace: 最低等级

CITA-Cloud会打印所设置等级及其以上的日志。

CITA-Cloud默认的日志等级为`info`.

```
$ cat controller-log4rs.yaml
# Set the default logging level and attach the default appender to the root
root:
  level: info
  appenders:
    - journey-service
```

`info `等级打印出的日志

```
$ tail -10f /tmp/hostpath-provisioner/default/local-pvc/test-chain-0/logs/controller-service.log
2021-08-03T06:17:06.966466511+00:00 INFO controller::chain - finalize_block: 3117, block_hash: 0x78bc32f8c352cba5af069d4882048c8957b3011486eeaf152446667ad8a187c0
2021-08-03T06:17:06.967758688+00:00 INFO controller::node_manager - update node: 0x417181eb1961c70d19ce84cb7ffb102c8cfc2e2e
2021-08-03T06:17:06.967789307+00:00 INFO controller::node_manager - update node: 0xd8cd634b16a9aaa5ae9b4e3a181c80d70f0cbf56
2021-08-03T06:17:07.012242063+00:00 INFO controller::controller - add remote proposal(0x8cc8d96407be8ae14c703c51dc8d8d19bd9e5c3321ede21b41c15937ea717e20) through check_proposal
2021-08-03T06:17:09.949227372+00:00 INFO controller::controller - add remote proposal(0xd8f56a1d049112a5a4a4cc8918a2beea9faa68d59ee2ad230733bf8a7c9e81f3) through check_proposal
2021-08-03T06:17:09.969532670+00:00 INFO controller::node_manager - update node: 0x417181eb1961c70d19ce84cb7ffb102c8cfc2e2e
2021-08-03T06:17:09.971888525+00:00 INFO controller::util - height: 3118 hash 0xd8f56a1d049112a5a4a4cc8918a2beea9faa68d59ee2ad230733bf8a7c9e81f3
···
```

* 在生成节点配置时，可以通过`--is_stdout`和`--log_level`设置日志的输出和等级。详情参见`配置说明`章节的相关内容。

创建时设置打印日志的默认等级为`Trace`，可以直接答应出`Trace`以及`Trace`级别以上的日志

```
$ helm install other-chain cita-cloud/cita-cloud-local-cluster --set pvcName=other-pvc --set config.logLevel=Trace
$ tail -10f /tmp/hostpath-provisioner/default/other-pvc/other-chain-0/logs/controller-service.log
2021-08-03T02:21:12.915282130+00:00 TRACE tracing::span::active - -> Prioritize::queue_frame
2021-08-03T02:21:12.915284920+00:00 TRACE h2::proto::streams::prioritize - schedule_send stream.id=StreamId(1303)
2021-08-03T02:21:12.915287840+00:00 TRACE h2::proto::streams::store - Queue::push
2021-08-03T02:21:12.915290190+00:00 TRACE h2::proto::streams::store -  -> first entry
2021-08-03T02:21:12.915292720+00:00 TRACE tracing::span::active - <- Prioritize::queue_frame
2021-08-03T02:21:12.915295570+00:00 TRACE tracing::span - -- Prioritize::queue_frame
2021-08-03T02:21:12.915298740+00:00 TRACE h2::proto::streams::prioritize - reserve_capacity; stream.id=StreamId(1303) requested=1 effective=1 curr=0
··· 
```

* 运行过程中也可动态修改微服务的日志配置文件`xxx-log4rs.yaml`。实现动态调整日志等级等。

## 异常排查

`k8s`环境调试非常不便，且制作容器镜像时要考虑体积的问题，容器内缺乏调试用的工具和命令。

为了便于调试，我们在节点的`pod`中增加了一个专门用于调试的容器。

* 如果部署时使用[cita_cloud_operator](https://github.com/cita-cloud/operator)。可以通过设置`--need_debug`参数为`true`开启该功能。默认用于调试的镜像是`praqma/network-multitool`，包含的工具和命令详情参见[链接](https://github.com/Praqma/Network-MultiTool)。
* 如果部署时使用`Chart`工程，则不论是`cita-cloud-local-cluster`还是`cita-cloud-multi-cluster-node`，都可以通过设置`debug.enabled`为`true`开启该功能，并且可以通过`debug.imageName`和`debug.imageTag`设置用于调试的容器镜像。

### 节点状况查询

* 使用`kubectl`的`get`命令直接查看当前节点的运行情况

```
$ kubectl get pod
NAME                      READY   STATUS    RESTARTS   AGE
test-chain-0              7/7     Running   0          149m
test-chain-1              7/7     Running   0          149m
test-chain-2              7/7     Running   0          149m
```

* CITA-Cloud使用`kubectl`的`describe`命令可以查看指定节点的详细状况。

```
$ kubectl describe pod test-chain-0
Name:         test-chain-0
Namespace:    default
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Mon, 02 Aug 2021 20:40:08 -0700
Labels:       app.kubernetes.io/instance=test-chain
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=cita-cloud-local-cluster
              app.kubernetes.io/version=6.1.0
              controller-revision-hash=test-chain-5b9f9b6b84
              helm.sh/chart=cita-cloud-local-cluster-6.1.0
              statefulset.kubernetes.io/pod-name=test-chain-0
Annotations:  <none>
Status:       Running
IP:           172.17.0.4
IPs:
  IP:           172.17.0.4
Controlled By:  StatefulSet/test-chain
Init Containers:
  cita-cloud-config:
    Container ID:  docker://fc1b13ce38fe81c50c77843f866df71a20bffb9ad0ec58402ab22e42c668b7e0
    Image:         citacloud/cita_cloud_config:v6.0.0
    Image ID:      docker-pullable://citacloud/cita_cloud_config@sha256:c148fd70cf28b886b0d23440625562bff559f16d4728be8bb256bc77855d2bfa
    Port:          <none>
    Host Port:     <none>
    Args:
      init
      --chain_name
      test-chain
      --super_admin
      0x415f568207900b6940477396fcd2c201efe49beb
      --is_bft
      true
      --work_dir
      /data
      --peers_count
      3
      --kms_password
      123456
      --is_stdout
      false
      --log_level
      info
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Mon, 02 Aug 2021 20:40:12 -0700
      Finished:     Mon, 02 Aug 2021 20:40:12 -0700
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data from datadir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-65x4l (ro)
Containers:
  debug:
    Container ID:   docker://69a64a03102154f06625b3ef4bc26c2bb1d57498f01433d858422e88a07ac0c4
    Image:          praqma/network-multitool:latest
    Image ID:       docker-pullable://praqma/network-multitool@sha256:7222852f7f120b44268f5bb8e2631bc667d740bd62dfc9eb695e89babd3e6d71
    Port:           9999/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 02 Aug 2021 20:40:14 -0700
    Ready:          True
    Restart Count:  0
    Environment:
      HTTP_PORT:  9999
      POD_NAME:   test-chain-0 (v1:metadata.name)
    Mounts:
      /data from datadir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-65x4l (ro)
  network:
    Container ID:  docker://b7b5eb98e9e2aabc7fcad6f3e07fb879ebd75015055ec628a287a138a1e7424c
    Image:         citacloud/network_direct:v6.0.0
    Image ID:      docker-pullable://citacloud/network_direct@sha256:385580e69f9a68bea808b072614feecd50e079d7bf4be3aad616a738cab6062d
    Ports:         40000/TCP, 50000/TCP
    Host Ports:    0/TCP, 0/TCP
    Command:
      sh
      -c
      network run -p 50000 -k network_key
    State:          Running
      Started:      Mon, 02 Aug 2021 20:40:15 -0700
    Ready:          True
    Restart Count:  0
    Environment:
      POD_NAME:  test-chain-0 (v1:metadata.name)
    Mounts:
      /data from datadir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-65x4l (ro)
  consensus:
    Container ID:  docker://d58537e6339069d33d9cf8e7b5216191705b5204fa556214d876dd3cf17830e4
    Image:         citacloud/consensus_bft:v6.0.0
    Image ID:      docker-pullable://citacloud/consensus_bft@sha256:c56c1f873783bb6f2a4400d9718592c3121779fb4f682b8646482573e1c33853
    Port:          50001/TCP
    Host Port:     0/TCP
    Command:
      sh
      -c
      consensus run -p 50001
    State:          Running
      Started:      Mon, 02 Aug 2021 20:40:16 -0700
    Ready:          True
    Restart Count:  0
    Environment:
      POD_NAME:  test-chain-0 (v1:metadata.name)
    Mounts:
      /data from datadir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-65x4l (ro)
  executor:
    Container ID:  docker://e886f8b826061e5c5d5fe8980b2bb594f65933815432a159c87c015d96419a49
    Image:         citacloud/executor_evm:v6.1.0
    Image ID:      docker-pullable://citacloud/executor_evm@sha256:25cde829576d6fa776e4c4022421da547152a628151ff038744fef2169da0892
    Port:          50002/TCP
    Host Port:     0/TCP
    Command:
      sh
      -c
      executor run -p 50002
    State:          Running
      Started:      Mon, 02 Aug 2021 20:40:17 -0700
    Ready:          True
    Restart Count:  0
    Environment:
      POD_NAME:  test-chain-0 (v1:metadata.name)
    Mounts:
      /data from datadir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-65x4l (ro)
  storage:
    Container ID:  docker://1ec6ecf500b3f046511dd3c82f9db887f99028f7d5945924e9e4b75fcb35c105
    Image:         citacloud/storage_rocksdb:v6.1.0
    Image ID:      docker-pullable://citacloud/storage_rocksdb@sha256:747f445b36e0e747212742de6ebc37e59f19a21fc2b6f5c6d7f1b8e8f8aba433
    Port:          50003/TCP
    Host Port:     0/TCP
    Command:
      sh
      -c
      storage run -p 50003
    State:          Running
      Started:      Mon, 02 Aug 2021 20:40:19 -0700
    Ready:          True
    Restart Count:  0
    Environment:
      POD_NAME:  test-chain-0 (v1:metadata.name)
    Mounts:
      /data from datadir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-65x4l (ro)
  controller:
    Container ID:  docker://2a661310c5d392496bf2721c26e98af8412c790276e0258171025ff0e4154ada
    Image:         citacloud/controller:v6.1.0
    Image ID:      docker-pullable://citacloud/controller@sha256:b0908004f6ea1a20c22f2d51a9f2e7e179348f5ebd68339c6c596934a8b8f761
    Port:          50004/TCP
    Host Port:     0/TCP
    Command:
      sh
      -c
      controller run -p 50004
    State:          Running
      Started:      Mon, 02 Aug 2021 20:40:20 -0700
    Ready:          True
    Restart Count:  0
    Environment:
      POD_NAME:  test-chain-0 (v1:metadata.name)
    Mounts:
      /data from datadir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-65x4l (ro)
  kms:
    Container ID:  docker://57a48e2f7fd968617c194499c31113c8746ba78260b5d9bdec0ab35336c20d6b
    Image:         citacloud/kms_sm:v6.0.0
    Image ID:      docker-pullable://citacloud/kms_sm@sha256:ae41f2d99d26f6b684f04f4c0e4c956db6c8984673a29e752673c4aed40f64da
    Port:          50005/TCP
    Host Port:     0/TCP
    Command:
      sh
      -c
      kms run -p 50005 -k /kms/key_file
    State:          Running
      Started:      Mon, 02 Aug 2021 20:40:22 -0700
    Ready:          True
    Restart Count:  0
    Environment:
      POD_NAME:  test-chain-0 (v1:metadata.name)
    Mounts:
      /data from datadir (rw)
      /kms from kms-key (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-65x4l (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kms-key:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  test-chain-kms-secret
    Optional:    false
  datadir:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  test-pvc
    ReadOnly:   false
  default-token-65x4l:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-65x4l
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m18s  default-scheduler  Successfully assigned default/test-chain-0 to minikube
  Normal  Pulled     2m15s  kubelet            Container image "citacloud/cita_cloud_config:v6.0.0" already present on machine
  Normal  Created    2m14s  kubelet            Created container cita-cloud-config
  Normal  Started    2m13s  kubelet            Started container cita-cloud-config
  Normal  Pulled     2m13s  kubelet            Container image "praqma/network-multitool:latest" already present on machine
  Normal  Started    2m12s  kubelet            Started container debug
  Normal  Created    2m12s  kubelet            Created container debug
  Normal  Pulled     2m12s  kubelet            Container image "citacloud/network_direct:v6.0.0" already present on machine
  Normal  Created    2m12s  kubelet            Created container network
  Normal  Started    2m11s  kubelet            Started container network
  Normal  Pulled     2m11s  kubelet            Container image "citacloud/consensus_bft:v6.0.0" already present on machine
  Normal  Created    2m11s  kubelet            Created container consensus
  Normal  Pulled     2m10s  kubelet            Container image "citacloud/executor_evm:v6.1.0" already present on machine
  Normal  Started    2m10s  kubelet            Started container consensus
  Normal  Created    2m9s   kubelet            Created container executor
  Normal  Started    2m9s   kubelet            Started container executor
  Normal  Pulled     2m9s   kubelet            Container image "citacloud/storage_rocksdb:v6.1.0" already present on machine
  Normal  Created    2m8s   kubelet            Created container storage
  Normal  Started    2m7s   kubelet            Started container storage
  Normal  Pulled     2m7s   kubelet            Container image "citacloud/controller:v6.1.0" already present on machine
  Normal  Created    2m6s   kubelet            Created container controller
  Normal  Started    2m5s   kubelet            Started container controller
  Normal  Pulled     2m5s   kubelet            Container image "citacloud/kms_sm:v6.0.0" already present on machine
  Normal  Created    2m4s   kubelet            Created container kms
  Normal  Started    2m4s   kubelet            Started container kms
```

## 服务监控

`CITA-Cloud`有针对节点的监控组件[CITA-Monitor](https://github.com/cita-cloud/cita_cloud_monitor)。

开启方法：

* 如果部署时使用[cita_cloud_operator](https://github.com/cita-cloud/operator)。可以通过设置`--need_monitor`参数为`true`开启该功能。
