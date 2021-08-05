# geo_point

**index 的 mapping 必须显式声明字段类型为 geo_point**

## 创建 mapping

```json
PUT /<index>
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}
```

## 添加 doc

```json
PUT /<index>/_doc/1
{
  "text": "Geopoint as an object",
  "location": { 
    "lat": 41.12,
    "lon": -71.34
  }
}
```

## ES 的地图检索方式

### geo_bounding_box

geo_bounding_box: 以左上角和右下角这两个点，确定一个矩形，获取矩形内的全部地理坐标。

查询的结构体:

```json
GET /<index>/_search
{
  "query": {
    "bool": {
      "filter": {
        "geo_bounding_box": {
          "FIELD": {
            "top_left": {
              "lat": 40.73,
              "lon": -74.1
            },
            "bottom_right": {
              "lat": 40.01,
              "lon": -71.12
            }
          }
        }
      }
    }
  }
}
```

参数说明:
- FIELD: 自定义的地理位置字段
- top_left: 左上点的坐标
- bottom_right: 右下点的坐标

### geo_distance

geo_distance: 以一个点和一个半径，确定一个圆形，获取圆形内的全部地理坐标。

查询的结构体:

```json
GET /<index>/_search
{
  "query": {
    "bool": {
      "filter": {
        "geo_distance": {
          "distance": "200km",
	  "distance_type": "arc",
          "FIELD": {
            "lat": 40,
            "lon": -70
          }
        }
      }
    }
  }
}
```

参数说明:
- FIELD: 自定义的地理位置字段
- distance: 半径，单位默认是 m
- distance_type: 图形的类型，默认是圆形 arc

### geo_polygon

geo_polygon: 以多个点，确定一个多边形，获取多边形内的全部地理坐标。

查询的结构体:

```json
GET /<index>/_search
{
  "query": {
    "bool": {
      "filter": {
        "geo_polygon": {
          "FIELD": {
            "points": [
              { "lat": 40, "lon": -70 },
              { "lat": 30, "lon": -80 },
              { "lat": 20, "lon": -90 }
            ]
          }
        }
      }
    }
  }
}
```

参数说明:
- FIELD: 自定义的地理位置字段
- points: 数组，存储多点的坐标，数组的元素个数必须 >= 3
