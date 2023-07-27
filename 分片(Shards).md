# 分片（Shards）

## 分片重要性
ES 中所有的数据均衡地存储在集群中各个节点的分片中，直接影响 ES 的性能、安全和稳定性。

## 分片概述
分片是 ES 中所有数据的文件块，也是数据的最小单元块，整个 ES 集群的核心就是对所有分片的分布、索引、负载、路由等。

### 主分片（primary shard）与副本分片（replica shard）

- ```index``` 包含多个 ```shard```（分片）
- 每个 ```shard``` 都是一个最小工作单元，承载部分数据，lucene 实例，完整的建立索引和处理请求的能力
- 增减节点时，```shard``` 会自动在 ```nodes``` 中负载均衡
- ```primary shard``` 和 ```replica shard```，每个 document 只存在于某一个 ```primary shard``` 以及其对应的 ```replica shard``` 中，不可能存在于多个 ```primary shard``` 中
- ```replica shard``` 是 ```primary shard``` 的副本，负责容错，以及承担读请求负载
- ```primary shard``` 的数量在创建索引的时候就固定了，```replica shard``` 的数量可以随时修改
- ```primary shard``` 默认是 ```5```，```replica shard``` 默认是 ```1```。默认有 ```10``` 个 ```shard```，则包含 ```5``` 个 ```primary shard``` 和 ```5``` 个 ```replica shard```
- ```primary shard``` 不能和自己的 ```replica shard``` 放在同一个节点上（否则节点宕机，```primary shard``` 和 ```replica shard``` 都丢失，起不到容错的作用），但是可以和其他 ```primary shard``` 的replica shard 放在同一个节点上

### 分片数量（number_of_shards）与副本数量（number_of_replicas）

- ```number_of_shards```：每个索引有多少个分片，只能在创建索引时指定，后期无法修改
- ```number_of_replicas```：每个分片有多少个副本，后期可以动态修改

一般情况下，副本的数量根据 nodes 个数而定，比如只有一个 node，则副本分片的数量设置为 0；有两个 node ，则副本分片的数量设置为 1

```
实列场景：

假设 IndexA 有 2 个分片，向 IndexA 中插入 10 条数据 (10 个文档)，那么这 10 条数据会尽可能平均的分为 5 条存储在第一个分片，剩下的 5 条会存储在另一个分片中。
```

## 分片设置

可以在***创建索引***的时候指定主分片和副本分片的数量：

```bash
PUT indexName
{
    "settings": {
        "number_of_shards": 5,
        "number_of_replicas" : 1
    }
}
```

修改副本分片的数量：

```bash
curl -H "Content-Type: application/json" -XPUT -u elastic:password -k 'https://localhost:9200/_all/_settings' -d '
{
	"index": {
		"number_of_replicas": 0
	}
}'
```

但是还存在一个问题，下次创建新的索引时 ```number_of_replicas``` 仍然是 ```1```，此时就需要把模板里的 ```number_of_replicas``` 设置为 ```1``` 然后更新模板。

```
注意

索引建立后，分片个数是不可以更改的
```

## 分片个数（数据节点计算）
分片个数是越多越好，还是越少越好了？根据整个索引的数据量来判断。

```
实列场景：

如果 IndexA 所有数据文件大小是300G，改怎么定制方案？(可以通过Head插件来查看)

建议：（仅参考）

   1、每一个分片数据文件小于30GB

   2、每一个索引中的一个分片对应一个节点

   3、节点数大于等于分片数
```

根据建议，至少需要 10 个分片。

结果： 建 10 个节点 (Node)，Mapping 指定分片数为 10，满足每一个节点一个分片，每一个分片数据带下在30G左右。

**SN(分片数) = IS(索引大小) / 30**

**NN(节点数) = SN(分片数) + MNN(主节点数[无数据]) + NNN(负载节点数)**

## 分片查询
- **randomizeacross shards**

随机选择分片查询数据，es的默认方式

- **_local**

优先在本地节点上的分片查询数据然后再去其他节点上的分片查询，本地节点没有IO问题但有可能造成负载不均问题。数据量是完整的。

- **_primary**

只在主分片中查询不去副本查，一般数据完整。

- **_primary_first**

优先在主分片中查，如果主分片挂了则去副本查，一般数据完整。

- **_only_node**

只在指定id的节点中的分片中查询，数据可能不完整。

- **_prefer_node**

优先在指定你给节点中查询，一般数据完整。

- **_shards**

在指定分片中查询，数据可能不完整。

- **_only_nodes**

可以自定义去指定的多个节点查询，es不提供此方式需要改源码。
