# FAQ

## 1 blocked by: [FORBIDDEN/12/index read-only / allow delete (api)]

### 问题原因
1. 这是由于ES新节点的数据目录data存储空间不足，导致从master主节点接收同步数据的时候失败，此时ES集群为了保护数据，会自动把索引分片index置为只读read-only

2. 即使磁盘扩容了，但是在扩容之前（也就是磁盘空间不足的时候），对索引进行了写入的操作，由于 ES 的保护机制，会将该索引置为只读状态。所以扩容之后，依旧会提示上述错误

**不管是哪种情况，都是因为存储空间不足引起的。**


### 解决方案
- 提供足够的存储空间供数据写入，如需在配置文件中更改ES数据存储目录，注意重启ES

- 修改索引的只读设置

```
PUT /_all/_settings
{
	"index": {
		"blocks": {
			"read_only_allow_delete": "false"
		}
	}
}
```

**_all 可以修改为具体的索引名称**

### 官网建议

If your elasticsearch is responding with 403 and this message:
```
{
  "type": "cluster_block_exception",
  "reason": "blocked by: [FORBIDDEN/12/index read-only / allow delete (api)];"
}
```

Then you probably recovered from a full hard drive. The thing is, elasticsearch is switching to read-only if it cannot index more documents because your hard drive is full. With this it ensures availability for read-only queries. Elasticsearch will not switch back automatically but you can disable it by sending

```
curl -u elastic:password -XPUT -H "Content-Type: application/json" http://localhost:9200/_all/_settings -d '{"index":{"blocks":{"read_only_allow_delete":"false"}}}'
```

## 2 java.lang.IllegalStateException: failed to obtain node locks, tried [[/usr/local/elasticsearch/data/elasticsearch]] with lock id [0]; maybe these locations are not writable or multiple nodes were started without increasing [node.max_local_storage_nodes]

```bash
$ cd /usr/local/elasticsearch/data

$ rm -rf nodes
```

## 3 max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]

```bash
$ su root
Password: 

# vim /etc/security/limits.conf
* soft nofile 65536
* hard nofile 131072
```

## 4 max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

```bash
$ su root
Password: 

# vim /etc/sysctl.conf
vm.max_map_count=655360

# sysctl -p
vm.max_map_count = 655360
```

## 5 elasticsearch status red

### 单机

```bash
# curl -u elastic:elastic -X GET http://localhost:9200/_cat/health?v

# curl -u elastic:elastic -X GET http://localhost:9200/_cat/indices?v

# curl -u elastic:elastic -X DELETE http://localhost:9200/<red_status_index>
```

### 集群

```bash
# curl -u elastic:elastic -X GET http://localhost:9200/_cluster/health?level=indices

# curl -u elastic:elastic -X DELETE http://localhost:9200/<red_status_index>
```

### 删除部分数据

示例，删除 ```8:00 - 10:00 ``` 之间的全部数据:

```bash
# curl -u elastic:elastic -X POST "http://localhost:9200/<red_status_index>/_delete_by_query?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-3h",
        "lt": "now-1h"
      }
    }
  }
}'

# curl -u elastic:elastic -X POST http://localhost:9200/<red_status_index>/_forcemerge?max_num_segments=1&only_expunge_deletes=true
```

***注: 删除的时候，依据实际情况适当设置时间戳的范围。***

## 6 Result window is too large, from + size must be less than or equal to: [10000] but was [10050].

如果在 elasticsearch.yml 中增加 index.max_result_window: 20000 配置的话，启动 elasticsearch 会抛出如下异常：

```
*************************************************************************************
Found index level settings on node level configuration.

Since elasticsearch 5.x index level settings can NOT be set on the nodes 
configuration like the elasticsearch.yaml, in system properties or command line 
arguments.In order to upgrade all indices the settings must be updated via the 
/${index}/_settings API. Unless all settings are dynamic all indices must be closed 
in order to apply the upgradeIndices created in the future should use index templates 
to set default values. 

Please ensure all required values are updated on all indices by executing: 

curl -XPUT 'http://localhost:9200/_all/_settings?preserve_existing=true' -d '{
  "index.max_result_window" : "20000"
}'
*************************************************************************************
```

### 正确的做法

1. 用 curl 命令行
   ```bash
   curl -XPUT 'http://localhost:9200/${index}/_settings?preserve_existing=true' -d '{"index.max_result_window" : "20000"}'
   ```

2. 用 Kibana 命令行
   ```
   PUT /${index}/_settings
   {
      "index": {
        "max_result_window": 20000
      }
   }
   ```

## 7 totalHits 最大值是 10000

[Track total hits](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-track-total-hits.html "Elasticsearch Reference [7.2] » Track total hits")

1. 查询匹配的精确计数
   ```
   GET twitter/_search
   {
       "track_total_hits": true,
        "query": {
           "match" : {
               "message" : "Elasticsearch"
           }
        }
   }
   ```

2. 查询匹配的最多 N 个计数
   ```
   GET twitter/_search
   {
       "track_total_hits": 100,
        "query": {
           "match" : {
               "message" : "Elasticsearch"
           }
        }
   }
   ```

## 8 must filter should must_not的正确用法

[Bool Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html "Elasticsearch Reference [7.2] » Bool Query")

**特别注意**

- filter

   The clause (query) must appear in matching documents. **However unlike must the score of the query will be ignored.** Filter clauses are executed in filter context, meaning that scoring is ignored and clauses are considered for caching.

- should

   should is roughly equivalent to the boolean OR. **Note that should isn't exactly like a boolean OR,** but we can use it to that effect. 

```
因为 should context 下的查询语句可以一个都不满足，所以务必结合 minimum_should_match 使用
```

## 9 查看分词结果

[Term Vectors](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-termvectors.html "Elasticsearch Reference [7.2] » Term Vectors")

```
GET /${index}/_termvectors/${id}?fields=${fields}
```

## 10 An HTTP line is larger than 4096 bytes

{"type":"too_long_frame_exception","reason":"An HTTP line is larger than 4096 bytes."}，默认情况下ES对请求参数设置为4K，如果遇到请求参数长度限制可以在elasticsearch.yml中修改如下参数：

```yml
http.max_initial_line_length: "8k"

http.max_header_size: "16k"
```

[官网配置说明](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-http.html)

## 11 Kibana启动报错：[resource_already_exists_exception]

### 1. 查看 es 索引

```bash
# curl -u elastic:your_password -X GET http://localhost:9200/_cat/indices
```

### 2. 删除 kibana 索引

```bash
# curl -u elastic:your_password -X DELETE http://localhost:9200/.kibana*
```

## 12 429 circuit_breaking_exception Data too large

### 查询所有节点的统计信息

- curl
   ```bash
   # curl -u elastic:your_password -X GET http://localhost:9200/_nodes/stats | python -m json.tool

   # curl -u elastic:your_password -X GET http://localhost:9200/_nodes/stats | json

   # curl -u elastic:your_password -X GET http://localhost:9200/_nodes/stats -s | python -m json.tool

   # curl -u elastic:your_password -X GET http://localhost:9200/_nodes/stats -s | json
   ```

- 控制台命令行
   ```
   GET /_nodes/stats
   ```

### 异常信息

```json
{
	"error": {
		"root_cause": [{
			"type": "circuit_breaking_exception",
			"reason": "[parent] Data too large, data for [<http_request>] would be [1064145540/1014.8mb], which is larger than the limit of [1020054732/972.7mb], real usage: [1064142368/1014.8mb], new bytes reserved: [3172/3kb], usages [request=491968/480.4kb, fielddata=13010/12.7kb, in_flight_requests=3172/3kb, model_inference=0/0b, accounting=4561688/4.3mb]",
			"bytes_wanted": 1064145540,
			"bytes_limit": 1020054732,
			"durability": "PERMANENT"
		}],
		"type": "circuit_breaking_exception",
		"reason": "[parent] Data too large, data for [<http_request>] would be [1064145540/1014.8mb], which is larger than the limit of [1020054732/972.7mb], real usage: [1064142368/1014.8mb], new bytes reserved: [3172/3kb], usages [request=491968/480.4kb, fielddata=13010/12.7kb, in_flight_requests=3172/3kb, model_inference=0/0b, accounting=4561688/4.3mb]",
		"bytes_wanted": 1064145540,
		"bytes_limit": 1020054732,
		"durability": "PERMANENT"
	},
	"status": 429
}
```

***可以在节点的统计信息中搜索 1020054732/972.7mb，对应的字段为 parent.limit_size_in_bytes***

- Data too large, data for [<http_request>] would be [1064145540/1014.8mb], HTTP 请求的数据大小, 即实际需要的内存
- which is larger than the limit of [1020054732/972.7mb], ES 的内存上限，默认是 JVM 启动内存的 95%
- real usage: [1064142368/1014.8mb], ES 已经使用的内存, 即 JVM 启动的内存大小
- new bytes reserved: [3172/3kb], 当前查询需要缓存的内存大小
- bytes_wanted, 实际需要的内存
- bytes_limit, ES 的内存上限
- status, 429 代表 circuit_breaking_exception

当"缓存数据的数据量 + 当前查询需要缓存的数据量"到达熔断器限制时，就会返回 Data too large 错误，阻止用户进行操作。

### 解决方法

1. 增加 ES 的 JVM 内存大小
   ```bash
   # vim ./config/jvm.options
   -Xms2g
   -Xmx2g
   ```
2. 修改缓冲区
   ```bash
   # curl -u elastic:your_password -X PUT -H "Content-Type: application/json" http://localhost:9200/_cluster/settings -d '{"persistent":{"indices.breaker.fielddata.limit":"60%"}}'
   ```

## 13 this action would add [10] total shards, but this cluster currently has [995]/[1000] maximum shards open;

### 问题

原因是因为本次需要生成 10 个分片，但是分片数达到了上限，默认只允许 1000 个分片，问题是因为集群分片数不足引起的。

### 分析

从 Elasticsearch v7.0.0 开始，集群中的每个节点默认限制 1000 个 shard，如果你的 es 集群有 2 个数据节点，那么最多 2000 shards。

```
Elasticsearch 中的数据组织成索引。每一个索引由一个或多个分片组成。每个分片是 Luncene 索引的一个实例，你可以把实例理解成自管理的搜索引擎，用于在 Elasticsearch 集群中对一部分数据进行索引和处理查询。
```

当然并不是分片数越多越好，一般分片数多少与内存挂钩，您可以在集群节点上保存的分片数量与您可用的堆内存大小成正比。

一个很好的经验法则是：

确保每个节点的分片数量保持在低于每 1GB 堆内存对应集群的分片在 20-25 之间。

因此，具有 30GB 堆内存的节点最多可以有 600-750 个分片，但是进一步低于此限制，您可以保持更好。 这通常会帮助群体保持处于健康状态。

```
提示：避免有非常大的分片，因为大的分片可能会对集群从故障中恢复的能力产生负面影响。
对于多大的分片没有固定的限制，但是分片大小为 50GB 通常被界定为适用于各种用例的限制。
```

### 解决办法

```bash
# curl -u elastic:password -XPUT -H "Content-Type:application/json" http://localhost:9200/_cluster/settings -d '{"transient":{"cluster":{"max_shards_per_node":2000}}}'
{"acknowledged":true,"persistent":{},"transient":{"cluster":{"max_shards_per_node":"2000"}}}

# curl -XGET -u elastic:password http://localhost:9200/_cluster/settings
{"persistent":{},"transient":{"cluster":{"max_shards_per_node":"2000"}}}
```

## 14 TOO_MANY_REQUESTS/12/disk usage exceeded flood-stage watermark, index has read-only-allow-delete block

### 原因

因为一次请求中批量插入的数据条数巨多，以及短时间内的请求次数巨多引起ES节点服务器内存超过限制，ES主动给索引上锁。

### 解决办法

```bash
curl -u elastic:password -H "Content-type: application/json" -X PUT http://localhost:9200/_all/_settings -d '{ "index.blocks.read_only_allow_delete": null }'
```

## 15 Not enough space

### 解决方法

修改 ES 的 JVM 内存大小

```bash
# vim ./config/jvm.options
-Xms2g
-Xmx2g
```

## 16 jar hell

报错如下:

```
fatal exception while booting Elasticsearchjava.lang.IllegalStateException: jar hell!
class: sun.applet.AppletSecurity
jar1: /usr/java/jdk1.8.0_151/jre/lib/rt.jar
jar2: /usr/java/jdk1.8.0_151/lib/tools.jar
	at org.elasticsearch.base@8.4.1/org.elasticsearch.jdk.JarHell.checkClass(JarHell.java:315)
	at org.elasticsearch.base@8.4.1/org.elasticsearch.jdk.JarHell.checkJarHell(JarHell.java:233)
	at org.elasticsearch.base@8.4.1/org.elasticsearch.jdk.JarHell.checkJarHell(JarHell.java:84)
	at org.elasticsearch.server@8.4.1/org.elasticsearch.bootstrap.Elasticsearch.initPhase2(Elasticsearch.java:180)
	at org.elasticsearch.server@8.4.1/org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:66)
```

如果是 ```jdk1.8```，并且在 ```/etc/profile``` 文件中配置如下：

```
export JAVA_HOME=/usr/local/jdk1.8.0_333
export JRE_HOME=/$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```

只需要将 ```CLASSPATH ``` 修改为 ```export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH```:

```
# vim /etc/profile
export JAVA_HOME=/usr/local/jdk1.8.0_333
export JRE_HOME=/$JAVA_HOME/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin

# source /etc/profile
```
