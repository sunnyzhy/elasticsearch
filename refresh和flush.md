# refresh
当我们向 ES发送请求的时候，我们发现 es 貌似可以在我们发请求的同时进行搜索。而这个实时建索引并可以被搜索的过程实际上是一次 es 索引提交（commit）的过程，如果这个提交的过程直接将数据写入磁盘（fsync）必然会影响性能，所以 es 中设计了一种机制，即：先将 index buffer 中文档（document）解析完成的 segment 写到 filesystem cache 之中，这样避免了比较损耗性能 io 操作，又可以使 document 可以被搜索。以上从 index buffer 中取数据到 filesystem cache 中的过程叫做 refresh 。

![](images/refresh&flush.jpg)

refresh 操作可以通过 API 设置：
```
POST /index/_settings
{
  "refresh_interval": "10s"
}
```

当我们进行大规模的创建索引操作的时候，最好将 refresh 关闭：
```
POST /index/_settings
{
  "refresh_interval": "-1"
}
```
 
es 默认的 refresh 间隔时间是 1s ，这也是为什么 ES 可以进行近乎实时的搜索。
 
# flush
我们可能已经意识到如果数据在 filesystem cache 之中是很有可能在意外的故障中丢失。这个时候就需要一种机制，可以将对 es 的操作记录下来，来确保当出现故障的时候，保留在 filesystem 的数据不会丢失，并在重启的时候可以从这个记录中将数据恢复过来。 elasticsearch 提供了 translog 来记录这些操作。
当向 elasticsearch 发送创建 document 索引请求的时候， document 数据会先进入到 index buffer 之后，与此同时会将操作记录在 translog 之中，当发生 refresh 时（数据从 index buffer 中进入 filesystem cache 的过程）translog 中的操作记录并不会被清除，而是当数据从 filesystem cache 中被写入磁盘之后才会将 translog 中清空。而从 filesystem cache 写入磁盘的过程就是 flush 。

## translog 的功能
1. 保证在 filesystem cache 中的数据不会因为 elasticsearch 重启或是发生意外故障的时候丢失。
2. 当系统重启时会从 translog 中恢复之前记录的操作。
3. 当对 elasticsearch 进行 CRUD 操作的时候，会先到 translog 之中进行查找，因为 tranlog 之中保存的是最新的数据。
4. translog 的清除时间时进行 flush 操作之后（将数据从 filesystem cache 刷入 disk 之中）。
 
 
## flush 操作的时间点
1. es 的各个 shard 会每个 30 分钟进行一次 flush 操作。
2. 当 translog 的数据达到某个上限的时候会进行一次 flush 操作。

## 关于translog和flush的一些配置项
- index.translog.flush_threshold_ops:当发生多少次操作时进行一次flush，默认是 unlimited。

- index.translog.flush_threshold_size:当translog的大小达到此值时会进行一次flush操作，默认是512mb。

- index.translog.flush_threshold_period:在指定的时间间隔内如果没有进行flush操作，会进行一次强制flush操作，默认是30m。

- index.translog.interval:多少时间间隔内会检查一次translog，来进行一次flush操作。es会随机的在这个值到这个值的2倍大小之间进行一次操作，默认是5s。
