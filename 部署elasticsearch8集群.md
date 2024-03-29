# 部署 elasticsearch8 集群

## 前言

### 准备三台物理机

|物理机IP|node.name|
|--|--|
|192.168.0.1|node-1|
|192.168.0.2|node-2|
|192.168.0.3|node-3|

### 创建 elasticsearch 账号

```bash
userdel elastic
groupadd elastic
useradd elastic -g elastic -p elastic
```

### 安装 elasticsearch8

[安装elasticsearch8](./安装elasticsearch8.md '安装elasticsearch8')

## 创建证书

### 自动创建证书

#### 启动 node-1

##### 启动 node-1

```bash
# chown -R elastic:elastic /usr/local/elasticsearch-8.4.1

# su - elastic -c '/usr/local/elasticsearch-8.4.1/bin/elasticsearch -d'
```

***启动 ```node-1``` 之后，会在 ```/usr/local/elasticsearch-8.4.1/config/certs/``` 目录下自动创建证书。***

##### 停止 node-1

```bash
# ps -ef | grep elastic

# kill -9 <pid>
```

##### 修改 node-1 的配置文件

```bash
# vim /usr/local/elasticsearch-8.4.1/config/elasticsearch.yml
```

***注释或删掉 ```BEGIN SECURITY AUTO CONFIGURATION``` 和 ```END SECURITY AUTO CONFIGURATION``` 之间的 ```cluster.initial_master_nodes```:***

```yml
#cluster.initial_master_nodes: ["localhost.localdomain"]
```

### 创建自签名证书

[创建自签名证书](./创建自签名证书.md '创建自签名证书')

### 查看证书密码

***证书的密码保存在 ```config/elasticsearch.keystore```***

```bash
# ./elasticsearch-keystore list
keystore.seed
xpack.security.http.ssl.keystore.secure_password
xpack.security.transport.ssl.keystore.secure_password
xpack.security.transport.ssl.truststore.secure_password
```

查看 ```http.p12``` 的密码:

```bash
# ./elasticsearch-keystore show xpack.security.http.ssl.keystore.secure_password

# ./elasticsearch-keystore show xpack.security.http.ssl.truststore.secure_password
```

查看 ```transport.p12``` 的密码:

```bash
# ./elasticsearch-keystore show xpack.security.transport.ssl.keystore.secure_password

# ./elasticsearch-keystore show xpack.security.transport.ssl.truststore.secure_password
```

## 修改 node-1 的配置文件

```bash
# vim /usr/local/elasticsearch-8.4.1/config/elasticsearch.yml
```

```yml
cluster.name: elastic-cluster
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["192.168.1.2", "192.168.1.3", "192.168.1.4"]
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
```

注意：
1. ```node.name``` 尽量不要跟主机名相同。
2. 如果安装了 keepalived 并配置了 VIP，那么 ```network.host``` 就必须使用固定的主机IP，而不能使用 ```0.0.0.0```。

## 分发 elasticsearch

***把 node-1 完整的 elasticsearch 目录复制到集群内的其他服务器上。***

## 修改其他集群节点的配置

### node-2

```bash
# vim /usr/local/elasticsearch-8.4.1/config/elasticsearch.yml
```

```yml
cluster.name: elastic-cluster
node.name: node-2
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["192.168.1.2", "192.168.1.3", "192.168.1.4"]
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
```

### node-3

```bash
# vim /usr/local/elasticsearch-8.4.1/config/elasticsearch.yml
```

```yml
cluster.name: elastic-cluster
node.name: node-3
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["192.168.1.2", "192.168.1.3", "192.168.1.4"]
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
```

## 启动集群

### 启动 node-1

```bash
# su - elastic -c '/usr/local/elasticsearch-8.4.1/bin/elasticsearch -d'
```

### 启动 node-2

```bash
# chown -R elastic:elastic /usr/local/elasticsearch-8.4.1

# su - elastic -c '/usr/local/elasticsearch-8.4.1/bin/elasticsearch -d'
```

### 启动 node-3

```bash
# chown -R elastic:elastic /usr/local/elasticsearch-8.4.1

# su - elastic -c '/usr/local/elasticsearch-8.4.1/bin/elasticsearch -d'
```

## 重置集群密码

***在集群里任意节点下执行以下命令:***

```bash
# /usr/local/elasticsearch-8.4.1/bin/elasticsearch-reset-password -i -u elastic
This tool will reset the password of the [elastic] user.
You will be prompted to enter the password.
Please confirm that you would like to continue [y/N]y


Enter password for [elastic]: [elastic]
Re-enter password for [elastic]: [elastic]
Password for the [elastic] user successfully reset.
```

***密码设置为 ```elastic```***

## 查看集群状态

***在集群里任意节点下执行以下命令:***

- 如果指定 ```--cacert``` 的话，请求的 ```url``` 就必须指明自签名证书里配置的 ```hostnames``` 或 ```IP addresses```

```bash
# curl -k -u elastic:elastic https://localhost:9200/_cat/nodes?v
ip           heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
192.168.0.1             1          62   0    0.00    0.01     0.05 cdfhilmrstw *      node-1
192.168.0.3             4          63   0    0.01    0.03     0.07 cdfhilmrstw -      node-3
192.168.0.2             4          60   0    0.85    0.62     0.34 cdfhilmrstw -      node-2

# curl --cacert /usr/local/elasticsearch-8.4.1/config/certs/elasticsearch-ca.pem -u elastic:elastic https://node-1:9200/_cat/nodes?v

# curl --cacert /usr/local/elasticsearch-8.4.1/config/certs/elasticsearch-ca.pem -u elastic:elastic https://192.168.0.1:9200/_cat/nodes?v

# curl -k -u elastic:elastic https://localhost:9200/_cluster/health?pretty
{
  "cluster_name" : "elastic-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 1,
  "active_shards" : 2,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}

# curl --cacert /usr/local/elasticsearch-8.4.1/config/certs/elasticsearch-ca.pem -u elastic:elastic https://node-1:9200/_cluster/health?pretty

# curl --cacert /usr/local/elasticsearch-8.4.1/config/certs/elasticsearch-ca.pem -u elastic:elastic https://192.168.0.1:9200/_cluster/health?pretty
```

## FAQ

### 节点加入集群失败

删除所有节点的 ```data``` 目录:

```bash
rm -rf /usr/local/elasticsearch-8.4.1/config/data
```
