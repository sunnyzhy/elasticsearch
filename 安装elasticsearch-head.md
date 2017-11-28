# 官网
http://mobz.github.io/elasticsearch-head/

# 安装
```
# cd /usr/local

# unzip elasticsearch-head-master.zip

# mv elasticsearch-head-master elasticsearch-head

# cp -r elasticsearch-head /var/www/html/es
```

# 设置跨域访问
```
# su zhy

$ cd /usr/local/elasticsearch

$ vim config/elasticsearch.yml
http.cors.enabled: true
http.cors.allow-origin: /.*/

$ ./bin/elasticsearch -d
```

# 浏览器访问
http://localhost/es/
