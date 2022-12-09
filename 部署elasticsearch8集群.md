# 部署 elasticsearch8 集群

## 安装 elasticsearch8

[安装elasticsearch8](./安装elasticsearch8.md '安装elasticsearch8')

## 配置集群节点

```bash
# vim /usr/local/elasticsearch-8.4.1/config/ elasticsearch.yml
```

```yml
cluster.name: elastic-cluster
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["192.168.1.2", "192.168.1.3", "192.168.1.4"]
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
```

## 配置集群间的证书

***在集群的任意一台服务器执行以下命令(不用输入任何内容，一路回车即可):***

```bash
# ./elasticsearch-certutil ca

# ./elasticsearch-certutil cert --ca elastic-stack-ca.p12
...
Certificates written to /usr/local/elasticsearch-8.4.1/elastic-certificates.p12
...
```

***注意返回内容里的证书位置，本示例中生成的证书位置在 ```/usr/local/elasticsearch-8.4.1/elastic-certificates.p12```***

***把生成的证书复制到 ```/usr/local/elasticsearch-8.4.1/config/certs``` 目录下:***

```bash
# mv /usr/local/elasticsearch-8.4.1/*.p12 /usr/local/elasticsearch-8.4.1/config/certs

# ls /usr/local/elasticsearch-8.4.1/config/certs
elastic-certificates.p12  elastic-stack-ca.p12
```

***在 ```elasticsearch.yml``` 文件中配置证书:***

```bash
# vim /usr/local/elasticsearch-8.4.1/config/ elasticsearch.yml
```

```yml
xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/elastic-stack-ca.p12

xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/elastic-certificates.p12
  truststore.path: certs/elastic-certificates.p12
```

## 分发 elasticsearch

***把配置好的 elasticsearch 完整的目录复制到集群内的其他服务器上。***
