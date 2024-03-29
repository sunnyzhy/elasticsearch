# Elasticsearch

## elasticsearch、kibana、elasticsearch-analysis-ik 安装须知

1. 安装路径不能有空格或中文字符
2. 各个安装包的版本必须保持一致

## Elasticsearch示例文档

[https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html "Elasticsearch示例文档")

## Elasticsearch开发文档

[https://www.elastic.co/guide/en/elasticsearch/client/index.html](https://www.elastic.co/guide/en/elasticsearch/client/index.html "Elasticsearch开发文档")

## 数据库与Elasticsearch结合使用

### 设计

- 创建的每个Elasticsearch索引都应该由符合ACID的数据存储支持。

- 数据库应该是真实的最终来源，从中填充索引。

- 如果异常情况发生（节点丢失，中断或误操作 ）导致丢失了索引，您将能够完全恢复它。

- 一般的用法是另外的数据库比如NOSQL里面有一份，然后实时同步到ES，这样一个用于键值查询，一个用于各种其他查询。 如果ES升级了之类的，比如数据结构变了，那么老版本数据可以不要，直接NOSQL再导入一份到新版本，还可以恢复。

- logstash的同步插件如logstash_input_jdbc 不支持同步删除操作，建议改为更新操作加标记flag，或者通过业务逻辑实现同步删除操作。

### 核心操作

- ES 中只存储检索字段，方便快速检索、全文检索。

- Mysql 中存储全部字段，利用 ACID 事务特性。

- 通过关联字段建立关联，比如：news_id 在 ES 和 mysql 中要有相同的值。

- 核心数据先通过 ES 快速获取 Id（如 news_id ）,再通过 Mysql 二次查询。

## 集群健康

[Cluster health API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-health.html)

Health status of the cluster, based on the state of its primary and replica shards. Statuses are:

- **green**

    All shards are assigned.

- **yellow**

    All primary shards are assigned, but one or more replica shards are unassigned. If a node in the cluster fails, some data could be unavailable until that node is repaired.

- **red**

    One or more primary shards are unassigned, so some data is unavailable. This can occur briefly during cluster startup as primary shards are assigned.

## 配置用户名和密码

### 1. 设置 Elasticsearch 的认证密码

```bash
# ./bin/elasticsearch-setup-passwords interactive
```

### 2. 重置 Elasticsearch 的用户密码

#### 2.1. 重置为随机密码

```bash
# ./bin/elasticsearch-reset-password -u <username>
```

#### 2.2. 重置为指定密码

```bash
# ./bin/elasticsearch-reset-password -u <username> -i
```

#### 2.3. 重置远程的用户密码

```bash
# ./bin/elasticsearch-reset-password --url "http://<remote_ip>:9200" -u <username> -i
```

### 3. 修改 Elasticsearch 的认证密码

#### 3.1. 方法1

```
POST /_security/user/<username>/_password
{
  "password": "123456"
}
```

#### 3.2. 方法2

```bash
# curl --insecure --anyauth -u <username>:<old_password> -X POST -H "Content-Type: application/json" http://localhost:9200/_security/user/<username>/_password -d '{"password":"123456"}'
```

### 4. 配置 Kibana 的认证密码

```bash
# vim ./config/kibana.yml

elasticsearch.username: "kibana_system"
elasticsearch.password: "password"
```

### 5. spring-boot 连接认证

添加依赖:

```xml
<properties>
    <elastic.version>8.4.1</elastic.version>
    <jakarta.version>2.0.1</jakarta.version>
    <jackson.version>2.12.3</jackson.version>
</properties>

<dependencies>
    <dependency>
        <groupId>co.elastic.clients</groupId>
        <artifactId>elasticsearch-java</artifactId>
        <version>${elastic.version}</version>
        <exclusions>
            <exclusion>
                <groupId>org.elasticsearch.client</groupId>
                <artifactId>elasticsearch-rest-client</artifactId>
            </exclusion>
            <exclusion>
                <groupId>jakarta.json</groupId>
                <artifactId>jakarta.json-api</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-client</artifactId>
        <version>${elastic.version}</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>jakarta.json</groupId>
        <artifactId>jakarta.json-api</artifactId>
        <version>${jakarta.version}</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>${jackson.version}</version>
    </dependency>
</dependencies>
```

```https(忽略证书)```:

```java
@Bean
public ElasticsearchClient elasticsearchClient() throws IOException, CertificateException, NoSuchAlgorithmException, KeyStoreException, KeyManagementException {
    //  设置账号密码
    final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
    credentialsProvider.setCredentials(AuthScope.ANY, new UsernamePasswordCredentials("elastic", "123456"));
    //  忽略证书
    SSLContext sslContext = SSLContext.getInstance("SSL");
    TrustManager[] trustAllCerts = new TrustManager[]{
            new X509TrustManager() {
                @Override
                public X509Certificate[] getAcceptedIssuers() {
                    return null;
                }

                @Override
                public void checkClientTrusted(X509Certificate[] certs, String authType) {
                }

                @Override
                public void checkServerTrusted(X509Certificate[] certs, String authType) {
                }
            }
    };
    sslContext.init(null, trustAllCerts, new SecureRandom());
    RestClient client = RestClient
            .builder(new HttpHost("127.0.0.1",9200, "https"))
            .setRequestConfigCallback(builder -> {
                builder.setConnectionRequestTimeout(5000);
                builder.setSocketTimeout(30000);
                return builder;
            })
            .setHttpClientConfigCallback(httpAsyncClientBuilder -> httpAsyncClientBuilder
                    .setDefaultCredentialsProvider(credentialsProvider)
                    .setSSLContext(sslContext)
                    .setSSLHostnameVerifier(new HostnameVerifier() {
                        @Override
                        public boolean verify(String s, SSLSession sslSession) {
                            return true;
                        }
                    }))
            .build();
    RestClientTransport restClientTransport = new RestClientTransport(client, new JacksonJsonpMapper());
    return new ElasticsearchClient(restClientTransport);
}
```

## Elasticsearch 迁移升级，数据同步

### 1. 安装最新版本的 nodejs

#### 1.1 卸载 nodejs

##### 1.1.1 使用 yum 先删除一次

```bash
# yum -y remove nodejs npm
```

##### 1.1.2 手动删除残留

- 进入 /usr/local/lib 删除所有 node 和 node_modules文件夹
- 进入 /usr/local/include 删除所有 node 和 node_modules 文件夹
- 检查 ~ 文件夹里面的"local" "lib" "include" 文件夹，然后删除里面的所有 "node" 和 "node_modules" 文件夹
   可以使用以下命令查找：
   ```bash
   # find ~/ -name node
   # find ~/ -name node_modules
   ```

##### 1.1.3 进入 /usr/local/bin 删除 node 的可执行文件

- 删除: /usr/local/bin/npm
- 删除: /usr/local/share/man/man1/node.1
- 删除: /usr/local/lib/dtrace/node.d
- 删除: rm -rf /home/[homedir]/.npm
- 删除: rm -rf /home/root/.npm

#### 1.2 最新版本的 nodejs

[安装 nodejs](https://github.com/sunnyzhy/nodejs/blob/master/nodejs.md 'nodejs')

### 2. 安装最新版本的 elasticdump

[elasticdump github](https://github.com/elasticsearch-dump/elasticsearch-dump 'elasticsearch-dump')

```bash
# cnpm install elasticdump -g
```

### 3. 数据迁移

#### 3.1 迁移单个索引

##### 3.1.1 从源 ES 里导出索引

###### 3.1.1.1 导出索引的 mapping 结构

```bash
# elasticdump --input=http://<username>:<password>@localhost:9200/index_name --output=/usr/local/elastic/mapping.json --type=mapping
```

###### 3.1.1.2 导出索引

```bash
# elasticdump --input=http://<username>:<password>@localhost:9200/index_name --output=/usr/local/elastic/data.json --type=data
```

##### 3.1.2 向目标 ES 里导入索引

###### 3.1.2.1 导入索引的 mapping 结构

```bash
# elasticdump --input=/usr/local/elastic/mapping.json --output=http://<username>:<password>@localhost:9200/index_name --type=mapping
```

###### 3.1.2.2 导入索引

```bash
# elasticdump --input=/usr/local/elastic/data.json --output=http://<username>:<password>@localhost:9200/index_name --type=data
```

#### 3.2 迁移所有索引（模板）

##### 3.2.1 从源 ES 里导出索引

###### 3.2.1.1 导出索引的 template 结构

```bash
# elasticdump --input=http://<username>:<password>@localhost:9200/template_* --output=/home/saftop/elastic/template.json --type=template --all=true
```

###### 3.2.1.2 导出索引

```bash
# elasticdump --input=http://<username>:<password>@localhost:9200/index_name_prefix_* --output=/usr/local/elastic/data.json --type=data --all=true
```

**如果报错 "no matches found"，就在通配符前加转义符 "\\"**

```bash
# elasticdump --input=http://<username>:<password>@localhost:9200/template_\* --output=/home/saftop/elastic/template.json --type=template --all=true
# elasticdump --input=http://<username>:<password>@localhost:9200/index_name_prefix_\* --output=/usr/local/elastic/data.json --type=data --all=true
```

##### 3.2.2 向目标 ES 里导入索引

###### 3.2.2.1 导入索引的 template 结构

```bash
# elasticdump --input=/home/saftop/elastic/template.json --output=http://<username>:<password>@localhost:9200 --type=template --all=true
```

###### 3.2.2.2 导入索引

```bash
# elasticdump --input=/usr/local/elastic/data.json --output=http://<username>:<password>@localhost:9200 --type=data --all=true
```

#### 3.3 迁移所有索引

##### 3.3.1 从源 ES 里导出索引

###### 3.3.1.1 导出索引的 mapping 结构

```bash
# elasticdump --input=http://<username>:<password>@localhost:9200 --output=/usr/local/elastic/mapping.json --type=mapping --all=true
```

###### 3.3.1.2 导出索引

```bash
# elasticdump --input=http://<username>:<password>@localhost:9200 --output=/usr/local/elastic/data.json --type=data --all=true
```

##### 3.3.2 向目标 ES 里导入索引

###### 3.3.2.1 导入索引的 mapping 结构

```bash
# elasticdump --input=/usr/local/elastic/mapping.json --output=http://<username>:<password>@localhost:9200 --type=mapping --all=true
```

###### 3.3.2.2 导入索引

```bash
# elasticdump --input=/usr/local/elastic/data.json --output=http://<username>:<password>@localhost:9200 --type=data --all=true
```

#### 3.4 其它

- elasticdump 访问 Elasticsearch 时需要账号认证，在 http 后面添加 **username:password@**
   ```bash
   # elasticdump --input=http://<username>:<password>@localhost:9200/index_name --output=/usr/local/elastic/data.json --type=data
   ```

- elasticdump 访问 https
   ```bash
   # NODE_TLS_REJECT_UNAUTHORIZED=0 elasticdump --input=/usr/local/elastic/data.json --output=https://<username>:<password>@localhost:9200 --type=data --all=true --limit 500
   ```

- 本地迁移
   ```bash
   # elasticdump --input=http://localhost:9200/index_name --output=/usr/local/elastic/data.json --type=data
   ```

- 服务器之间迁移
   ```bash
   # elasticdump --input=http://192.168.0.100:9200/index_name --output=http://192.168.0.101:9200/index_name --type=data
   ```

- 更多数据迁移方法请参考 [elasticdump github](https://github.com/elasticsearch-dump/elasticsearch-dump 'elasticsearch-dump')
