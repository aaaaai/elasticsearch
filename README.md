# elasticsearch

download elasticsearch 6.4.2 and kibana 6.4.2,version must be same.
https://www.elastic.co/downloads/past-releases
unzip elasticsearch.zip and kibana.zip

use chinese analyzer:ansj
download elasticsearch-analysis-ansj
https://github.com/NLPchina/elasticsearch-analysis-ansj

user chrome brower download https://github.com/NLPchina/elasticsearch-analysis-ansj/releases/download/v6.4.2/elasticsearch-analysis-ansj-6.4.2.0-release.zip
copy zip to web project:http://127.0.0.1:8080/app/elasticsearch-analysis-ansj-6.4.2.0-release.zip

in elasticsearch folder execute command:
./bin/elasticsearch-plugin install http://127.0.0.1:8080/app/elasticsearch-analysis-ansj-6.4.2.0-release.zip
version must be same.

update ..\elasticsearch-6.4.2\config\elasticsearch.yml

# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
# network.host: 192.168.0.1
network.host: 192.168.3.163
#
# Set a custom port for HTTP:
#
# http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
# discovery.zen.ping.unicast.hosts: ["host1", "host2"]
discovery.zen.ping.unicast.hosts: ["192.168.3.163","127.0.0.1","192.168.3.119"]



update ..\kibana-6.4.2\config\kibana.yml

elasticsearch.url: "http://192.168.3.163:9200"

java maven project pom.xml

		<!-- https://mvnrepository.com/artifact/org.ansj/ansj_seg -->
		<dependency>
			<groupId>org.ansj</groupId>
			<artifactId>ansj_seg</artifactId>
			<version>5.1.6</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.ansj/ansj_lucene7_plug -->
		<dependency>
			<groupId>org.ansj</groupId>
			<artifactId>ansj_lucene7_plug</artifactId>
			<version>5.1.5.1</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.elasticsearch/elasticsearch -->
		<dependency>
			<groupId>org.elasticsearch</groupId>
			<artifactId>elasticsearch</artifactId>
			<version>6.4.2</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.elasticsearch.client/transport -->
		<dependency>
			<groupId>org.elasticsearch.client</groupId>
			<artifactId>transport</artifactId>
			<version>6.4.2</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.elasticsearch.plugin/transport-netty4-client -->
		<dependency>
			<groupId>org.elasticsearch.plugin</groupId>
			<artifactId>transport-netty4-client</artifactId>
			<version>6.4.2</version>
		</dependency>

java class:
/**
 *
 //https://www.elastic.co/downloads/elasticsearch
 //G:\app\elasticsearch-6.4.3
 //http://127.0.0.1:9200/
 //https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/index.html
 */
public class ESManager {

    private static TransportClient tclient  = null;
    /**
     * Parameter	                                Description
     client.transport.ignore_cluster_name           Set to true to ignore cluster name validation of connected nodes. (since 0.19.4)

     client.transport.ping_timeout                  The time to wait for a ping response from a node. Defaults to 5s.

     client.transport.nodes_sampler_interval        How often to sample / ping the nodes listed and connected. Defaults to 5s.

     */
    private static TransportClient getTransportClient(){
        Settings settings = Settings.builder()
                .put("cluster.name", "elasticsearch").build();

        // on startup
        TransportClient client = new PreBuiltTransportClient(settings);
        try {
            //默认端口号9300，url访问127.0.0.1:9200,端口不一样
//            String ip = "127.0.0.1";
            String ip = "192.168.3.163";
            InetAddress ia = InetAddress.getByName(ip);
            System.out.println(ia);
            client.addTransportAddress(new TransportAddress(ia, 9300));

        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
        // on shutdown
        //client.close();
        return client;
    }
    private static TransportClient getESClient(){
        if (tclient==null){
            System.setProperty("es.set.netty.runtime.available.processors", "false");
            tclient = getTransportClient();

        }
        return tclient;
    }



    /**
     * loc:存储位置：index，type,id
     * doc:存储文档内容
     *
     * 节点 说明
     _index	文档存储的地方--数据库
     _type	文档代表的对象的类--表
     _id	文档的唯一标识--id
     * @return
     */
    public static int index(String index, String type, String id,Map<String, Object> doc) {

        TransportClient client = getESClient();

//        Map<String, Object> json = new HashMap<String, Object>();
//        json.put("user","孙悟空");
//        json.put("postDate",new Date());
//        json.put("message","trying out Elasticsearch");


        //prepareIndex(String index, String type, @Nullable String id) {
        IndexRequestBuilder indexBuilder = client.prepareIndex(index,type,id);
        try {
            XContentBuilder contentBuilder = XContentFactory.jsonBuilder();
            Set<String> keys = doc.keySet();
            contentBuilder.startObject();
            for (String key:keys) {
                Object v = doc.get(key);
                contentBuilder.field(key, v);
            }
            contentBuilder.endObject();
            //IndexResponse response =  indexBuilder.setSource(map).get();
            IndexResponse response =  indexBuilder.setSource(contentBuilder).get();
            // Index name
            String _index = response.getIndex();
            // Type name
            String _type = response.getType();
             // Document ID (generated or not)
            String _id = response.getId();
            // Version (if it's the first time you index this document, you will get: 1)
            long _version = response.getVersion();
            // status has stored current instance statement.
            RestStatus restStatus = response.status();//RestStatus
            int status =  restStatus.getStatus();
            System.out.println(_index);
            System.out.println(_type);
            System.out.println(_id);
            System.out.println(_version);
            System.out.println(status);
            return  status;
        } catch (Exception e) {
            e.printStackTrace();
            return -1;
        }


    }
    public static Map<String,Object> get(String index, String type, String id) {
        TransportClient client = getESClient();
        GetRequestBuilder getBuilder = client.prepareGet(index,type,id);
        Map<String,Object>  map = getBuilder.get().getSource();
//        System.out.println(map);
        return map;
    }


    public static int delete(String index, String type, String id) {
        TransportClient client = getESClient();
        DeleteResponse response = client.prepareDelete(index,type,id).get();
        RestStatus restStatus = response.status();
        int status =  restStatus.getStatus();
        System.out.println(status);
//        return status==200;
        return status;
    }


    public static int update(){
        TransportClient client = getESClient();

        try {
            client.prepareUpdate("ttl", "doc", "1")
                    .setScript(new Script(ScriptType.INLINE,null,null,null))
                    .get();

            client.prepareUpdate("ttl", "doc", "1")
                    .setDoc(XContentFactory.jsonBuilder()
                            .startObject()
                            .field("gender", "male")
                            .endObject())
                    .get();
        } catch (Exception e) {
            e.printStackTrace();
        }

        return -1;
    }
    public static int update2(){
        TransportClient client = getESClient();
        UpdateRequest updateRequest = new UpdateRequest();
        updateRequest.index("index");
        updateRequest.type("_doc");
        updateRequest.id("1");
        try {
            updateRequest.doc(XContentFactory.jsonBuilder()
                    .startObject()
                    .field("gender", "male")
                    .endObject());
            client.update(updateRequest).get();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return -1;
    }
    //有改，无加
    public static void upsert(){
        TransportClient client = getESClient();
        try {
            IndexRequest indexRequest = new IndexRequest("index", "type", "1")
                    .source(XContentFactory.jsonBuilder()
                            .startObject()
                            .field("name", "Joe Smith")
                            .field("gender", "male")
                            .endObject());
            UpdateRequest updateRequest = new UpdateRequest("index", "type", "1")
                    .doc(XContentFactory.jsonBuilder()
                            .startObject()
                            .field("gender", "male")
                            .endObject())
                    .upsert(indexRequest);
            client.update(updateRequest).get();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    //多个查询
    public static void multiGet(){
        TransportClient client = getESClient();
        MultiGetResponse multiGetItemResponses = client.prepareMultiGet()
                .add("twitter", "_doc", "1")
                .add("twitter", "_doc", "2", "3", "4")
                .add("another", "_doc", "foo")
                .get();

        for (MultiGetItemResponse itemResponse : multiGetItemResponses) {
            GetResponse response = itemResponse.getResponse();
            if (response.isExists()) {
                String json = response.getSourceAsString();
            }
        }
    }
    //批量插入，删除
    public static void bulk(){
        TransportClient client = getESClient();
        BulkRequestBuilder bulkRequest = client.prepareBulk();

        try {
            // either use client#prepare, or use Requests# to directly build index/delete requests
            bulkRequest.add(client.prepareIndex("twitter", "_doc", "1")
                    .setSource(XContentFactory.jsonBuilder()
                            .startObject()
                            .field("user", "kimchy")
                            .field("postDate", new Date())
                            .field("message", "trying out Elasticsearch")
                            .endObject()
                    )
            );

            bulkRequest.add(client.prepareIndex("twitter", "_doc", "2")
                    .setSource(XContentFactory.jsonBuilder()
                            .startObject()
                            .field("user", "kimchy")
                            .field("postDate", new Date())
                            .field("message", "another post")
                            .endObject()
                    )
            );

            BulkResponse bulkResponse = bulkRequest.get();
            if (bulkResponse.hasFailures()) {
                // process failures by iterating through each bulk response item
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static Pagination<Map<String,Object>> search(QueryBuilder query, QueryBuilder filter, SortBuilder sort, PageParam page, String... indices){
        TransportClient client = getESClient();

//        SearchResponse response = client.prepareSearch("index1", "index2")
        SearchRequestBuilder searchBuilder = client.prepareSearch(indices);
        /**
         QUERY_THEN_FETCH:          查询是针对所有的块执行的，但返回的是足够的信息，而不是文档内容（Document）。结果会被排序和分级，基于此，只有相关的块的文档对象会被返回。由于被取到的仅仅是这些，故而返回的 hit 的大小正好等于指定的 size。这对于有许多块的 index 来说是很便利的（返回结果不会有重复的，因为块被分组了）
         QUERY_AND_FETCH:           最原始（也可能是最快的）实现就是简单的在所有相关的 shard上执行检索并返回结果。每个 shard 返回一定尺寸的结果。由于每个shard已经返回了一定尺寸的hit，这种类型实际上是返回多个 shard的一定尺寸的结果给调用者。
         DFS_QUERY_THEN_FETCH：      与 QUERY_THEN_FETCH 相同，预期一个初始的散射相伴用来为更准确的 score 计算分配了的term频率。
         DFS_QUERY_AND_FETCH:       与 QUERY_AND_FETCH 相同，预期一个初始的散射相伴用来为更准确的 score 计算分配了的term频率。
         SCAN：              在执行了没有进行任何排序的检索时执行浏览。此时将会自动的开始滚动结果集。
         COUNT：             只计算结果的数量，也会执行 facet。
         */
        searchBuilder.setSearchType(SearchType.DFS_QUERY_THEN_FETCH);

        if(query!=null){
            searchBuilder.setQuery(query) ;                // Query  QueryBuilder
        }
        if(filter!=null){
            searchBuilder.setPostFilter(filter);    // Filter
        }
        if(sort!=null){
            searchBuilder.addSort(sort);
        }

//        BoolQueryBuilder booleanQuery = QueryBuilders.boolQuery();
//        booleanQuery.must(QueryBuilders.matchPhraseQuery("title", "好"));
//        booleanQuery.must(QueryBuilders.matchPhraseQuery("title", "中"));

//        QueryBuilder q = QueryBuilders.termQuery("gender", 2);
//        searchBuilder.setQuery(QueryBuilders.termQuery("gender", 2)) ;                // Query  QueryBuilder
//        searchBuilder.setQuery(QueryBuilders.termQuery("age", 20)) ;                // Query  QueryBuilder
//        QueryBuilder filter = QueryBuilders.rangeQuery("age").from(12).to(100);
//        searchBuilder.setPostFilter(QueryBuilders.rangeQuery("gender").from(0).to(2));    // Filter
//        searchBuilder.setPostFilter(QueryBuilders.rangeQuery("age").from(12).to(100));    // Filter
//        searchBuilder.addSort("age", SortOrder.ASC);//字符串字段
//        searchBuilder.addSort("gender", SortOrder.ASC);
        int pageFirst = page.getPageFirst();
        int pageSize = page.getPageSize();
        searchBuilder.setFrom(pageFirst);
        searchBuilder.setSize(pageSize);
//        // 设置高亮显示--->设置相应的前置标签和后置标签
//        searchBuilder.setHighlighterPreTags("<span color='blue' size='18px'>");
//        searchBuilder.setHighlighterPostTags("</span>");
//        // 哪个字段要求高亮显示
//        searchBuilder.addHighlightedField("author");
//        searchBuilder.addHighlightedField("url");
//        //聚合group---https://www.cnblogs.com/xionggeclub/p/7975982.html
//        TermsBuilder teamAgg= AggregationBuilders.terms("player_count ").field("team");
//        sbuilder.addAggregation(teamAgg);

        searchBuilder.setExplain(true);// 设置是否按查询匹配度排序
//        SearchResponse response = searchBuilder.get();
        SearchResponse response = searchBuilder.execute().actionGet();

        SearchHits hits = response.getHits();
        long totalCount =  hits.totalHits;
        System.out.println(hits.totalHits);
        Pagination<Map<String,Object>> result = new Pagination<Map<String,Object>>();
        List<Map<String,Object>> list = new ArrayList<Map<String,Object>>();
        for (SearchHit hit : hits) {
            Map<String,Object> map = hit.getSourceAsMap();
            System.out.println(hit.getId()+"--"+hit.getScore());
            String json = hit.getSourceAsString();
            System.out.println(json);
            list.add(map);
        }
        //PageUtil.get();
        for (Map<String,Object> m:list) {
            System.out.println(m);
        }
        result.setList(list);
        PageParam pageParam = PageUtil.get(page.getPageNum(),pageSize,totalCount);
        result.setPageParam(pageParam);
        return result;
    }

    public static void analyze(){
        TransportClient client = getESClient();
        AnalyzeRequest ar = new AnalyzeRequest();
        ar.text("他叫孙悟空");
        ar.analyzer("standard");//standard,simple,cjk

        List<AnalyzeResponse.AnalyzeToken> tokens = client.admin().indices().analyze(ar).actionGet().getTokens();
        for (AnalyzeResponse.AnalyzeToken token : tokens)
        {
            System.out.println(token.getTerm());
        }

//        client.admin().indices().prepareCreate("twitter")
//                .setSettings(Settings.settingsBuilder().loadFromSource(jsonBuilder()
//                        .startObject()
//                        .startObject("analysis")
//                        .startObject("analyzer")
//                        .startObject("steak")
//                        .field("type", "custom")
//                        .field("tokenizer", "standard")
//                        .field("filter", new String[]{"snowball", "standard", "lowercase"})
//                        .endObject()
//                        .endObject()
//                        .endObject()
//                        .endObject().string()))
//                .execute().actionGet();
    }
    public static void main(String[] args) {
//        delete();
//        index();
        String index =  "aichat";
        String type = "doc";
        String id = UUIDUtil.uuid();
        Map<String, Object> doc = new LinkedHashMap<String,Object>();
        doc.put("username","赵翘楚");
        doc.put("password","123456");
        doc.put("age",12);
        doc.put("gender",2);

        index(index,type,id,doc);

        //http://127.0.0.1:9200/aichat/_search?q=username:%E4%B8%AD%E5%9B%BD

//        System.out.println("---------------");
//        Map<String,Object>  map = get(index,type,id);
//        System.out.println(map);
//        delete(index,type,"59fa162c45314ea2b0433fbf0b1517f2");

//        PageParam page = PageUtil.get(1,10);
//        BoolQueryBuilder booleanQuery = QueryBuilders.boolQuery();
//        booleanQuery.must(QueryBuilders.matchPhraseQuery("title", "好"));
//        booleanQuery.must(QueryBuilders.matchPhraseQuery("title", "中"));

//        QueryBuilder query = QueryBuilders.matchPhraseQuery("username", "中国");
//        QueryBuilder query = QueryBuilders.termQuery("gender", 2);
//        searchBuilder.setQuery(QueryBuilders.termQuery("gender", 2)) ;                // Query  QueryBuilder
//        searchBuilder.setQuery(QueryBuilders.termQuery("age", 20)) ;                // Query  QueryBuilder
//        QueryBuilder filter = QueryBuilders.rangeQuery("age").from(12).to(100);
//        SortBuilder sort = SortBuilders.fieldSort("age").order(SortOrder.DESC);
//        search(query,null,null,page,"aichat");

        //https://www.jianshu.com/p/77fc4d9e4a16
//        analyze();
//
    }
}
