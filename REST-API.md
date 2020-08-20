# Delete
## Delete API
```
DELETE /<index>/_doc/<_id>
```

等价于以下SQL

```sql
DELETE FROM table_name WHERE id = _id;
```

## Delete by query API
```
POST /<index>/_delete_by_query
{
  "query": {
    "term": {
      "id": _id
    }
  }
}
```

等价于以下SQL

```sql
DELETE FROM table_name WHERE id = _id;
```

# Update
## Update API
```
POST /<index>/_update/_id
{
  "script" : {
    "source": "ctx._source['field1']=value1;ctx._source['field2']=value2"
  }
}
```

等价于以下SQL

```sql
UPDATE table_name SET field1 = value1, field2 = value2 WHERE id = _id;
```

## Update By Query API
```
POST /<index>/_update_by_query
{
  "script": {
    "source": "ctx._source['field1']=value1;ctx._source['field2']=value2"
  },
  "query": {
    "bool": {
      "filter": {
        "range": {
          "id": {
            "gte": _id
          }
        }
      }
    }
  }
}
```

等价于以下SQL

```sql
UPDATE table_name SET field1 = value1, field2 = value2 WHERE id >= _id;
```
