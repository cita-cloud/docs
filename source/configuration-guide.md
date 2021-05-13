# 配置说明

参见[runner_k8s文档](https://github.com/cita-cloud/runner_k8s/blob/master/README.md)。

## 链级配置

> **注意**
>
> 当你计划使用 CITA-Cloud设计产品时，[环境准备] 好之后，不要着急启动节点，请仔细阅读本节内容，并选择最适合你产品需求的配置。

链级配置指的是链自身的一些属性、系统初始配置，创世块，节点网络地址等配置，用户在`起链前`进行初始化配置。

本文档会详细介绍链的各个可配置项，然后通过具体的操作示例，演示如何起链前对链进行初始化配置。

#### `--chain_name`

指定链的名字。

* 链的名字本身不会保存在链上，但是系统初始配置中的`chain_id`是用`chain_name`计算得到的。计算方式为`chain_id = sha3_256(chain_name)`。
* `chain_id`在生成的 `init_sys_config.toml` 文件中可以查看到。
* 链的名字会作为文件夹的名称，该文件夹里面再按节点序号创建`node0，node1，node2`等节点文件夹，分别存放每个节点的配置文件。
* 如果没有传递 `chain_name` 参数，则默认链的名字为 `test-chain`。

#### `--timestamp`

指定起链的时间戳。

* 具体数值是指自 1970-1-1 以来的毫秒数，默认是取当前的时间。
* 这个值在生成的 `genesis.toml` 文件中可以查看到。
* 单集群模式下没有该参数，直接采用默认行为，取当前时间。

#### `--super_admin`

指定超级管理员地址。

* 该账户拥有最高权限，用来治理整条链。用户**必须**自己设置超级管理员。
* 单集群模式下没有该参数，自动创建相关的账户。
* 这个值在生成的 `init_sys_config.toml` 文件中可以查看到。

#### `--authorities`

指定所有共识节点的账户地址。

* 账户地址用逗号分割，个数需要和创建节点的个数保持一致，顺序也要和节点网络地址等保持一致。
* 安全起见，我们建议的流程是：先由每个共识节点单独生成各自的私钥和地址，私钥请务必由自己妥善保管；地址交由负责起链的超级管理员，生成链的配置。分发配置之后，起链前，由各节点独自将自己的私钥补充进来。
* 单集群模式下没有该参数，自动生成对应节点数量的私钥/地址对。

#### `--nodes`

指定所有节点的网络地址，`ip`或者域名。

* 节点网络地址用逗号分割，个数需要和创建节点的个数保持一致，顺序也要和共识节点的账户地址等保持一致。
* 单集群模式下没有该参数，自动创建`service`作为节点网络地址。

#### `--lbs_tokens`

负载均衡服务的`token`列表。

* 使用云厂商的负载均衡服务将集群内服务端口暴露到集群之外，需要事先申请`token`。
* `token`用逗号分割，个数需要和创建节点的个数保持一致，顺序也要和节点网络地址等保持一致。
* 单集群模式下没有该参数，因为单集群模式下不使用负载均衡服务，而是创建`NodePort`类型的`service`。

#### `--node_ports`

指定每个节点的起始端口。

* 每个链节点都有多个服务需要暴露到集群之外，用户只需要指定一个起始端口，多个服务自动往后排列。每个节点的端口信息如下：
    ```
    1. 网络 40000
    2. syncthing 22000
    3. rpc 50004
    4. executor_call 50002
    5. monitor_process 9256
    6. monitor_exporter 9349
    7. executor_chaincode 7052
    8. executor_eventhub 7053
    9. debug 9999
    
    比如，预留端口为30000~30008，则要使用的参数为30000。
    注意：根据配置参数的不同，其中部分端口背后的服务可能不会启动，但是端口依然会保留。
    例如，预留端口为30000~30008，不开启monitor的时候，30005和30006端口将不会使用，但是debug对应的依然是30008,而不会往前顺延。
    ```
* 多个节点的起始端口用逗号分割，个数需要和创建节点的个数保持一致，顺序也要和节点网络地址等保持一致。
* 单集群模式下对应的参数为`--node_port`，只需要传递一个起始端口，所有的节点的要暴露的端口会依次往下排。

#### `--sync_device_ids`

指定每个节点用于同步的`device id`。

* 通过`create_syncthing_config.py`创建，在创建`device id`的同时，会创建`cert.pem`和`key.pem`两个密钥文件。与共识节点的账户类似，生成配置阶段只需提供`device id`，分发配置之后，起链前，由各节点独自将自己的密钥补充进来。
* 多个节点的`device id`用逗号分割，个数需要和创建节点的个数保持一致，顺序也要和节点网络地址等保持一致。
* 单集群模式下没有该参数，自动为每个节点创建`device id`。

##### `--kms_passwords`

指定每个节点的`kms`服务的密码。

* 每个节点在创建自己的节点账户时就需要设置密码，生成配置时也需要提供。
* 多个节点的密码用逗号分割，个数需要和创建节点的个数保持一致，顺序也要和节点网络地址等保持一致。
* 单集群模式下对应的参数为`--kms_password`，只需要传递一个密码，所有的节点共用一个密码。

#### `--pvc_names`

指定每个节点使用的持久化存储的`persistentVolumeClaim name`。

* 公有云环境请咨询相应的云服务商。
* 自建环境可以使用`create_pvc.py`创建，目前支持`local path`和`nfs`。
* 多个节点的`pvc name`用逗号分割，个数需要和创建节点的个数保持一致，顺序也要和节点网络地址等保持一致。
* 单集群模式下对应的参数为`--pvc_name`，只需要传递一个`pvc name`，所有的节点共用同一个持久化存储。

#### `--need_debug`

是否需要增加调试用的容器。

* 如果设置为`true`,则会在每个节点的`Pod`内增加一个包含丰富调试工具的容器，方便定位问题。
* 默认为`false`。

#### `--enable_tls`

是否打开通信加密功能。

* 默认为`true`。

#### `--service_config`

选择链使用的各个微服务的具体实现。

`cita-cloud`分为六个微服务：`network`, `consensus`, `executor`, `storage`, `controller`, `kms`。

每个微服务都可能有多个不同的实现，`service-config.toml`用来配置每个微服务分别选择哪些实现，以及相应的启动命令。

目前微服务的实现有：

1. `network`。目前有`network_p2p`和`network_direct`两个实现。分别实现`p2p`和直连两种网络链接方式。
2. `consensus`。目前有`consensus_raft`和`consensus_bft`两个实现。
3. `executor`。目前有`executor_poc`和`executor_evm`两个实现。前者是一个空的实现，什么都不做，后者集成了以太坊的`EVM`。
4. `storage`。目前有`storage_rocksdb`，`storage_sqlite`两个实现。
5. `controller`。目前只有`controller`这一个实现。
6. `kms`。目前有`kms_eth`和`kms_sm`两个实现，分别兼容以太坊和国密。

注意：六个微服务缺一不可；每个微服务只能选择一个实现，不能多选。

## 微服务配置

#### network_p2p

命令行参数：

```
USAGE:
    network run [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -p, --port <grpc-port>       Sets grpc port of this service [default: 50000]
    -k, --key_file <key-file>    Sets path of network key file [default: network-key]
```

配置文件：

1. 日志配置文件 `network-log4rs.yaml`。日志配置参见[log4rs文档](https://docs.rs/log4rs/1.0.0/log4rs/#configuration-via-a-yaml-file)。
2. 节点私钥`network-key`。用于通信加密，文件内容为`0x`开头的十六进制字符串。
3. 服务配置文件 `network-config.toml`。

服务配置文件例子：

```
$ cat network-config.toml 
enable_tls = true
port = 40000
[[peers]]
ip = "test-chain-1"
port = 40000

[[peers]]
ip = "test-chain-2"
port = 40000
```

* `enable_tls` 是否打开通信加密功能。
* `peers`：配置对端节点信息。在网络服务启动时会首先连接 `peers` 中的节点。

#### network_direct

命令行参数：
```
USAGE:
    network run [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -p, --port <grpc-port>       Sets grpc port of this service [default: 50000]
    -k, --key_file <key-file>    Sets path of network key file. It's not used for network_direct.
                                 Leave it here for compatibility [default: network-key]
```

配置文件：

1. 日志配置文件 `network-log4rs.yaml`。日志配置参见[log4rs文档](https://docs.rs/log4rs/1.0.0/log4rs/#configuration-via-a-yaml-file)。
2. 服务配置文件 `network-config.toml`。同`network_p2p`。

#### consensus_raft

命令行参数：
```
USAGE:
    consensus run [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -p, --port <grpc-port>    Sets grpc port of this service [default: 50003]
```

配置文件：

1. 日志配置文件 `consensus-log4rs.yaml`。日志配置参见[log4rs文档](https://docs.rs/log4rs/1.0.0/log4rs/#configuration-via-a-yaml-file)。
2. 节点账户地址`node_address`。文件内容为`0x`开头的十六进制字符串。
3. 服务配置文件 `consensus-config.toml`。

服务配置文件例子：

```
network_port = 50000
controller_port = 50004
node_id = 0
```

* `network_port` 网络微服务的`gRPC`端口。
* `controller_port` 控制微服务的`gRPC`端口。
* `node_id` 废弃参数。

#### consensus_bft

命令行参数：
```
USAGE:
    consensus run [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -p, --port <grpc-port>    Sets grpc port of this service [default: 50001]
```

配置文件：

1. 日志配置文件 `consensus-log4rs.yaml`。日志配置参见[log4rs文档](https://docs.rs/log4rs/1.0.0/log4rs/#configuration-via-a-yaml-file)。
2. 节点账户地址`node_address`。文件内容为`0x`开头的十六进制字符串。
3. 节点私钥文件`node_key`。文件内容为`0x`开头的十六进制字符串。
4. 服务配置文件 `consensus-config.toml`。同`consensus_raft`。

#### executor_poc

命令行参数：
```
USAGE:
    executor run [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -p, --port <grpc-port>    Sets grpc port of this service [default: 50002]
```

配置文件：

1. 日志配置文件 `executor-log4rs.yaml`。日志配置参见[log4rs文档](https://docs.rs/log4rs/1.0.0/log4rs/#configuration-via-a-yaml-file)。

#### executor_evm

命令行参数：
```
USAGE:
    executor run [FLAGS] [OPTIONS]

FLAGS:
    -c, --compatibility    Sets eth compatibility, default false
    -h, --help             Prints help information
    -V, --version          Prints version information

OPTIONS:
    -p, --port <grpc-port>    Set executor port, default 50002
```

* `-c`参数表示是否与以太坊兼容（CITA 默认与以太坊在块的时间戳精度上不兼容，CITA 为毫秒，以太坊为秒）。

配置文件：

1. 日志配置文件 `executor-log4rs.yaml`。日志配置参见[log4rs文档](https://docs.rs/log4rs/1.0.0/log4rs/#configuration-via-a-yaml-file)。

#### storage_rocksdb

命令行参数：
```
USAGE:
    storage run [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -d, --db <db-path>        Sets db path [default: chain_data]
    -p, --port <grpc-port>    Sets grpc port of this service [default: 50003]
```

配置文件：

1. 日志配置文件 `storage-log4rs.yaml`。日志配置参见[log4rs文档](https://docs.rs/log4rs/1.0.0/log4rs/#configuration-via-a-yaml-file)。

#### storage_sqlite

命令行参数：
```
USAGE:
    storage run [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -d, --db <db-path>        Sets db path [default: storage.db]
    -p, --port <grpc-port>    Sets grpc port of this service [default: 50003]
```

配置文件：

1. 日志配置文件 `storage-log4rs.yaml`。日志配置参见[log4rs文档](https://docs.rs/log4rs/1.0.0/log4rs/#configuration-via-a-yaml-file)。

#### controller

命令行参数：
```
USAGE:
    controller run [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -p, --port <grpc-port>    Sets grpc port of this service [default: 50004]
```

配置文件：

1. 日志配置文件 `controller-log4rs.yaml`。日志配置参见[log4rs文档](https://docs.rs/log4rs/1.0.0/log4rs/#configuration-via-a-yaml-file)。
2. `key_id`，内容为一个十进制数字，表示节点账户在`kms`数据库中的序号。
2. 服务配置文件`controller-config.toml`。

服务配置文件例子：

```
network_port = 50000
consensus_port = 50001
storage_port = 50003
kms_port = 50005
executor_port = 50002
block_delay_number = 0
```

* `network_port` 网络微服务的`gRPC`端口。
* `consensus_port` 共识微服务的`gRPC`端口。
* `storage_port` 存储微服务的`gRPC`端口。
* `kms_port` `kms`微服务的`gRPC`端口。
* `executor_port` 执行器微服务的`gRPC`端口。
* `block_delay_number` 区块从共识完成到最终确认，延迟的区块数量。

#### kms_sm

命令行参数：
```
USAGE:
    kms run [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -d, --db <db-path>        Sets path of db file [default: kms.db]
    -p, --port <grpc-port>    Sets grpc port of this service [default: 50005]
    -k, --key <key-file>      Sets path of key_file
```

* `-k` 参数传递一个文件，内容为`kms`服务的密码。如果不传递该参数，则会进入交互模式，需要用户在命令行上输入密码。

配置文件：

1. 日志配置文件 `kms-log4rs.yaml`。日志配置参见[log4rs文档](https://docs.rs/log4rs/1.0.0/log4rs/#configuration-via-a-yaml-file)。

#### kms_eth

跟`kms_sm`相同。

## 密码学算法配置

目前`CITA-Cloud`支持两种密码学算法的组合（分别为签名算法和散列算法）：

1. [secp256k1] 与 [kecak]
2. [sm2] 与 [sm3]

用户可以通过在`service-config.toml`选择不同的`kms`微服务具体实现来选择不同的密码学组合。

1. 对应`kms_eth`。
2. 对应`kms_sm`。
