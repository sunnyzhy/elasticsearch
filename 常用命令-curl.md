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

### 索引

查看全部索引:

```bash
curl -k -u elastic:{password} https://localhost:9200/_cat/indices?v
```

查看以索引前缀开头的索引:

```bash
curl -k -u elastic:{password} https://localhost:9200/_cat/indices/{index_prefix}*?v
```

### 模板

查看全部的索引模板:

```bash
curl -k -u elastic:{password} https://localhost:9200/_cat/templates?v
```

查看某个索引模板的内容:

```bash
curl -k -u elastic:{password} https://localhost:9200/_index_template/{template}?pretty
```

## 索引

获取索引:

```bash
curl -k -u elastic:{password} https://localhost:9200/{index}/_doc/{id}?pretty
```

添加数据:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" -XPOST https://localhost:9200/{index}/_doc/{id}?pretty -d'{"a": "a","b": "b"}'
```

修改数据:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" -XPOST https://localhost:9200/{index}/_update/{id}?pretty -d'{"doc":{"a": "a","b": "b"}}'
```

删除索引:

```bash
curl -k -u elastic:{password} -XDELETE https://localhost:9200/{index}/_doc/{id}?pretty
```

模糊查询:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" https://localhost:9200/{index}/_search?pretty -d '{"query":{"match":{"name":"xx"}}}'
```

精确查询:

```bash
curl -k -u elastic:{password} -H "Content-Type:application/json" https://localhost:9200/{index}/_search?pretty -d '{"query":{"term":{"name":"xx"}}}'
```

获取 mapping:

```bash
curl -k -u elastic:{password} https://localhost:9200/{index}/_mapping?pretty
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
