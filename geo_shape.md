# geo_shape

**index 的 mapping 必须显式声明字段类型为 geo_shape**

## 创建 mapping

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

## 添加 doc

- type: 必须字段，shape 类型: Point、LineString、Polygon、MultiPoint、MultiLineString、MultiPolygon、GeometryCollection、Envelope、Circle
- coordinates: 必须字段，coordinate 的顺序是 longitude, latitude (X, Y)

### Point

单个地理坐标。

```json
POST /example/_doc
{
  "location" : {
    "type" : "point",
    "coordinates" : [-77.03653, 38.897676]
  }
}
```

### LineString

1. 只指定两个点，表示一条直线
2. 指定两个以上的点，表示一条任意曲线

```json
POST /example/_doc
{
  "location" : {
    "type" : "linestring",
    "coordinates" : [[-77.03653, 38.897676], [-77.009051, 38.889939]]
  }
}
```

### Polygon

1. 多边形必须是闭合的，即第一个点和最后一个点必须相同。
2. 点的个数必须 >= 4

```json
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

### MultiPoint

```json
POST /example/_doc
{
  "location" : {
    "type" : "multipoint",
    "coordinates" : [
      [102.0, 2.0], [103.0, 2.0]
    ]
  }
}
```

### MultiLineString

```json
POST /example/_doc
{
  "location" : {
    "type" : "multilinestring",
    "coordinates" : [
      [ [102.0, 2.0], [103.0, 2.0], [103.0, 3.0], [102.0, 3.0] ],
      [ [100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0] ],
      [ [100.2, 0.2], [100.8, 0.2], [100.8, 0.8], [100.2, 0.8] ]
    ]
  }
}
```

### MultiPolygon

```json
POST /example/_doc
{
  "location" : {
    "type" : "multipolygon",
    "coordinates" : [
      [ [[102.0, 2.0], [103.0, 2.0], [103.0, 3.0], [102.0, 3.0], [102.0, 2.0]] ],
      [ [[100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0]],
        [[100.2, 0.2], [100.8, 0.2], [100.8, 0.8], [100.2, 0.8], [100.2, 0.2]] ]
    ]
  }
}
```

### GeometryCollection

点、线、面的集合。

```json
POST /example/_doc
{
  "location" : {
    "type": "geometrycollection",
    "geometries": [
      {
        "type": "point",
        "coordinates": [100.0, 0.0]
      },
      {
        "type": "linestring",
        "coordinates": [ [101.0, 0.0], [102.0, 1.0] ]
      }
    ]
  }
}
```

### Envelope

通过指定左上角和右下角，从而确定的边界矩形。coordinates的数组格式 [[minLon, maxLat], [maxLon, minLat]]

```json
POST /example/_doc
{
  "location" : {
    "type" : "envelope",
    "coordinates" : [ [100.0, 1.0], [101.0, 0.0] ]
  }
}
```

### Circle

由一个圆点和一个半径组成的圆。

```json
PUT /circle-example
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_shape",
        "strategy": "recursive"
      }
    }
  }
}

POST /circle-example/_doc
{
  "location" : {
    "type" : "circle",
    "coordinates" : [101.0, 1.0],
    "radius" : "100m"
  }
}
```

## ES 的地图检索方式

geo_shape 支持以下类型查询:
- 查询 geo_shape 类型
- 查询 geo_point 类型

### 查询 geo_shape 类型

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

POST /example/_doc
{
  "name": "Wind & Wetter, Berlin, Germany",
  "location": {
    "type": "point",
    "coordinates": [ 13.400544, 52.530286 ]
  }
}

GET /example/_search
{
  "query": {
    "bool": {
      "filter": {
        "geo_shape": {
          "location": {
            "shape": {
              "type": "envelope",
              "coordinates": [ [ 13.0, 53.0 ], [ 14.0, 52.0 ] ]
            },
            "relation": "within"
          }
        }
      }
    }
  }
}
```

### 查询 geo_point 类型

```json
PUT /example_points
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}

PUT /example_points/_doc/1
{
  "name": "Wind & Wetter, Berlin, Germany",
  "location": [13.400544, 52.530286]
}

GET /example_points/_search
{
  "query": {
    "bool": {
      "filter": {
        "geo_shape": {
          "location": {
            "shape": {
              "type": "envelope",
              "coordinates": [ [ 13.0, 53.0 ], [ 14.0, 52.0 ] ]
            },
            "relation": "intersects"
          }
        }
      }
    }
  }
}
```

relation 参数说明:

- INTERSECTS: 返回 geo_shape 或 geo_point 字段与查询几何相交的文档，**默认值**
- DISJOINT: 返回 geo_shape 或 geo_point 字段与查询几何没有相交的文档
- WITHIN: 返回 geo_shape 或 geo_point 字段在查询几何范围内的文档，不支持线
- CONTAINS: 返回 geo_shape 或 geo_point 字段包含查询几何的文档

### Pre-Indexed Shape 查询

1. 定义索引 A，即预索引图形
2. 在索引 B 的查询中，引用索引 A (预索引图形)

```json
PUT /shapes
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_shape"
      }
    }
  }
}

PUT /shapes/_doc/deu
{
  "location": {
    "type": "envelope",
    "coordinates" : [[13.0, 53.0], [14.0, 52.0]]
  }
}

GET /example/_search
{
  "query": {
    "bool": {
      "filter": {
        "geo_shape": {
          "location": {
            "indexed_shape": {
              "index": "shapes",
              "id": "deu",
              "path": "location"
            }
          }
        }
      }
    }
  }
}
```

查询的参数说明:

- id: 预索引图形的文档id
- index: 预索引图形的索引名称，默认为 shapes
- path: 预索引图形的地理位置字段，默认为shape
- routing: 图形的文档的路由

### 扩展

以下两种写法是等价的:

```json
PUT /test/_doc/1
{
  "location": [
    {
      "coordinates": [46.25,20.14],
      "type": "point"
    },
    {
      "coordinates": [47.49,19.04],
      "type": "point"
    }
  ]
}
```

```json
PUT /test/_doc/1
{
  "location":
    {
      "coordinates": [[46.25,20.14],[47.49,19.04]],
      "type": "multipoint"
    }
}
```
