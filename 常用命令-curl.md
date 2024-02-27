# 常用命令 - curl

- ```?v```: 格式化输出的内容，并显示表头
- ```?pretty```: 格式化输出的内容

## ```_cat```

### 查看可用的 API

```bash
# curl -k -u elastic:{password} https://localhost:9200/_cat?pretty
=^.^=
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/tasks
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/thread_pool/{thread_pools}
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
/_cat/nodeattrs
/_cat/repositories
/_cat/snapshots/{repository}
/_cat/templates
/_cat/component_templates/_cat/transforms
/_cat/transforms/{transform_id}
/_cat/ml/anomaly_detectors
/_cat/ml/anomaly_detectors/{job_id}
/_cat/ml/trained_models
/_cat/ml/trained_models/{model_id}
/_cat/ml/datafeeds
/_cat/ml/datafeeds/{datafeed_id}
/_cat/ml/data_frame/analytics
/_cat/ml/data_frame/analytics/{id}
```

### 健康状态

```bash
curl -k -u elastic:{password} https://localhost:9200/_cat/health?v
```

### 节点

```bash
curl -k -u elastic:{password} https://localhost:9200/_cat/nodes?v
```

## 模板

查看全部的索引模板:

```bash
curl -k -u elastic:{password} https://localhost:9200/_cat/templates?v
```

查看以索引模板前缀开头的索引模板:

```bash
curl -k -u elastic:{password} https://localhost:9200/_cat/templates/{template_prefix}*?v
```

查看某个索引模板的内容:

```bash
curl -k -u elastic:{password} https://localhost:9200/_index_template/{template}?pretty
```

创建某个索引模板：

```bash
curl -k -u elastic:{password} -XPUT https://localhost:9200/_index_template/{template}?pretty -H 'Content-Type: application/json' -d '
{
	"index_patterns": ["log_*"],
	"template": {
		"settings": {
			"number_of_shards": 5,
			"number_of_replicas": 1
		},
		"mappings": {
			"_source": {
				"enabled": true
			},
			"properties": {
				"id": {
					"type": "integer",
					"index": true
				},
				"name": {
					"type": "text",
					"index": true
				}
			}
		}
	}
}'
```

删除某个索引模板:

```bash
curl -k -u elastic:{password} -XDELETE https://localhost:9200/_index_template/{template}?pretty
```

批量删除索引模板:

```bash
curl -k -u elastic:{password} -XDELETE https://localhost:9200/_index_template/{template1},{template2},{template3}?pretty
```

批量删除索引模板（shell 脚本）:

```bash
#!/bin/sh

array=($(curl -k -u elastic:{password} https://localhost:9200/_cat/templates/{template_prefix}*?v | awk 'NR>1{print $1}'))

for i in ${array[@]};do
  echo $i
  curl -k -u elastic:{password} -XDELETE https://localhost:9200/_index_template/$i?pretty
done
```

## 索引

创建索引:

```bash
curl -k -u elastic:{password} -XPUT https://localhost:9200/{index}?pretty
```

查看全部索引:

```bash
curl -k -u elastic:{password} https://localhost:9200/_cat/indices?v
```

查看以索引前缀开头的索引:

```bash
curl -k -u elastic:{password} https://localhost:9200/_cat/indices/{index_prefix}*?v
```

查看单个索引信息:

```bash
curl -k -u elastic:{password} -XGET https://localhost:9200/{index}?pretty
```

删除索引:

```bash
curl -k -u elastic:{password} -XDELETE https://localhost:9200/{index}?pretty
```

批量删除索引:

```bash
curl -k -u elastic:{password} -XDELETE https://localhost:9200/{index1},{index2},{index3}?pretty
```

批量删除索引（shell 脚本）:

```bash
#!/bin/sh

array=($(curl -k -u elastic:{password} https://localhost:9200/_cat/indices/{index_prefix}*?v | awk 'NR>1{print $3}'))

for i in ${array[@]};do
  echo $i
  curl -k -u elastic:{password} -XDELETE https://localhost:9200/$i?pretty
done
```

获取 mapping:

```bash
curl -k -u elastic:{password} https://localhost:9200/{index}/_mapping?pretty
```

添加数据:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" -XPOST https://localhost:9200/{index}/_doc/{id}?pretty -d'{"a": "a","b": "b"}'
```

修改数据:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" -XPOST https://localhost:9200/{index}/_update/{id}?pretty -d'{"doc":{"a": "a","b": "b"}}'
```

删除数据:

```bash
curl -k -u elastic:{password} -XDELETE https://localhost:9200/{index}/_doc/{id}?pretty
```

条件删除:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" -XPOST https://localhost:9200/{index}/_doc/_delete_by_query?pretty -d '{"query":{"match":{"name":"xx"}}}'
```

查询指定 id 的数据:

```bash
curl -k -u elastic:{password} https://localhost:9200/{index}/_doc/{id}?pretty
```

查询指定所有库的所有数据:

```bash
curl -k -u elastic:{password} -XGET https://localhost:9200/{index}/_search?pretty
```

查询指定索引库的所有数据记录的name值:

```bash
curl -k -u elastic:{password} -XGET https://localhost:9200/{index}/_search?_source=name?pretty
```

查询指定 id 的数据记录的 name 值:

```bash
curl -k -u elastic:{password} -XGET https://localhost:9200/{index}/{id}?_source=name&pretty
```

查询所有数据:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" -XGET https://localhost:9200/{index}/_search?pretty -d '{"query":{"match_all":{}}}'
```

指定条数:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" -XGET https://localhost:9200/{index}/_search?pretty -d '{"query":{"match_all":{}},"size":2}'
```

分页查询:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" -XGET https://localhost:9200/{index}/_search?pretty -d '{"query":{"match_all":{}},"from":0,"size":2}'
```

查询指定的列:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" -XGET https://localhost:9200/{index}/_search?pretty -d '{"query":{"match_all":{}},"_source":["name","id"]}'
```

排序:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" -XGET https://localhost:9200/{index}/_search?pretty -d '{"query":{"match_all":{}},"sort":{"price":{"order":"desc"}}}'
```

模糊查询:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" https://localhost:9200/{index}/_search?pretty -d '{"query":{"match":{"name":"xx"}}}'
```

多条件模糊查询:

- bool must: 必须满足的条件
- must_not: 必须不能满足的条件
- should: 应该，可有可无

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" -XGET https://localhost:9200/{index}/_search?pretty -d '{"query":{"bool":{"must":{"match":{"name":"xx"}}}}}'
```

精确查询:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" https://localhost:9200/{index}/_search?pretty -d '{"query":{"term":{"name":"xx"}}}'
```

精确匹配多个词:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" -XGET https://localhost:9200/{index}/_search?pretty -d '{"query":{"terms":{"name":["xx","yy"]}}}'
```

范围查询:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" -XGET https://localhost:9200/{index}/_search?pretty -d '{"query":{"range":{"age":{"gt":"20","lte":"25"}}}}'
```

count 查询:

```bash
curl -k -u elastic:{password} -XGET https://localhost:9200/{index}/_count
```

## ```_cluster```

查看集群的健康状态:

```bash
curl -k -u elastic:{password} https://localhost:9200/_cluster/health?pretty
```

查看集群的详细信息:

包括节点、分片等。

```bash
curl -k -u elastic:{password} https://localhost:9200/_cluster/state?pretty
```

查看集群的系统信息:

包括 CPU、JVM 等等。

```bash
curl -k -u elastic:{password} https://localhost:9200/_cluster/stats?pretty
```

## ```_nodes```

查询节点的状态:

```bash
curl -k -u elastic:{password} https://localhost:9200/_nodes?pretty
```
