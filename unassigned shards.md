# 产生unassigned分片的原因
## 1. 集群有目的的延迟分配

当某个节点脱离集群时，主节点会暂时的延迟重分配分片，以减少重新平衡分片带来的资源浪费，这种情况下，如果源节点在一定时间（默认1分钟）内重新加入，可以恢复分片信息。这种情况的日志信息如下：

```
[TIMESTAMP][INFO][cluster.routing] [MASTER NODE NAME] delaying allocation for [54] unassigned shards, next check in [1m]
```

手动修改延迟时间：

```
curl  -H "Content-type: application/json" -X PUT localhost:9200/<INDEX_NAME>/_settings -d 
'{
    "settings": {
       "index.unassigned.node_left.delayed_timeout": "30s"
    }
}'
```

**如果需要修改所有索引的阀值，则可以使用 _all 替换 <INDEX_NAME>。**

## 2. 分片数目过多，而节点数不足（注意，多数场景均与此有关）

当节点加入和离开集群时，主节点会自动重新分配分片，确保分片的多个副本未分配给同一节点。换句话说，**主节点不会将主分片和副本分片分配至同一个节点**，同样，也不会将同一分片的两个副本节点分配到同一个节点，所以当没有足够的节点分配分片时，会出现未分配的状态。

要避免此问题，请确保使用以下公式初始化集群中的每个索引，每个主分片的副本数少于集群中的节点数：

```
N >= R + 1
```

其中 N 是集群中的节点数，R 是集群中所有索引的最大分片复制因子。

可以向集群添加更多数据节点或减少副本数。例如减少副本数：

```
curl -H "Content-type: application/json" -X PUT localhost:9200/<INDEX_NAME>/_settings -d
'{
    "number_of_replicas": 2
}'
```

## 3. 加入一个新的节点，需要重新启用分片分配
分片重分配默认是开启的，但是可能因为某些原因关闭了重分配但是忘记开启了。

开启重分配：

curl -H "Content-type: application/json" -X PUT localhost:9200/_cluster/settings -d
'{ "transient":
    { "cluster.routing.allocation.enable" : "all" 
    }
}'

## 4. 集群中分片数据已不存在

处理方法：

1. 恢复存有0分片的源节点，并加入到集群中（不强制重新分配主分片）

2. 使用Reroute API强制重分配分片

```
curl -H "Content-type: application/json" -X POST localhost:9200/_cluster/reroute -d
'{ "commands" :
  [ { "allocate_empty_primary" :
      { "index" :"constant-updates", "shard" : 0, "node":"<NODE_NAME>", "accept_data_loss": "true" }
  }]
}'
```

3. 从原始数据重建索引或者从备份快照中恢复

## 5. 磁盘空间不足
一般情况下，当磁盘利用率达到 85% 时，主节点将不再分配分片至该节点上。

可以使用如下命令查看磁盘利用率：

```
curl -s 'localhost:9200/_cat/allocation?v'
```

如果磁盘空间比较大，而 85% 利用率有些浪费，可以通过设置
cluster.routing.allocation.disk.watermark.low 和/或 cluster.routing.allocation.disk.watermark.high 来增加该值：

```
curl -H "Content-type: application/json" -X PUT localhost:9200/_cluster/settings -d
'{
    "transient": { 
     "cluster.routing.allocation.disk.watermark.low":"90%"   
    }
}'
```

注：如果需要集群重启有效，可将 transient 改为 persistent；ES 设置中百分比多指已使用空间，字节值多指未使用空间

## 6. 多版本问题
ES集群中存在多版本ES，导致不兼容问题

# 定位未分配的分片
## 1. 查看集群健康状态
```
curl -X GET localhost:9200/_cluster/health?pretty
{
  "cluster_name" : "zz",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 54,
  "active_shards" : 54,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 50,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 51.92307692307693
}
```

可以看出，有 50 个 unassigned 的分片。

## 2. 定位 unassigned 分片的位置

```
curl -X GET localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason| grep UNASSIGNED
```

每行列出索引的名称、分片编号、是主分片 p 还是副本分片 r、其未分配的原因。

## 3. 查看未分配分片的具体原因

```
curl -X GET localhost:9200/_cluster/allocation/explain?pretty
```

### 3.1 已删除的索引

可以直接使用删除命令删除索引：

```
curl -X DELETE localhost:9200/<INDEX_NAME>
```

### 3.2 使用中的索引

- 设置单个索引的副本为 0 ：

```
curl -H "Content-type: application/json" -X PUT localhost:9200/<INDEX_NAME>/_settings -d
'{ 
    "index" : { 
        "number_of_replicas" : 0
    } 
}'
```

压缩版: 
```
curl -H "Content-type: application/json" -X PUT localhost:9200/<INDEX_NAME>/_settings -d '{ "index" : { "number_of_replicas" : 0 } }'
```

- 设置所有索引的副本为 0 :

```
curl -H "Content-type: application/json" -X PUT localhost:9200/_settings -d
'{ 
    "index" : { 
        "number_of_replicas" : 0
    } 
}'
```

压缩版: 
```
curl -H "Content-type: application/json" -X PUT localhost:9200/_settings -d '{ "index" : { "number_of_replicas" : 0 } }'
```
