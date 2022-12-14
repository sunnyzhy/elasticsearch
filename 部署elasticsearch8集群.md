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

##### 修改配置文件

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

#cluster.initial_master_nodes: ["localhost.localdomain"]
```

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

#cluster.initial_master_nodes: ["localhost.localdomain"]
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

#cluster.initial_master_nodes: ["localhost.localdomain"]
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
