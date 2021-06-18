# REST APIs

## Search APIs

### Search API
- Request
   ```
   GET /<target>/_search
   
   GET /_search
   
   POST /<target>/_search
   
   POST /_search
   ```

- Examples
   ```json
   GET /my-index-000001/_search?from=40&size=20
   {
     "query": {
       "term": {
         "user.id": "kimchy"
       }
     }
   }
   ```

### Count API
- Request
   ```
   GET /<target>/_count
   ```

- Examples
   ```json
   GET /my-index-000001/_count
   {
     "query" : {
       "term" : { "user.id" : "kimchy" }
     }
   }
   ```

## Document APIs

### Index API
- Request
   ```
   PUT /<target>/_doc/<_id>
   
   POST /<target>/_doc/
   
   PUT /<target>/_create/<_id>
   
   POST /<target>/_create/<_id>
   ```

- Examples
   ```json
   POST my-index-000001/_doc/
   {
     "@timestamp": "2099-11-15T13:12:00",
     "message": "GET /search HTTP/1.1 200 1070000",
     "user": {
       "id": "kimchy"
     }
   }
   ```

### Get API
- Request
   ```
   GET <index>/_doc/<_id>

   HEAD <index>/_doc/<_id>

   GET <index>/_source/<_id>

   HEAD <index>/_source/<_id>
   ```

- Examples
   ```json
   GET my-index-000001/_doc/0
   ```
  
### Delete API
- Request
   ```
   DELETE /<index>/_doc/<_id>
   ```
 
- Examples
   ```json
   DELETE /my-index-000001/_doc/1
   ```

### Delete by query API
- Request
   ```
   POST /<target>/_delete_by_query
   ```
 
- Examples
   ```json
   POST /my-index-000001/_delete_by_query
   {
     "query": {
       "match": {
         "user.id": "elkbee"
       }
     }
   }
   ```

### Update API
- Request
   ```
   POST /<index>/_update/<_id>
   ```
 
- Examples
   ```json
   POST test/_update/1
   {
     "doc": {
       "name": "new_name"
     }
   }
   ```

### Update By Query API
- Request
   ```
   POST /<target>/_update_by_query
   ```
 
- Examples
   ```json
   POST my-index-000001/_update_by_query
   {
     "slice": {
       "id": 0,
       "max": 2
     },
     "script": {
       "source": "ctx._source['extra'] = 'test'"
     }
   }
   ```

### Multi get (mget) API
- Request
   ```
   GET /_mget
   
   GET /<index>/_mget
   ```
 
- Examples
   ```json
   GET /_mget
   {
     "docs": [
       {
         "_index": "my-index-000001",
         "_id": "1"
       },
       {
         "_index": "my-index-000001",
         "_id": "2"
       }
     ]
   }

   GET /my-index-000001/_mget
   {
     "docs": [
       {
         "_type": "_doc",
         "_id": "1"
       },
       {
         "_type": "_doc",
         "_id": "2"
       }
     ]
   }
   ```

### Bulk API
- Request
   ```
   POST /_bulk
   
   POST /<target>/_bulk
   ```
 
- Examples
   ```json
   POST _bulk
   { "index" : { "_index" : "test", "_id" : "1" } }
   { "field1" : "value1" }
   { "delete" : { "_index" : "test", "_id" : "2" } }
   { "create" : { "_index" : "test", "_id" : "3" } }
   { "field1" : "value3" }
   { "update" : {"_id" : "1", "_index" : "test"} }
   { "doc" : {"field2" : "value2"} }
   ```

### Reindex API
- Request
   ```
   POST /_reindex
   ```
 
- Examples
   ```json
   POST _reindex
   {
     "source": {
       "index": "my-index-000001"
     },
     "dest": {
       "index": "my-new-index-000001"
     }
   }
   ```
