# ElasticSearch reindex 超时 "error": "Gateway Time-out", "message": "Client request timeout"

## 现象
```
{
	"statusCode": 504,
	"error": "Gateway Time-out",
	"message": "Client request timeout"
}
```

## 解决方法
### 设置 ES 副本数为 0 && 增加 refresh 间隔
```
PUT /index_name/_settings
{
     "number_of_replicas": 0,
     "refresh_interval": -1
}
```

### 借助 scroll 的 sliced 提升效率
```
POST /_reindex?slices=5&refresh
{
  "source": {
    "index": "index_name",
     "size": 10000
  }, 
  "dest": {
    "index": "index_name_1"
  }
}
```

slices大小设置注意事项：

1. slices大小的设置可以手动指定，或者设置slices设置为auto，auto的含义是：针对单索引，slices大小 = 分片数；针对多索引，slices=分片的最小值。

2. 当slices的数量等于索引中的分片数量时，查询性能最高效。slices大小大于分片数，非但不会提升效率，反而会增加开销。

3. 如果这个slices数字很大(例如500)，建议选择一个较低的数字，因为过大的slices 会影响性能。

### 重置 ES 副本数和 refresh 间隔
```
PUT /index_name/_settings
{
     "number_of_replicas": 1,
     "refresh_interval": "1s"
}
```
