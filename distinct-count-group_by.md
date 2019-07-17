# distinct
```sql
SELECT DISTINCT(user_name) FROM user WHERE tenant_id = '9c71ab9d8b7440249acdebd01cc7db88';
```
```
GET /user/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "tenant_id": "9c71ab9d8b7440249acdebd01cc7db88"
        }
      }
    }
  },
  "collapse": {
    "field": "user_name"
  }
}
```
```
    "hits": [
      {
        "_index" : "user",
        "_type" : "_doc",
        "_id" : "45if1WsBJTe7vRlriSTz",
        "_score" : 0.0,
        "_source" : {
          "tenant_id" : "9c71ab9d8b7440249acdebd01cc7db88",
          "user_name" : "kate",
          "user_type" : 3
        },
        "fields" : {
          "user_name" : [
            "kate"
          ]
        }
      }
    ]
```

总结：使用 collapse 字段后，查询结果中 [hits] 中会出现 [fields] 字段，其中包含了去重后的 user_name 。

# count + distinct
```sql
SELECT COUNT(DISTINCT(user_name)) FROM user WHERE tenant_id = '9c71ab9d8b7440249acdebd01cc7db88';
```
```
GET /user/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "tenant_id": "9c71ab9d8b7440249acdebd01cc7db88"
        }
      }
    }
  },
  "from": 0,
  "size": 0, 
  "aggs": {
    "count": { // 聚合名称
      "cardinality": { // 等价于distinct
        "field": "user_name"
      }
    }
  }
}
```
```
  "aggregations" : {
    "count" : {
      "value" : 10
    }
  }
```

# count + group by
```sql
SELECT COUNT(*) FROM user GROUP BY user_type;
```
```
GET /user/_search
{
  "from": 0,
  "size": 0, 
  "aggs": {
    "user_type": { // 聚合名称
      "terms": { // 等价于group by
        "field": "user_type"
      }
    }
  }
}
```
```
  "aggregations" : {
    "user_type" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "普通用户",
          "doc_count" : 1000483
        },
        {
          "key" : "管理员",
          "doc_count" : 1261
        },
        {
          "key" : "超级管理员",
          "doc_count" : 255
        }
      ]
    }
  }
```

# count + distinct + group by
```sql
SELECT COUNT(DISTINCT(user_name)) FROM user GROUP BY user_type;
```
```
GET /user/_search
{
  "from": 0,
  "size": 0, 
  "aggs": {
    "user_type": { // 聚合名称
      "terms": { // 等价于group by
        "field": "user_type"
      },
      "aggs": { // 子聚合
        "count": { // 聚合名称
          "cardinality": { // 等价于distinct
            "field": "user_name"
          }
        }
      }
    }
  }
}
```
```
  "aggregations" : {
    "user_type" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "普通用户",
          "doc_count" : 1000483, //去重前数据1000483条
          "count" : {
            "value" : 10 //去重后数据10条
          }
        },
        {
          "key" : "管理员",
          "doc_count" : 1261,
          "count" : {
            "value" : 10
          }
        },
        {
          "key" : "超级管理员",
          "doc_count" : 255,
          "count" : {
            "value" : 10
          }
        }
      ]
    }
  }
```

# count + distinct + group by + where
```sql
SELECT COUNT(DISTINCT(user_name)) FROM user WHERE tenant_id = '9c71ab9d8b7440249acdebd01cc7db88' GROUP BY user_type;
```
```
GET /user/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "tenant_id": "9c71ab9d8b7440249acdebd01cc7db88"
        }
      }
    }
  },
  "from": 0,
  "size": 0, 
  "aggs": {
    "user_type": {
      "terms": {
        "field": "user_type"
      },
      "aggs": {
        "count": {
          "cardinality": {
            "field": "user_name"
          }
        }
      }
    }
  }
}
```
```
  "aggregations" : {
    "user_type" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "普通用户",
          "doc_count" : 1000483, //去重前数据1000483条
          "count" : {
            "value" : 10 //去重后数据10条
          }
        },
        {
          "key" : "管理员",
          "doc_count" : 1261,
          "count" : {
            "value" : 10
          }
        },
        {
          "key" : "超级管理员",
          "doc_count" : 255,
          "count" : {
            "value" : 10
          }
        }
      ]
    }
  }
  ```

# 注意事项
## collapse关键字
1. 折叠功能 ES5.3 版本之后才发布的

2. 聚合 & 折叠只能针对 keyword 类型有效
