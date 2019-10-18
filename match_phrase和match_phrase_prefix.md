# 编辑模板
```
PUT /_template/template_test
{
	"index_patterns": "test_*",
	"mappings": {
		"_source": {
			"enabled": true
		},
		"properties": {
			"message": {
				"type": "text",
				"analyzer": "ik_max_word",
				"search_analyzer": "ik_max_word",
				"index": true
			}
		}
	}
}
```

# 添加测试数据
```
POST /test_1/_doc
{
  "message": "I like swimming and riding!"
}

POST /test_1/_doc
{
  "message": "I like swift."
}

POST /test_1/_doc
{
  "message": "I like smile."
}
```

## 查询
```
GET /test_1/_search
{
  "query": {
    "match_all": {}
  }
}
```

## 返回
```
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "test_1",
        "_type" : "_doc",
        "_id" : "RAfZ3W0B7_Xb3VtLJWkY",
        "_score" : 1.0,
        "_source" : {
          "message" : "I like swimming and riding!"
        }
      },
      {
        "_index" : "test_1",
        "_type" : "_doc",
        "_id" : "kwfa3W0B7_Xb3VtLg2mL",
        "_score" : 1.0,
        "_source" : {
          "message" : "I like swift."
        }
      },
      {
        "_index" : "test_1",
        "_type" : "_doc",
        "_id" : "QAf53W0B7_Xb3VtLyHRP",
        "_score" : 1.0,
        "_source" : {
          "message" : "I like smile."
        }
      }
    ]
  }
}
```

# match_phrase
## 查询1
```
GET /test_1/_search
{
  "query": {
    "match_phrase": {
      "message": "I like swimming"
    }
  }
}
```

## 返回1
```
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0577903,
    "hits" : [
      {
        "_index" : "test_1",
        "_type" : "_doc",
        "_id" : "RAfZ3W0B7_Xb3VtLJWkY",
        "_score" : 1.0577903,
        "_source" : {
          "message" : "I like swimming and riding!"
        }
      }
    ]
  }
}
```

## 查询2
```
GET /test_1/_search
{
  "query": {
    "match_phrase": {
      "message": "I like swi"
    }
  }
}
```

## 返回2
```
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
```

# match_phrase_prefix
match_phrase_prefix 和 match_phrase 用法是一样的，区别就在于它允许对最后一个词条前缀匹配。

## 查询1
```
GET /test_1/_search
{
  "query": {
    "match_phrase_prefix": {
      "message": "I like swi"
    }
  }
}
```

## 返回1
```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 2.4440846,
    "hits" : [
      {
        "_index" : "test_1",
        "_type" : "_doc",
        "_id" : "RAfZ3W0B7_Xb3VtLJWkY",
        "_score" : 2.4440846,
        "_source" : {
          "message" : "I like swimming and riding!"
        }
      },
      {
        "_index" : "test_1",
        "_type" : "_doc",
        "_id" : "kwfa3W0B7_Xb3VtLg2mL",
        "_score" : 2.4440846,
        "_source" : {
          "message" : "I like swift."
        }
      }
    ]
  }
}
```

## 查询2
```
GET /test_1/_search
{
  "query": {
    "match_phrase_prefix": {
      "message": "I like swimming"
    }
  }
}
```

## 返回2
```
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0577903,
    "hits" : [
      {
        "_index" : "test_1",
        "_type" : "_doc",
        "_id" : "RAfZ3W0B7_Xb3VtLJWkY",
        "_score" : 1.0577903,
        "_source" : {
          "message" : "I like swimming and riding!"
        }
      }
    ]
  }
}
```
