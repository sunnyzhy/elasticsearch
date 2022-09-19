# 安装 elasticsearch8

[elasticsearch官网](https://www.elastic.co/downloads/elasticsearch 'elasticsearch')

## 安装

```bash
# tar -zxvf /usr/local/elasticsearch-8.4.1-linux-x86_64.tar.gz -C /usr/local

# vim /usr/local/elasticsearch-8.4.1/config/elasticsearch.yml
```

```yml
network.host: 0.0.0.0
```

## 添加非 root 用户

```bash
# groupadd zhy

# useradd zhy -g zhy -p zhy

# chown -R zhy:zhy /usr/local/elasticsearch-8.4.1

# echo "* soft nofile 65536" >> /etc/security/limits.conf

# echo "* hard nofile 65536" >> /etc/security/limits.conf

# echo "vm.max_map_count=655360" >> /etc/sysctl.conf

# sysctl -p
```

## 启动 elasticsearch

***注: 必须使用非 root 账号启动 elasticsearch***

```bash
# su - zhy -c '/usr/local/elasticsearch-8.4.1/bin/elasticsearch -d' &
```

elasticsearch8 会自动开启 xpack，即启动成功之后:

1. 在 ```config/certs``` 的目录下自动生成自签名证书:

    ```bash
    # tree /usr/local/elasticsearch-8.4.1/config
    /usr/local/elasticsearch-8.4.1/config
    ├── certs
    │   ├── http_ca.crt
    │   ├── http.p12
    │   └── transport.p12
    ├── elasticsearch.keystore
    ├── elasticsearch-plugins.example.yml
    ├── elasticsearch.yml
    ├── jvm.options
    ├── jvm.options.d
    ├── log4j2.properties
    ├── role_mapping.yml
    ├── roles.yml
    ├── users
    └── users_roles
    ```

2. 在 ```elasticsearch.yml``` 里自动添加以下配置:

    ```yml
    #----------------------- BEGIN SECURITY AUTO CONFIGURATION -----------------------
    #
    # The following settings, TLS certificates, and keys have been automatically      
    # generated to configure Elasticsearch security features on 16-09-2022 01:35:28
    #
    # --------------------------------------------------------------------------------

    # Enable security features
    xpack.security.enabled: true

    xpack.security.enrollment.enabled: true

    # Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
    xpack.security.http.ssl:
      enabled: true
      keystore.path: certs/http.p12

    # Enable encryption and mutual authentication between cluster nodes
    xpack.security.transport.ssl:
      enabled: true
      verification_mode: certificate
      keystore.path: certs/transport.p12
      truststore.path: certs/transport.p12
    # Create a new cluster with the current node only
    # Additional nodes can still join the cluster later
    cluster.initial_master_nodes: ["node1"]

    #----------------------- END SECURITY AUTO CONFIGURATION -------------------------
    ```

## http 访问

```bash
# curl --cacert /usr/local/elasticsearch-8.4.1/config/certs/http_ca.crt -u elastic:elastic -XGET https://192.168.0.10:9200
{
  "name" : "node1",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "cewXlORZQaegCUa9oPtweQ",
  "version" : {
    "number" : "8.4.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "2bd229c8e56650b42e40992322a76e7914258f0c",
    "build_date" : "2022-08-26T12:11:43.232597118Z",
    "build_snapshot" : false,
    "lucene_version" : "9.3.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## 重启 elasticsearch

```bash
# ps aux | grep elasticsearch

# kill pid

# su - zhy -c '/usr/local/elasticsearch-8.4.1/bin/elasticsearch -d' &
```
