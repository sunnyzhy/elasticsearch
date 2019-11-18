# blocked by: [FORBIDDEN/12/index read-only / allow delete (api)]

## 问题原因
1. 这是由于ES新节点的数据目录data存储空间不足，导致从master主节点接收同步数据的时候失败，此时ES集群为了保护数据，会自动把索引分片index置为只读read-only

2. 即使磁盘扩容了，但是在扩容之前（也就是磁盘空间不足的时候），对索引进行了写入的操作，由于 ES 的保护机制，会将该索引置为只读状态。所以扩容之后，依旧会提示上述错误

**不管是哪种情况，都是因为存储空间不足引起的。**


## 解决方案
- 提供足够的存储空间供数据写入，如需在配置文件中更改ES数据存储目录，注意重启ES

- 修改索引的只读设置

```
PUT _settings {
	"index": {
		"blocks": {
			"read_only_allow_delete": "false"
		}
	}
}
```

## 官网建议

If your elasticsearch is responding with 403 and this message:
```
{
  "type": "cluster_block_exception",
  "reason": "blocked by: [FORBIDDEN/12/index read-only / allow delete (api)];"
}
```

Then you probably recovered from a full hard drive. The thing is, elasticsearch is switching to read-only if it cannot index more documents because your hard drive is full. With this it ensures availability for read-only queries. Elasticsearch will not switch back automatically but you can disable it by sending

```
curl -XPUT -H "Content-Type: application/json" http://localhost:9200/_all/_settings -d '{"index.blocks.read_only_allow_delete": null}'
```

**_all 可以修改为具体的索引名称**
