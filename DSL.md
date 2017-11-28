# 创建索引
PUT /index

# 添加文档
PUT /index/doc/1
{
  "title": "my title",
  "content": "my content"
}

# 搜索
GET /index/doc/_search
{
  "query": {
    "match": {
      "content": {
        "query": "content"
      }
    }
  }
}

# 创建映射
PUT /index_ik/doc/_mapping
{
  "properties": {
    "title": {
      "type": "text",
      "analyzer": "ik_max_word",
      "search_analyzer": "ik_max_word"
    },
    "content": {
      "type": "text",
      "analyzer": "ik_max_word",
      "search_analyzer": "ik_max_word"
    }
  }
}

# 索引模板
~~~
PUT /_template/my_temp
{
    "index_patterns": "temp-*",
    "order": 1,
    "settings": {
        "number_of_shards": 5
    },
    "mappings": {
        "_default_": {
            "properties": {
                "title": {
                    "type": "text",
                    "analyzer": "ik_max_word",
                    "search_analyzer": "ik_max_word"
                },
                "content": {
                    "type": "text",
                    "analyzer": "ik_max_word",
                    "search_analyzer": "ik_max_word"
                }
            }
        }
    },
    "aliases": {
        "alarm_current": {}
    }
}
~~~
