# geo java 示例

## 1 创建索引

```json
PUT /example
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_shape"
      }
    }
  }
}
```

## 2 添加文档

```json
POST /example/_doc
{
  "location" : {
    "type" : "point",
    "coordinates" : [-77.03653, 38.897676]
  }
}

POST /example/_doc
{
  "location" : {
    "type" : "polygon",
    "coordinates" : [
      [ [100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0] ]
    ]
  }
}
```

## 3 java 示例代码

```java
@Autowired
private RestHighLevelClient restHighLevelClient;
private String index = "example";
private String fieldName = "location";

@Test
public void geoPoint() throws IOException {
    Point point = new Point(-77.03653, 38.897676);
    GeoShapeQueryBuilder geoShapeQueryBuilder = new GeoShapeQueryBuilder(fieldName, point);
    geoShapeQueryBuilder.relation(ShapeRelation.INTERSECTS);
    geoSearch(index, 0, 20, geoShapeQueryBuilder);
}

@Test
public void geoPolygon() throws IOException {
    double[] x = {99.0, 100.5, 100.5, 99.0, 99.0};
    double[] y = {0.0, 0.0, 0.05, 0.05, 0.0};
    LinearRing linearRing = new LinearRing(x, y);
    Polygon polygon = new Polygon(linearRing);
    GeoShapeQueryBuilder geoShapeQueryBuilder = new GeoShapeQueryBuilder(fieldName, polygon);
    geoShapeQueryBuilder.relation(ShapeRelation.INTERSECTS);
    geoSearch(index, 0, 20, geoShapeQueryBuilder);
}

@Test
public void geoEnvelope() throws IOException {
    // Rectangle构造方法: Rectangle(double minX, double maxX, double maxY, double minY)
    Rectangle rectangle = new Rectangle(-77.03654, -77.03652, 38.897677, 38.897675);
    GeoShapeQueryBuilder geoShapeQueryBuilder = new GeoShapeQueryBuilder(fieldName, rectangle);
    geoShapeQueryBuilder.relation(ShapeRelation.WITHIN);
    geoSearch(index, 0, 20, geoShapeQueryBuilder);
}

@Test
public void geoCircle() throws IOException {
    Circle circle = new Circle(100.0, 1.0, 100);
    GeoShapeQueryBuilder geoShapeQueryBuilder = new GeoShapeQueryBuilder(fieldName, circle);
    geoShapeQueryBuilder.relation(ShapeRelation.INTERSECTS);
    geoSearch(index, 0, 20, geoShapeQueryBuilder);
}

private void geoSearch(String index, int from, int size, QueryBuilder queryBuilder) throws IOException {
    // 初始化SearchSourceBuilder
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    sourceBuilder.timeout(new TimeValue(30, TimeUnit.SECONDS));
    sourceBuilder.from(from);
    sourceBuilder.size(size);
    sourceBuilder.query(queryBuilder);
    // 初始化SearchRequest
    SearchRequest searchRequest = new SearchRequest(index);
    searchRequest.source(sourceBuilder);
    // 查询
    SearchResponse response = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
    for (SearchHit hit : response.getHits().getHits()) {
        System.out.println(hit.getSourceAsMap());
    }
}
```
