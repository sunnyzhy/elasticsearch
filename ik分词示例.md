# 查看字段的分词结果(V7.2)
```
GET /${index}/_termvectors/${id}?fields=${fields_name}
```

# 默认分词
- 创建索引
```
PUT /index
```

- 添加文档
```
PUT /index/doc/1
{
  "title": "今日快报",
  "content": "重庆取消路桥费"
}

PUT /index/doc/2
{
  "title": "今日快报",
  "content": "巴厘岛火山喷发"
}

PUT /index/doc/3
{
  "title": "今日快报",
  "content": "偶遇奚梦瑶何献君"
}

PUT /index/doc/4
{
  "title": "今日快报",
  "content": "巴厘岛机场关闭"
}

PUT /index/doc/5
{
  "title": "今日快报",
  "content": "上海虹桥机场"
}
```

- 搜索
```
GET /index/doc/_search
{
  "query": {
    "match": {
      "content": {
        "query": "虹桥"
      }
    }
  }
}

{
  "took": 54,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.5753642,
    "hits": [
      {
        "_index": "index",
        "_type": "doc",
        "_id": "5",
        "_score": 0.5753642,
        "_source": {
          "title": "今日快报",
          "content": "上海虹桥机场"
        }
      },
      {
        "_index": "index",
        "_type": "doc",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "title": "今日快报",
          "content": "重庆取消路桥费"
        }
      }
    ]
  }
}
```

# ik分词
- 创建索引
```
PUT /index_ik
```

- 创建映射
```
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
```

- 添加文档
```
PUT /index_ik/doc/1
{
  "title": "今日快报",
  "content": "重庆取消路桥费"
}

PUT /index_ik/doc/2
{
  "title": "今日快报",
  "content": "巴厘岛火山喷发"
}

PUT /index_ik/doc/3
{
  "title": "今日快报",
  "content": "偶遇奚梦瑶何献君"
}

PUT /index_ik/doc/4
{
  "title": "今日快报",
  "content": "巴厘岛机场关闭"
}

PUT /index_ik/doc/5
{
  "title": "今日快报",
  "content": "上海虹桥机场"
}
```

- 搜索
```
GET /index_ik/doc/_search
{
  "query": {
    "match": {
      "content": {
        "query": "虹桥"
      }
    }
  }
}

{
  "took": 46,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "index_ik",
        "_type": "doc",
        "_id": "5",
        "_score": 0.2876821,
        "_source": {
          "title": "今日快报",
          "content": "上海虹桥机场"
        }
      }
    ]
  }
}
```
