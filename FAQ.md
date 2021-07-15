# FAQ

## java.lang.IllegalStateException: failed to obtain node locks, tried [[/usr/local/elasticsearch/data/elasticsearch]] with lock id [0]; maybe these locations are not writable or multiple nodes were started without increasing [node.max_local_storage_nodes]

```bash
$ cd /usr/local/elasticsearch/data

$ rm -rf nodes
```

## max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]

```bash
$ su root
Password: 

# vim /etc/security/limits.conf
* soft nofile 65536
* hard nofile 131072
```

## max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

```bash
$ su root
Password: 

# vim /etc/sysctl.conf
vm.max_map_count=655360

# sysctl -p
vm.max_map_count = 655360
```

## elasticsearch status red

```bash
# curl GET http://localhost:9200/_cluster/health?level=indices

# curl -XDELETE http://localhost:9200/red_status_index
```

## 429 circuit_breaking_exception Data too large

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
   # curl -u elastic:password -X PUT -H "Content-Type: application/json" http://localhost:9200/_cluster/settings -d '{"persistent":{"indices.breaker.fielddata.limit":"60%"}}'
   ```
