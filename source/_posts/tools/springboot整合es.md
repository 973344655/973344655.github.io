---
title: springboot 整合 es 的RestHighLevelClient
date: 2019-07-16 18:55:31
tags: [tools]
---

### 一.整合

#### 1.maven

尽量和自己的Es版本一致，或者稍微大于es版本，防止功能不兼容。
```
<dependency>
   <groupId>org.elasticsearch.client</groupId>
   <artifactId>elasticsearch-rest-high-level-client</artifactId>
   <version>6.6.2</version>
</dependency>

<dependency>
   <groupId>org.elasticsearch</groupId>
   <artifactId>elasticsearch</artifactId>
   <version>6.6.2</version>
</dependency>
```
#### 2.RestHighLevelClient 配置类

RestHighLevelClient也需要一个RestClient.
```
/**
 * RestHighLevelClient配置类
 */
@Component
public class EsRestHighLevelClient {

  private static String[] ips;
  private static int[] ports;
  @Value("${risk.es.ips}")
  public void setIps(String[] ip){
      ips = ip;
  }
  @Value("${risk.es.ports}")
  public void setPorts(int[] port){
      ports = port;
  }


  public RestHighLevelClient getClient(){
      HttpHost[] httpHosts = new HttpHost[ips.length];
      for(int i=0; i<ips.length; i++){
          httpHosts[i] = new HttpHost(ips[i], ports[i], "http");
      }
      RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(httpHosts));
      return client;
  }
}

```

#### 3.基本CURD

<details>
<summary> Curd </summary>

```
@Component
public class EsCurdOptions {

    @Autowired
    EsRestHighLevelClient esRestHighLevelClient;

    private static final String Index = "accounts";

    private static final String Type = "company";

    //用来做id的key
    private static final String Key = "qiyemingcheng";

    /**
     * 增
     * 以key属性字段作为id
     * @param map
     */
    public void add(Map <String,Object> map){

        RestHighLevelClient client = esRestHighLevelClient.getClient();
        IndexRequest indexRequest = new IndexRequest(Index,Type, map.get("key").toString());
        indexRequest.source(map);

        try {
            IndexResponse response =  client.index(indexRequest, RequestOptions.DEFAULT);
            System.out.println(response.toString());
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    /**
     * 批量添加
     * 添加在 Index --> Type 下
     * @param list
     */
    public void bulkAdd(List<Map<String,Object>> list){
        RestHighLevelClient client = esRestHighLevelClient.getClient();

        BulkRequest bulkRequest = new BulkRequest(Index, Type);
        for(Map<String, Object> map: list){
            IndexRequest indexRequest = new IndexRequest(Index,Type, map.get(Key).toString());
            bulkRequest.add(indexRequest.source(map));
        }
        try {
            client.bulkAsync(bulkRequest, RequestOptions.DEFAULT, new ActionListener<BulkResponse>() {
                @Override
                public void onResponse(BulkResponse bulkItemResponses) {
                    for(BulkItemResponse response: bulkItemResponses){
                        System.out.println(response.status());
                    }
                }

                @Override
                public void onFailure(Exception e) {
                    System.out.println("批量失败");
                }
            });
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    /**
     * 查
     * 在企业名称属性中查找
     * 模糊
     * @param key
     */
    public List<Map<String,Object>> search(String key){
        List<Map<String,Object>> list = new ArrayList<>();
        RestHighLevelClient client = esRestHighLevelClient.getClient();
        //index
        SearchRequest request = new SearchRequest(Index);

        //type
        request.types(Type);
        RequestOptions options = RequestOptions.DEFAULT;
        //source
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        sourceBuilder.from(0);
        sourceBuilder.size(10);
        MatchQueryBuilder matchQueryBuilder = new MatchQueryBuilder("qiyemingcheng",key);
        matchQueryBuilder.fuzziness(Fuzziness.AUTO);
        sourceBuilder.query(matchQueryBuilder);
        request.source(sourceBuilder);
        try{
            SearchResponse response = client.search(request,options);
            System.out.println(response.toString());
            SearchHits hits = response.getHits();
            SearchHit[] hits1 = hits.getHits();
            for (SearchHit hit : hits1) {
                Map<String, Object> map = hit.getSourceAsMap();
                list.add(map);
            }
        }catch (Exception e){
            e.printStackTrace();
        }
        return list;
    }


    /**
     * 查询type下所有记录
     * 有个问题  默认有分页信息  返回的数据为分页size
     * 如果 size设置过大 数据太大
     * @return
     */
    public List searchType(){
        List<String> list = new ArrayList<>();
        RestHighLevelClient client = esRestHighLevelClient.getClient();

        SearchRequest searchRequest = new SearchRequest(Index);
        searchRequest.types(Type);
        SearchSourceBuilder sourceBuilder  = new SearchSourceBuilder();
        sourceBuilder.from(0);
        //sourceBuilder.size(1000);
        QueryBuilder queryBuilder = new MatchAllQueryBuilder();
        sourceBuilder.query(queryBuilder);
        searchRequest.source(sourceBuilder);
        try {
            SearchResponse response = client.search(searchRequest, RequestOptions.DEFAULT);
            SearchHits hits = response.getHits();
            SearchHit[] hits1 = hits.getHits();
            for (SearchHit hit : hits1) {

                list.add(hit.getId());
            }
        }catch (Exception e){
            e.printStackTrace();
        }
        return list;
    }
    /**
     * 获取指定id下的数据
     * 精确
     * @param key
     */
    public Map get(String key){
        Map<String,Object> map = new HashMap<>();
        RestHighLevelClient client = esRestHighLevelClient.getClient();

        GetRequest getRequest = new GetRequest(Index, Type, key);
        try {
            GetResponse response = client.get(getRequest, RequestOptions.DEFAULT);
            map = response.getSourceAsMap();
            System.out.println(response.toString());
        }catch (Exception e){
            e.printStackTrace();
        }
        return map;
    }

    /**
     * 删
     * 删除指定id的数据
     * @param key
     */
    public void  delete(String key){
        RestHighLevelClient client = esRestHighLevelClient.getClient();
        DeleteRequest deleteRequest = new DeleteRequest(Index, Type, key);
        try{
            DeleteResponse response = client.delete(deleteRequest,RequestOptions.DEFAULT);
            System.out.println(response.toString());
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    /**
     * 删除type下所有文档
     * 先查询 再删除
     */
    public void deleteType(){
        List<String> list = searchType();
        RestHighLevelClient client = esRestHighLevelClient.getClient();

        try{
            for(String id: list){
                DeleteRequest deleteRequest = new DeleteRequest(Index,Type, id);
                client.delete(deleteRequest,RequestOptions.DEFAULT);
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    /**
     * 更新指定id下的数据
     * key为id的值
     * @param key
     * @param map
     */
    public void update(String key, Map<String,Object> map){

        RestHighLevelClient client = esRestHighLevelClient.getClient();

        UpdateRequest updateRequest = new UpdateRequest(Index,Type,key);
        IndexRequest indexRequest = new IndexRequest().source(map);
        updateRequest.doc(indexRequest);

        try{
            UpdateResponse response = client.update(updateRequest, RequestOptions.DEFAULT);
            System.out.println(response.toString());
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}

```
</details>

### 二.Es一些基本知识

#### 1.Es中文档数据例子
index+type+id可以确定一个唯一文档
```
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
    }
}
```

>_index -- 索引名称<br>
>_type -- 类型名称<br>
>_id -- id(这个id可以自己指定也可以自动生成)<br>
>_version -- 版本号,每次改动会+1<br>
>found -- true表示在document存在<br>
>_source -- document的全部内容<br>
