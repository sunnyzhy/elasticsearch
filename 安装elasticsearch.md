# 安装 elasticsearch
[elasticsearch官网](https://www.elastic.co/downloads/elasticsearch 'elasticsearch')

## 安装
```bash
# cd /usr/local

# tar -zxvf elasticsearch-7.10.1-linux-x86_64.tar.gz

# mv elasticsearch-7.10.1-linux-x86_64 elasticsearch
```

## 使用 ES 自带的 JDK
```bash
# vim ./bin/elasticsearch-env
```

把以下配置

```
# now set the path to java
if [ ! -z "$JAVA_HOME" ]; then
  JAVA="$JAVA_HOME/bin/java"
  JAVA_TYPE="JAVA_HOME"
else
  if [ "$(uname -s)" = "Darwin" ]; then
    # macOS has a different structure
    JAVA="$ES_HOME/jdk.app/Contents/Home/bin/java"
  else
    JAVA="$ES_HOME/jdk/bin/java"
  fi
  JAVA_TYPE="bundled jdk"
fi
```

修改为：

```
if [ "$(uname -s)" = "Darwin" ]; then
  # macOS has a different structure
  JAVA="$ES_HOME/jdk.app/Contents/Home/bin/java"
else
  JAVA="$ES_HOME/jdk/bin/java"
fi
JAVA_TYPE="bundled jdk"
```

## 授权非root用户
```
# chown -R zhy /usr/local/elasticsearch
```

## 切换到非root用户 & 启动elasticsearch
```
# su zhy

$ cd /usr/local/elasticsearch/bin

$ ./elasticsearch
```

## http 访问
# curl -u elastic:your_password -XGET http://192.168.0.100:9200
{
  "name" : "node-1",
  "cluster_name" : "zhy",
  "cluster_uuid" : "KDc8wiESR-OVkC9_-Kqxog",
  "version" : {
    "number" : "7.10.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "1c34507e66d7db1211f66f3513706fdf548736aa",
    "build_date" : "2020-12-05T01:00:33.671820Z",
    "build_snapshot" : false,
    "lucene_version" : "8.7.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

## 设置外网访问
```
$ vim /usr/local/elasticsearch/config/elasticsearch.yml
network.host: 0.0.0.0

$ ./elasticsearch -d
```

## 重启elasticsearch，关闭 -> 开启
```
$ ps aux | grep elasticsearch

$ kill pid

$ ./elasticsearch -d
```
