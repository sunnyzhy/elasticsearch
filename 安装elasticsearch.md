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
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
[2017-11-22T15:39:40,848][INFO ][o.e.n.Node               ] [] initializing ...
[2017-11-22T15:39:41,102][INFO ][o.e.e.NodeEnvironment    ] [3breysr] using [1] data paths, mounts [[/ (rootfs)]], net usable_space [9.3gb], net total_space [17.4gb], types [rootfs]
[2017-11-22T15:39:41,102][INFO ][o.e.e.NodeEnvironment    ] [3breysr] heap size [1015.6mb], compressed ordinary object pointers [true]
[2017-11-22T15:39:41,103][INFO ][o.e.n.Node               ] node name [3breysr] derived from node ID [3breysr3TD6m4k8XRKo9aA]; set [node.name] to override
[2017-11-22T15:39:41,104][INFO ][o.e.n.Node               ] version[6.0.0], pid[36212], build[8f0685b/2017-11-10T18:41:22.859Z], OS[Linux/3.10.0-693.5.2.el7.x86_64/amd64], JVM[Oracle Corporation/OpenJDK 64-Bit Server VM/1.8.0_151/25.151-b12]
[2017-11-22T15:39:41,104][INFO ][o.e.n.Node               ] JVM arguments [-Xms1g, -Xmx1g, -XX:+UseConcMarkSweepGC, -XX:CMSInitiatingOccupancyFraction=75, -XX:+UseCMSInitiatingOccupancyOnly, -XX:+AlwaysPreTouch, -Xss1m, -Djava.awt.headless=true, -Dfile.encoding=UTF-8, -Djna.nosys=true, -XX:-OmitStackTraceInFastThrow, -Dio.netty.noUnsafe=true, -Dio.netty.noKeySetOptimization=true, -Dio.netty.recycler.maxCapacityPerThread=0, -Dlog4j.shutdownHookEnabled=false, -Dlog4j2.disable.jmx=true, -XX:+HeapDumpOnOutOfMemoryError, -Des.path.home=/usr/local/elasticsearch, -Des.path.conf=/usr/local/elasticsearch/config]
[2017-11-22T15:39:44,598][INFO ][o.e.p.PluginsService     ] [3breysr] loaded module [aggs-matrix-stats]
[2017-11-22T15:39:44,598][INFO ][o.e.p.PluginsService     ] [3breysr] loaded module [analysis-common]
[2017-11-22T15:39:44,598][INFO ][o.e.p.PluginsService     ] [3breysr] loaded module [ingest-common]
[2017-11-22T15:39:44,598][INFO ][o.e.p.PluginsService     ] [3breysr] loaded module [lang-expression]
[2017-11-22T15:39:44,598][INFO ][o.e.p.PluginsService     ] [3breysr] loaded module [lang-mustache]
[2017-11-22T15:39:44,598][INFO ][o.e.p.PluginsService     ] [3breysr] loaded module [lang-painless]
[2017-11-22T15:39:44,598][INFO ][o.e.p.PluginsService     ] [3breysr] loaded module [parent-join]
[2017-11-22T15:39:44,598][INFO ][o.e.p.PluginsService     ] [3breysr] loaded module [percolator]
[2017-11-22T15:39:44,598][INFO ][o.e.p.PluginsService     ] [3breysr] loaded module [reindex]
[2017-11-22T15:39:44,598][INFO ][o.e.p.PluginsService     ] [3breysr] loaded module [repository-url]
[2017-11-22T15:39:44,599][INFO ][o.e.p.PluginsService     ] [3breysr] loaded module [transport-netty4]
[2017-11-22T15:39:44,599][INFO ][o.e.p.PluginsService     ] [3breysr] loaded module [tribe]
[2017-11-22T15:39:44,599][INFO ][o.e.p.PluginsService     ] [3breysr] no plugins loaded
[2017-11-22T15:39:49,985][INFO ][o.e.d.DiscoveryModule    ] [3breysr] using discovery type [zen]
[2017-11-22T15:39:51,801][INFO ][o.e.n.Node               ] initialized
[2017-11-22T15:39:51,801][INFO ][o.e.n.Node               ] [3breysr] starting ...
[2017-11-22T15:39:52,274][INFO ][o.e.t.TransportService   ] [3breysr] publish_address {127.0.0.1:9300}, bound_addresses {[::1]:9300}, {127.0.0.1:9300}
[2017-11-22T15:39:52,319][WARN ][o.e.b.BootstrapChecks    ] [3breysr] max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
[2017-11-22T15:39:52,319][WARN ][o.e.b.BootstrapChecks    ] [3breysr] max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[2017-11-22T15:39:55,572][INFO ][o.e.c.s.MasterService    ] [3breysr] zen-disco-elected-as-master ([0] nodes joined), reason: new_master {3breysr}{3breysr3TD6m4k8XRKo9aA}{aRvY-6LJTxqPWC--NUEizw}{127.0.0.1}{127.0.0.1:9300}
[2017-11-22T15:39:55,577][INFO ][o.e.c.s.ClusterApplierService] [3breysr] new_master {3breysr}{3breysr3TD6m4k8XRKo9aA}{aRvY-6LJTxqPWC--NUEizw}{127.0.0.1}{127.0.0.1:9300}, reason: apply cluster state (from master [master {3breysr}{3breysr3TD6m4k8XRKo9aA}{aRvY-6LJTxqPWC--NUEizw}{127.0.0.1}{127.0.0.1:9300} committed version [1] source [zen-disco-elected-as-master ([0] nodes joined)]])
[2017-11-22T15:39:55,632][INFO ][o.e.g.GatewayService     ] [3breysr] recovered [0] indices into cluster_state
[2017-11-22T15:39:55,672][INFO ][o.e.h.n.Netty4HttpServerTransport] [3breysr] publish_address {127.0.0.1:9200}, bound_addresses {[::1]:9200}, {127.0.0.1:9200}
[2017-11-22T15:39:55,672][INFO ][o.e.n.Node               ] [3breysr] started
```

## 浏览器访问
http://localhost:9200/

```
{
  "name" : "3breysr",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "99BRBiSYRAWStXniuLFhFA",
  "version" : {
    "number" : "6.0.0",
    "build_hash" : "8f0685b",
    "build_date" : "2017-11-10T18:41:22.859Z",
    "build_snapshot" : false,
    "lucene_version" : "7.0.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
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

## 外网访问
http://ip:9200/

## 重启elasticsearch，关闭 -> 开启
```
$ ps aux | grep elasticsearch

$ kill pid

$ ./elasticsearch -d
```
