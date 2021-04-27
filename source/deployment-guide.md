# 部署指南

## 单集群

链的所有节点都在同一个`k8s`集群中。

#### NFS

持久化存储使用`NFS`。

硬件配置：

* 一个`k8s`集群。
* 一台专用的`NFS`服务器。

部署步骤：

1. 在`NFS`服务器上安装`nfs`服务。参考[链接](https://blog.csdn.net/qq_33789722/article/details/80280998)。
2. 生成`nfs-pvc`。
    ```
    $ ./create_pvc.py nfs_pvc --nfs_server 172.17.0.2 --nfs_path /data/nfs 
    $ ls
    nfs-pvc.yaml
    $ kubectl apply -f nfs-pvc.yaml
    ```
3. 生成链的配置。
    ```
    $ ./create_k8s_config.py local_cluster --kms_password 123456 --peers_count 3 --pvc_name nfs-pvc
    $ ls
    cita-cloud  test-chain.yaml
    ```
4. 拷贝配置文件到`NFS`服务器。
    ```
    scp -r ./cita-cloud 172.17.0.2:/data/nfs/
    ```
5. 部署链。
    ```
    kubectl apply -f test-chain.yaml
    ```

## 多集群

链的节点分布在多个k8s集群中。

## 对接云服务商

#### 阿里云

#### 华为云
