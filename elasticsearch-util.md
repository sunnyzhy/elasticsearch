# elasticsearch-util

## 添加依赖

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

## application.yml

```yml
server:
  port: 8080
spring:
  application:
    name: elastic-app
  elasticsearch:
    uris: https://${elasticsearch.hosts}
    username: ${elasticsearch.username}
    password: ${elasticsearch.password}

elasticsearch:
  username: elastic
  password: elastic
  hosts: 192.168.0.10:9200
  crt: http_ca.crt
```

注: 

1. ```http_ca.crt``` 复制的是 ```elasticsearch``` 服务器里的 ```config/certs/http_ca.crt```
2. ```http_ca.crt``` 证书在 ```resources``` 目录下，与 ```application.yml``` 同级

## ElasticsearchClient

### 验证证书

在 application.yml 里添加以下配置（跳过应用启动时对 elasticsearch 状态的健康检查，不然会报异常 ```General SSLEngine problem```）:

```yml
management:
  health:
    elasticsearch:
      enabled: false
```

```java
@ConfigurationProperties(prefix = "elasticsearch")
@Configuration
public class ElasticsearchBean {
    @Setter
    private String hosts;

    @Setter
    private String username;

    @Setter
    private String password;

    @Setter
    private String crt;

    @Bean
    public ElasticsearchClient elasticsearchClient() throws Exception {
        //  解析 HttpHost
        HttpHost[] hosts = getHttpHost();

        //  设置账号密码
        final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
        credentialsProvider.setCredentials(AuthScope.ANY, new UsernamePasswordCredentials(username, password));

        // 生成 SSLContext
        SSLContext sslContext = buildSSLContext();

        RestClient client = RestClient
                .builder(hosts)
                .setRequestConfigCallback(builder -> {
                    builder.setConnectionRequestTimeout(5000);
                    builder.setSocketTimeout(30000);
                    return builder;
                })
                .setHttpClientConfigCallback(httpAsyncClientBuilder -> httpAsyncClientBuilder
                        .setSSLContext(sslContext)
                        .setSSLHostnameVerifier(NoopHostnameVerifier.INSTANCE)
                        .setDefaultCredentialsProvider(credentialsProvider))
                .build();
        RestClientTransport restClientTransport = new RestClientTransport(client, new JacksonJsonpMapper());
        return new ElasticsearchClient(restClientTransport);
    }

    private HttpHost[] getHttpHost() throws Exception {
        if (!StringUtils.hasLength(hosts)) {
            throw new Exception("hosts 配置缺失");
        }
        String[] hosts = this.hosts.split(";");
        HttpHost[] httpHosts = new HttpHost[hosts.length];
        for (int i = 0; i < hosts.length; i++) {
            String[] strings = hosts[i].split(":");
            HttpHost httpHost = new HttpHost(strings[0], Integer.parseInt(strings[1]), "https");
            httpHosts[i] = httpHost;
        }
        return httpHosts;
    }

    private SSLContext buildSSLContext() throws CertificateException, IOException, KeyStoreException, NoSuchAlgorithmException, KeyManagementException {
        ClassPathResource resource = new ClassPathResource(crt);
        CertificateFactory factory = CertificateFactory.getInstance("X.509");
        Certificate certificate;
        try (InputStream is = resource.getInputStream()) {
            certificate = factory.generateCertificate(is);
        }
        KeyStore trustStore = KeyStore.getInstance("pkcs12");
        trustStore.load(null, null);
        trustStore.setCertificateEntry("ca", certificate);
        SSLContextBuilder sslContextBuilder = SSLContexts.custom()
                .loadTrustMaterial(trustStore, null);
        return sslContextBuilder.build();
    }
}
```

### 忽略证书

```java
@ConfigurationProperties(prefix = "elasticsearch")
@Configuration
public class ElasticsearchBean {
    @Setter
    private String hosts;

    @Setter
    private String username;

    @Setter
    private String password;

    @Setter
    private String crt;

    @Bean
    public ElasticsearchClient elasticsearchClient() throws Exception {
        //  解析 HttpHost
        HttpHost[] hosts = getHttpHost();

        //  设置账号密码
        final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
        credentialsProvider.setCredentials(AuthScope.ANY, new UsernamePasswordCredentials(username, password));

        // 生成 SSLContext
        SSLContext sslContext = buildSSLContext();

        RestClient client = RestClient
                .builder(hosts)
                .setRequestConfigCallback(builder -> {
                    builder.setConnectionRequestTimeout(5000);
                    builder.setSocketTimeout(30000);
                    return builder;
                })
                .setHttpClientConfigCallback(httpAsyncClientBuilder -> httpAsyncClientBuilder
                        .setSSLContext(sslContext)
                        .setSSLHostnameVerifier(new HostnameVerifier() {
                            @Override
                            public boolean verify(String s, SSLSession sslSession) {
                                return true;
                            }
                        })
                        .setDefaultCredentialsProvider(credentialsProvider))
                .build();
        RestClientTransport restClientTransport = new RestClientTransport(client, new JacksonJsonpMapper());
        return new ElasticsearchClient(restClientTransport);
    }

    private HttpHost[] getHttpHost() throws Exception {
        if (!StringUtils.hasLength(hosts)) {
            throw new Exception("hosts 配置缺失");
        }
        String[] hosts = this.hosts.split(";");
        HttpHost[] httpHosts = new HttpHost[hosts.length];
        for (int i = 0; i < hosts.length; i++) {
            String[] strings = hosts[i].split(":");
            HttpHost httpHost = new HttpHost(strings[0], Integer.parseInt(strings[1]), "https");
            httpHosts[i] = httpHost;
        }
        return httpHosts;
    }

    private SSLContext buildSSLContext() throws NoSuchAlgorithmException, KeyManagementException {
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
        return sslContext;
    }
}
```

## 普通类

```java
public class ElasticsearchCondition {
    private Integer from = 0;
    private Integer size = 10;
    private Integer maxWindow = 20000;
    private String sortField;
    private SortOrder sortOrder = SortOrder.Desc;

    public Integer getFrom() {
        return from;
    }

    public void setFrom(Integer from) {
        this.from = from;
    }

    public Integer getSize() {
        return size;
    }

    public void setSize(Integer size) {
        this.size = size;
    }

    public Integer getMaxWindow() {
        return maxWindow;
    }

    public void setMaxWindow(Integer maxWindow) {
        this.maxWindow = maxWindow;
    }

    public String getSortField() {
        return sortField;
    }

    public void setSortField(String sortField) {
        this.sortField = sortField;
    }

    public SortOrder getSortOrder() {
        return sortOrder;
    }

    public void setSortOrder(SortOrder sortOrder) {
        this.sortOrder = sortOrder;
    }
}
```

```java
public class ElasticsearchResponseData<T> {
    private long total = 0;
    private int count = 0;
    private List<T> rows = new ArrayList<>();
    private List<Hit> hits = new ArrayList<>();

    public long getTotal() {
        return total;
    }

    public void setTotal(long total) {
        this.total = total;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }

    public List<T> getRows() {
        return rows;
    }

    public void setRows(List<T> rows) {
        this.rows = rows;
    }

    public List<Hit> getHits() {
        return hits;
    }

    public void setHits(List<Hit> hits) {
        this.hits = hits;
    }
}
```

## 工具类

```java
@Component
public class ElasticsearchUtil {
    private final ElasticsearchClient elasticsearchClient;

    public ElasticsearchUtil(ElasticsearchClient elasticsearchClient) {
        this.elasticsearchClient = elasticsearchClient;
    }

    /**
     * 生成 uuid
     *
     * @return
     */
    public String createUuid() {
        return UUID.randomUUID().toString().replaceAll("-", "");
    }

    /**
     * 初始化索引
     *
     * @param indexPrefix
     * @param params
     * @return
     */
    public String initIndex(String indexPrefix, Object... params) {
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append(indexPrefix);
        if (params.length > 0) {
            for (Object param : params) {
                stringBuilder.append("_");
                stringBuilder.append(param);
            }
        }
        return stringBuilder.toString();
    }

    /**
     * 添加文档
     *
     * @param builder
     * @param index
     * @param doc
     */
    public void index(BulkRequest.Builder builder, String index, Object doc) {
        String id = createUuid();
        index(builder, index, id, doc);
    }

    /**
     * 添加文档
     *
     * @param builder
     * @param index
     * @param id
     * @param doc
     */
    public void index(BulkRequest.Builder builder, String index, String id, Object doc) {
        builder.operations(x -> x.index(y -> y.index(index).id(id).document(doc)));
    }

    /**
     * 修改文档
     *
     * @param builder
     * @param index
     * @param id
     * @param doc
     */
    public void update(BulkRequest.Builder builder, String index, String id, Object doc) {
        builder.operations(x -> x.update(y -> y.index(index).id(id).action(z -> z.doc(doc))));
    }

    /**
     * 删除文档
     *
     * @param builder
     * @param index
     * @param id
     */
    public void delete(BulkRequest.Builder builder, String index, String id) {
        builder.operations(x -> x.delete(y -> y.index(index).id(id)));
    }

    /**
     * 删除索引
     *
     * @param indices
     * @throws IOException
     */
    public void delete(List<String> indices) throws IOException {
        try {
            elasticsearchClient.indices().delete(d -> d.index(indices));
        } catch (ElasticsearchException ex) {
            if (ex.response().status() == HttpStatus.NOT_FOUND.value()) {
                return;
            }
            throw new IOException(ex);
        }
    }

    /**
     * 查询
     *
     * @param index
     * @param id
     * @param includes
     * @param clazz
     * @param <T>
     * @return
     * @throws IOException
     */
    public <T> T get(String index, String id, @Nullable List<String> includes, Class<T> clazz) throws IOException {
        if (isEmpty(index)) {
            return null;
        }
        GetRequest request = GetRequest.of(r -> {
            r.index(index).id(id);
            if (includes != null && !includes.isEmpty()) {
                r.sourceIncludes(includes);
            }
            return r;
        });
        GetResponse<T> response = null;
        try {
            response = elasticsearchClient.get(request, clazz);
        } catch (ElasticsearchException ex) {
            if (ex.response().status() == HttpStatus.NOT_FOUND.value()) {
                return null;
            }
            throw new IOException(ex);
        }
        if (response == null) {
            return null;
        }
        return response.source();
    }

    /**
     * 查询
     *
     * @param boolQuery
     * @param condition
     * @param indices
     * @param clazz
     * @param <T>
     * @return
     * @throws IOException
     */
    public <T> ElasticsearchResponseData<T> search(BoolQuery boolQuery, ElasticsearchCondition condition, List<String> indices, Class<T> clazz) throws IOException {
        if (isEmpty(indices)) {
            return new ElasticsearchResponseData<T>();
        }
        SearchRequest request = buildSearchRequest(boolQuery, condition, indices, null);
        if (request == null) {
            return new ElasticsearchResponseData<T>();
        }
        ElasticsearchResponseData<T> responseData = buildSearchResponseData(request, clazz);
        return responseData;
    }

    /**
     * 聚合查询
     *
     * @param boolQuery
     * @param aggregations
     * @param condition
     * @param indices
     * @param clazz
     * @param <T>
     * @return
     * @throws IOException
     */
    public <T> Map<String, Aggregate> search(BoolQuery boolQuery, Map<String, Aggregation> aggregations, ElasticsearchCondition condition, List<String> indices, Class<T> clazz) throws IOException {
        if (isEmpty(indices)) {
            return new HashMap<>();
        }
        SearchRequest request = buildSearchRequest(boolQuery, condition, indices, aggregations);
        if (request == null) {
            return new HashMap<>();
        }
        Map<String, Aggregate> aggregateMap = buildAggregationResponseData(request, clazz);
        return aggregateMap;
    }

    /**
     * 封装聚合查询
     *
     * @param index
     * @param field
     * @param aggregationAlias
     * @param clazz
     * @param <T>
     * @return
     * @throws IOException
     */
    public <T> List<String> search(String index, String field, String aggregationAlias, Class<T> clazz) throws IOException {
        List<String> list = new ArrayList<>();
        if (isEmpty(index)) {
            return list;
        }
        Aggregation aggregation = termsAggregation(field, null);
        SearchRequest request = new SearchRequest.Builder()
                .index(index)
                .from(0)
                .size(0)
                .aggregations(aggregationAlias, aggregation)
                .build();
        Map<String, Aggregate> aggregateMap = buildAggregationResponseData(request, clazz);
        for (Map.Entry<String, Aggregate> entry : aggregateMap.entrySet()) {
            for (StringTermsBucket bucket : ((StringTermsAggregate) entry.getValue()._get()).buckets().array()) {
                list.add(bucket.key());
            }
        }
        return list;
    }

    /**
     * 游标查询
     *
     * @param indices
     * @param size
     * @param boolQuery
     * @param clazz
     * @param responseHandler
     * @param <T>
     * @return
     * @throws Exception
     */
    public <T> boolean scroll(List<String> indices, int size, @Nullable BoolQuery boolQuery, Class<T> clazz, ResponseHandler<T> responseHandler) throws Exception {
        SearchRequest.Builder builder = new SearchRequest.Builder()
                .index(indices)
                .size(size)
                .scroll(s -> s.time("60s"));
        if (boolQuery != null) {
            builder.query(t -> t.bool(boolQuery));
        }
        SearchResponse<T> searchResponse = null;
        try {
            searchResponse = elasticsearchClient.search(builder.build(), clazz);
        } catch (ElasticsearchException ex) {
            if (ex.response().status() == HttpStatus.NOT_FOUND.value()) {
                return true;
            }
            throw new IOException(ex);
        }
        if (searchResponse == null) {
            return false;
        }
        List<Hit<T>> hits = searchResponse.hits().hits();
        if (responseHandler != null) {
            List<T> list = hits.stream().map(x -> x.source()).collect(Collectors.toList());
            responseHandler.handle(list);
        }
        String scrollId = searchResponse.scrollId();
        // 通过游标，分页取数据
        while (hits != null && !hits.isEmpty()) {
            ScrollRequest scrollRequest = new ScrollRequest.Builder()
                    .scrollId(scrollId)
                    .scroll(s -> s.time("60s"))
                    .build();
            ScrollResponse<T> scrollResponse = null;
            try {
                scrollResponse = elasticsearchClient.scroll(scrollRequest, clazz);
            } catch (ElasticsearchException ex) {
                if (ex.response().status() == HttpStatus.NOT_FOUND.value()) {
                    return true;
                }
                throw new IOException(ex);
            }
            if (scrollResponse == null) {
                return false;
            }
            scrollId = scrollResponse.scrollId();
            hits = scrollResponse.hits().hits();
            if (responseHandler != null) {
                List<T> list = hits.stream().map(x -> x.source()).collect(Collectors.toList());
                responseHandler.handle(list);
            }
        }
        // 清除游标
        ClearScrollRequest clearScrollRequest = new ClearScrollRequest.Builder().scrollId(scrollId).build();
        ClearScrollResponse clearScrollResponse = null;
        try {
            clearScrollResponse = elasticsearchClient.clearScroll(clearScrollRequest);
        } catch (ElasticsearchException ex) {
            if (ex.response().status() == HttpStatus.NOT_FOUND.value()) {
                return true;
            }
            throw new IOException(ex);
        }
        if (clearScrollResponse == null) {
            return false;
        }
        return clearScrollResponse.succeeded();
    }

    /**
     * 封装查询请求
     *
     * @param boolQuery
     * @param condition
     * @param indices
     * @param aggregations
     * @return
     */
    private SearchRequest buildSearchRequest(BoolQuery boolQuery, ElasticsearchCondition condition, List<String> indices, @Nullable Map<String, Aggregation> aggregations) {
        if (isEmpty(indices)) {
            return null;
        }
        SearchRequest.Builder builder = new SearchRequest.Builder()
                .index(indices)
                .trackTotalHits(t -> t.count(condition.getMaxWindow()))
                .query(t -> t.bool(boolQuery))
                .timeout("30s")
                .from(condition.getFrom())
                .size(condition.getSize());
        if (!isEmpty(condition.getSortField())) {
            builder.sort(s -> s.field(f -> f.field(condition.getSortField()).order(condition.getSortOrder())));
        }
        if (!isEmpty(aggregations)) {
            builder.aggregations(aggregations);
        }
        return builder.build();
    }

    /**
     * 封装响应
     *
     * @param request
     * @param clazz
     * @param <T>
     * @return
     * @throws IOException
     */
    private <T> ElasticsearchResponseData<T> buildSearchResponseData(SearchRequest request, Class<T> clazz) throws IOException {
        ElasticsearchResponseData<T> responseData = new ElasticsearchResponseData<>();
        SearchResponse<T> response = null;
        try {
            response = elasticsearchClient.search(request, clazz);
        } catch (ElasticsearchException ex) {
            if (ex.response().status() == HttpStatus.NOT_FOUND.value()) {
                return responseData;
            }
            throw new IOException(ex);
        }
        if (response == null) {
            return responseData;
        }
        TotalHits total = response.hits().total();
        responseData.setTotal(total.value());
        List<Hit<T>> hits = response.hits().hits();
        for (Hit<T> hit : hits) {
            responseData.getRows().add(hit.source());
            responseData.getHits().add(hit);
        }
        responseData.setCount(hits.size());
        return responseData;
    }

    /**
     * 封装聚合响应
     *
     * @param request
     * @param clazz
     * @param <T>
     * @return
     * @throws IOException
     */
    private <T> Map<String, Aggregate> buildAggregationResponseData(SearchRequest request, Class<T> clazz) throws IOException {
        Map<String, Aggregate> aggregateMap = new HashMap<>();
        SearchResponse<T> response = null;
        try {
            response = elasticsearchClient.search(request, clazz);
        } catch (ElasticsearchException ex) {
            if (ex.response().status() == HttpStatus.NOT_FOUND.value()) {
                return aggregateMap;
            }
            throw new IOException(ex);
        }
        if (response == null) {
            return aggregateMap;
        }
        aggregateMap.putAll(response.aggregations());
        return aggregateMap;
    }

    /**
     * 批量添加/修改/删除
     *
     * @param bulkRequest
     * @throws IOException
     */
    public void bulk(BulkRequest bulkRequest) throws IOException {
        if (isEmpty(bulkRequest.operations())) {
            return;
        }
        BulkResponse response = null;
        try {
            response = elasticsearchClient.bulk(bulkRequest);
        } catch (ElasticsearchException ex) {
            if (ex.response().status() == HttpStatus.NOT_FOUND.value()) {
                return;
            }
            throw new IOException(ex);
        }
        if (response.errors()) {
            StringBuilder sb = new StringBuilder();
            for (BulkResponseItem item : response.items()) {
                if (item.error() != null) {
                    sb.append(item.error().reason());
                }
            }
            String msg = sb.toString();
            if (msg.length() > 0) {
                throw new IOException(sb.toString());
            }
        }
    }

    /**
     * count查询
     *
     * @param boolQuery
     * @param indices
     * @return
     * @throws IOException
     */
    public long count(BoolQuery boolQuery, List<String> indices) throws IOException {
        if (isEmpty(indices)) {
            return 0L;
        }
        CountRequest request = CountRequest.of(b -> b
                .index(indices)
                .query(t -> t
                        .bool(boolQuery)));
        CountResponse response = null;
        try {
            response = elasticsearchClient.count(request);
        } catch (ElasticsearchException ex) {
            if (ex.response().status() == HttpStatus.NOT_FOUND.value()) {
                return 0L;
            }
            throw new IOException(ex);
        }
        return response.count();
    }

    /**
     * 判断索引是否存在
     *
     * @param indices
     * @return
     * @throws IOException
     */
    public Map<String, Boolean> indexExists(List<String> indices) throws IOException {
        Map<String, Boolean> map = new HashMap<>();
        if (isEmpty(indices)) {
            return map;
        }
        try {
            boolean b = elasticsearchClient.indices().exists(i -> i.index(indices)).value();
            if (b) {
                indices.forEach(x -> map.put(x, true));
            } else {
                for (String index : indices) {
                    b = elasticsearchClient.indices().exists(i -> i.index(index)).value();
                    map.put(index, b);
                }
            }
            return map;
        } catch (ElasticsearchException ex) {
            if (ex.response().status() == HttpStatus.NOT_FOUND.value()) {
                return map;
            }
            throw new IOException(ex);
        }
    }

    /**
     * termQuery
     *
     * @param field
     * @param value
     * @param clazz
     * @param <T>
     * @return
     */
    public <T> Query termQuery(String field, T value, Class<T> clazz) {
        FieldValue fieldValue = buildFieldValue(value, clazz);
        Query query = QueryBuilders.term(x -> x.field(field).value(fieldValue));
        return query;
    }

    /**
     * termsQuery
     *
     * @param field
     * @param values
     * @param clazz
     * @param <T>
     * @return
     */
    public <T> Query termsQuery(String field, List<T> values, Class<T> clazz) {
        List<FieldValue> fieldValues = new ArrayList<>();
        for (T value : values) {
            FieldValue fieldValue = buildFieldValue(value, clazz);
            fieldValues.add(fieldValue);
        }
        Query query = QueryBuilders.terms(x -> x.field(field).terms(y -> y.value(fieldValues)));
        return query;
    }

    /**
     * 封装查询字段
     *
     * @param value
     * @param clazz
     * @param <T>
     * @return
     */
    private <T> FieldValue buildFieldValue(T value, Class<T> clazz) {
        FieldValue fieldValue = null;
        if (clazz.getName().equals(Long.class.getName()) || clazz.getName().equals(Integer.class.getName())) {
            fieldValue = FieldValue.of(Long.parseLong(value.toString()));
        } else if (clazz.getName().equals(Boolean.class.getName())) {
            fieldValue = FieldValue.of(Boolean.parseBoolean(value.toString()));
        } else if (clazz.getName().equals(Double.class.getName())) {
            fieldValue = FieldValue.of(Double.parseDouble(value.toString()));
        } else {
            fieldValue = FieldValue.of(value.toString());
        }
        return fieldValue;
    }

    /**
     * rangeQuery(>=x && <=7)
     *
     * @param field
     * @param min
     * @param max
     * @param <T>
     * @return
     */
    public <T> Query rangeQuery(String field, T min, T max) {
        Query query = QueryBuilders.range(x -> x.field(field).gte(JsonData.of(min)).lte(JsonData.of(max)));
        return query;
    }

    /**
     * rangeQuery(>x)
     *
     * @param field
     * @param min
     * @param <T>
     * @return
     */
    public <T> Query rangeGtQuery(String field, T min) {
        Query query = QueryBuilders.range(x -> x.field(field).gt(JsonData.of(min)));
        return query;
    }

    /**
     * rangeQuery(>=x)
     *
     * @param field
     * @param min
     * @param <T>
     * @return
     */
    public <T> Query rangeGteQuery(String field, T min) {
        Query query = QueryBuilders.range(x -> x.field(field).gte(JsonData.of(min)));
        return query;
    }

    /**
     * rangeQuery(<y)
     *
     * @param field
     * @param max
     * @param <T>
     * @return
     */
    public <T> Query rangeLtQuery(String field, T max) {
        Query query = QueryBuilders.range(x -> x.field(field).lt(JsonData.of(max)));
        return query;
    }

    /**
     * rangeQuery(<=y)
     *
     * @param field
     * @param max
     * @param <T>
     * @return
     */
    public <T> Query rangeLteQuery(String field, T max) {
        Query query = QueryBuilders.range(x -> x.field(field).lte(JsonData.of(max)));
        return query;
    }

    /**
     * matchQuery
     *
     * @param field
     * @param value
     * @param <T>
     * @return
     */
    public <T> Query matchQuery(String field, String value) {
        Query query = QueryBuilders.match(x -> x.field(field).query(value));
        return query;
    }

    /**
     * multiMatchQuery
     *
     * @param fields
     * @param value
     * @param <T>
     * @return
     */
    public <T> Query multiMatchQuery(List<String> fields, String value) {
        Query query = QueryBuilders.multiMatch(x -> x.fields(fields).query(value).minimumShouldMatch("100%"));
        return query;
    }

    /**
     * termsAggregation
     *
     * @param field
     * @param size
     * @return
     */
    public Aggregation termsAggregation(String field, @Nullable Integer size) {
        Aggregation aggregation = Aggregation.of(x ->
                x.terms(y -> {
                    y.field(field);
                    if (size != null) {
                        y.size(size);
                    }
                    return y;
                }));
        return aggregation;
    }

    /**
     * termsAggregation
     *
     * @param field
     * @param size
     * @param subAggregation
     * @return
     */
    public Aggregation termsAggregation(String field, @Nullable Integer size, @Nullable Map<String, Aggregation> subAggregation) {
        Aggregation aggregation = Aggregation.of(x -> {
            Aggregation.Builder.ContainerBuilder builder = x.terms(y -> {
                y.field(field);
                if (size != null) {
                    y.size(size);
                }
                return y;
            });
            if (!isEmpty(subAggregation)) {
                builder.aggregations(subAggregation);
            }
            return builder;
        });
        return aggregation;
    }

    /**
     * topHitsAggregation
     *
     * @param field
     * @param size
     * @return
     */
    public Aggregation topHitsAggregation(String field, @Nullable Integer size) {
        Aggregation aggregation = Aggregation.of(x -> x.topHits(y -> {
            y.field(field).source(s -> s.fetch(false));
            if (size != null) {
                y.size(size);
            }
            return y;
        }));
        return aggregation;
    }

    /**
     * cardinalityAggregation
     *
     * @param field
     * @return
     */
    public Aggregation cardinalityAggregation(String field) {
        Aggregation aggregation = Aggregation.of(x -> x.cardinality(y -> y.field(field)));
        return aggregation;
    }

    /**
     * dateHistogramAggregation
     *
     * @param field
     * @param min
     * @param max
     * @return
     */
    public Aggregation dateHistogramAggregation(String field, Double min, Double max) {
        Aggregation aggregation = Aggregation.of(x ->
                x.dateHistogram(y -> y.field(field)
                        .calendarInterval(CalendarInterval.Day)
                        .timeZone(ZoneId.systemDefault().getId())
                        .format("yyyy-MM-dd HH:mm:ss")
                        .minDocCount(1)
                        .extendedBounds(z -> z.min(FieldDateMath.of(f -> f.value(min)))
                                .max(FieldDateMath.of(f -> f.value(max))))));
        return aggregation;
    }

    /**
     * 判断字符串是否为空
     *
     * @param cs
     * @return
     */
    private boolean isEmpty(String cs) {
        return cs == null || cs.length() == 0;
    }

    /**
     * 判断 list 是否为空
     *
     * @param css
     * @return
     */
    private boolean isEmpty(List<?> css) {
        return css == null || css.isEmpty();
    }

    /**
     * 判断 map 是否为空
     *
     * @param map
     * @return
     */
    private boolean isEmpty(Map<?, ?> map) {
        return map == null || map.isEmpty();
    }
}
```

## test

```java
@Autowired
private ElasticsearchUtil elasticsearchUtil;

@Test
void search() throws IOException {
    // boolQuery
    BoolQuery.Builder queryBuilder = new BoolQuery.Builder();
    queryBuilder.filter(elasticsearchUtil.termsQuery("name", Arrays.asList("tom"), String.class));

    // condition
    ElasticsearchCondition condition = new ElasticsearchCondition();
    condition.setFrom(0);
    condition.setSize(20);
    condition.setSortField("age");

    // search
    ElasticsearchResponseData<Student> responseData = elasticsearchUtil.search(queryBuilder.build(), condition, Arrays.asList("student"), Student.class);
    System.out.println(responseData);
}

@Test
void aggregate() throws IOException {
    // boolQuery
    BoolQuery.Builder queryBuilder = new BoolQuery.Builder();
    queryBuilder.filter(elasticsearchUtil.termsQuery("name", Arrays.asList("tom"), String.class));

    // aggregation
    Map<String, Aggregation> aggregationMap = new HashMap<>();
    aggregationMap.put("age", elasticsearchUtil.termsAggregation("age", null));

    // condition
    ElasticsearchCondition condition = new ElasticsearchCondition();
    condition.setFrom(0);
    condition.setSize(0);

    // search
    Map<String, Aggregate> response = elasticsearchUtil.search(queryBuilder.build(), aggregationMap, condition, Arrays.asList("student"), Student.class);

    // response
    for (Map.Entry<String, Aggregate> entry : response.entrySet()) {
        for (LongTermsBucket bucket : ((LongTermsAggregate) entry.getValue()._get()).buckets().array()) {
            long id = bucket.key();
            long count = bucket.docCount();
            System.out.println(id + ":" + count);
        }
    }
}
```
