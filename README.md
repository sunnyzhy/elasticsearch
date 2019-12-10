# elasticsearch、kibana、elasticsearch-analysis-ik 安装须知
1. 安装路径不能有空格或中文字符
2. 各个安装包的版本必须保持一致

# Elasticsearch示例文档
[https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html "Elasticsearch示例文档")

# Elasticsearch开发文档
[https://www.elastic.co/guide/en/elasticsearch/client/index.html](https://www.elastic.co/guide/en/elasticsearch/client/index.html "Elasticsearch开发文档")

```
 java 开发建议使用 Java High Level REST Client
 ```

# 数据库与Elasticsearch结合使用

## 设计

- 创建的每个Elasticsearch索引都应该由符合ACID的数据存储支持。

- 数据库应该是真实的最终来源，从中填充索引。

- 如果异常情况发生（节点丢失，中断或误操作 ）导致丢失了索引，您将能够完全恢复它。

- 一般的用法是另外的数据库比如NOSQL里面有一份，然后实时同步到ES，这样一个用于键值查询，一个用于各种其他查询。 如果ES升级了之类的，比如数据结构变了，那么老版本数据可以不要，直接NOSQL再导入一份到新版本，还可以恢复。

- logstash的同步插件如logstash_input_jdbc 不支持同步删除操作，建议改为更新操作加标记flag，或者通过业务逻辑实现同步删除操作。

## 核心操作
- ES中只存储检索字段，方便快速检索、全文检索。

- Mysql中存储全部字段，利用ACID事务特性。

- 通过关联字段建立关联，比如：news_id在ES和mysql中要有相同的值。

- 核心数据先通过ES快速获取Id（如news_id）,再通过Mysql二次查询。
