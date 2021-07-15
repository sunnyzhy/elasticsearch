# FAQ

## 1. java.lang.IllegalStateException: failed to obtain node locks, tried [[/usr/local/elasticsearch/data/elasticsearch]] with lock id [0]; maybe these locations are not writable or multiple nodes were started without increasing [node.max_local_storage_nodes]

```bash
$ cd /usr/local/elasticsearch/data

$ rm -rf nodes
```

## 2. max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]

```bash
$ su root
Password: 

# vim /etc/security/limits.conf
* soft nofile 65536
* hard nofile 131072
```

## 3. max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

```bash
$ su root
Password: 

# vim /etc/sysctl.conf
vm.max_map_count=655360

# sysctl -p
vm.max_map_count = 655360
```

## 4. elasticsearch status red

```bash
# curl GET http://localhost:9200/_cluster/health?level=indices

# curl -XDELETE http://localhost:9200/red_status_index
```

## 5. Result window is too large, from + size must be less than or equal to: [10000] but was [10050].

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

## 6. totalHits 最大值是 10000

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

## 7. must filter should must_not的正确用法

[Bool Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html "Elasticsearch Reference [7.2] » Bool Query")

**特别注意**

- filter

   The clause (query) must appear in matching documents. **However unlike must the score of the query will be ignored.** Filter clauses are executed in filter context, meaning that scoring is ignored and clauses are considered for caching.

- should

   should is roughly equivalent to the boolean OR. **Note that should isn't exactly like a boolean OR,** but we can use it to that effect. 

```
因为 should context 下的查询语句可以一个都不满足，所以务必结合 minimum_should_match 使用
```

## 8. 查看分词结果

[Term Vectors](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-termvectors.html "Elasticsearch Reference [7.2] » Term Vectors")

```
GET /${index}/_termvectors/${id}?fields=${fields}
```

## 9. An HTTP line is larger than 4096 bytes

{"type":"too_long_frame_exception","reason":"An HTTP line is larger than 4096 bytes."}，默认情况下ES对请求参数设置为4K，如果遇到请求参数长度限制可以在elasticsearch.yml中修改如下参数：

```yml
http.max_initial_line_length: "8k"

http.max_header_size: "16k"
```

[官网配置说明](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-http.html)

## 10. Kibana启动报错：[resource_already_exists_exception]

### 1. 查看 es 索引

```bash
# curl -u elastic:your_password -X GET http://localhost:9200/_cat/indices
```

### 2. 删除 kibana 索引

```bash
# curl -u elastic:your_password -X DELETE http://localhost:9200/.kibana*
```

## 11. 429 circuit_breaking_exception Data too large

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
