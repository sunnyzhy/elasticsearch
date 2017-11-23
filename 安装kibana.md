# 官网
https://www.elastic.co/downloads/kibana

# 安装
```
# cd /usr/local

# tar -zxvf kibana-6.0.0-linux-x86_64.tar.gz

# mv kibana-6.0.0-linux-x86_64 kibana
```

# 启动
```
# cd kibana/bin

# ./kibana
  log   [06:50:15.049] [info][status][plugin:kibana@6.0.0] Status changed from uninitialized to green - Ready
  log   [06:50:15.147] [info][status][plugin:elasticsearch@6.0.0] Status changed from uninitialized to yellow - Waiting for Elasticsearch
  log   [06:50:15.197] [info][status][plugin:console@6.0.0] Status changed from uninitialized to green - Ready
  log   [06:50:15.285] [info][status][plugin:metrics@6.0.0] Status changed from uninitialized to green - Ready
  log   [06:50:16.298] [info][status][plugin:timelion@6.0.0] Status changed from uninitialized to green - Ready
  log   [06:50:16.310] [info][listening] Server running at http://localhost:5601
  log   [06:50:16.312] [info][status][ui settings] Status changed from uninitialized to yellow - Elasticsearch plugin is yellow
  log   [06:50:21.920] [info][status][plugin:elasticsearch@6.0.0] Status changed from yellow to yellow - No existing Kibana index found
  log   [06:50:31.007] [info][status][plugin:elasticsearch@6.0.0] Status changed from yellow to green - Kibana index ready
  log   [06:50:31.008] [info][status][ui settings] Status changed from yellow to green - Ready
```

# 浏览器访问
http://127.0.0.1:5601

# 设置外网访问
```
# vim /usr/local/kibana/config/kibana.yml
server.host: 0.0.0.0

# ./kibana
```

# 外网访问
http://ip:5601

# 重启elasticsearch，关闭 -> 开启
```
# fuser -n tcp 5601
5601/tcp:            52371

# kill -9 52371
[1]+  Killed                  ./kibana

# ./kibana
```
