# elasticsearch

download elasticsearch 6.4.2 and kibana 6.4.2,version must be same.

https://www.elastic.co/downloads/past-releases

unzip elasticsearch.zip and kibana.zip

use chinese analyzer:ansj

download elasticsearch-analysis-ansj

https://github.com/NLPchina/elasticsearch-analysis-ansj

user chrome brower download https://github.com/NLPchina/elasticsearch-analysis-ansj/releases/download/v6.4.2/elasticsearch-analysis-ansj-6.4.2.0-release.zip  （because net download too slowly）

copy zip to web project:http://127.0.0.1:8080/app/elasticsearch-analysis-ansj-6.4.2.0-release.zip

in elasticsearch folder execute command:

./bin/elasticsearch-plugin install http://127.0.0.1:8080/app/elasticsearch-analysis-ansj-6.4.2.0-release.zip

version must be same.

# update ..\elasticsearch-6.4.2\config\elasticsearch.yml

#---------------------------------- Network -----------------------------------

#network.host: 192.168.0.1

network.host: 192.168.3.1

#--------------------------------- Discovery ----------------------------------

#discovery.zen.ping.unicast.hosts: ["host1", "host2"]

discovery.zen.ping.unicast.hosts: ["192.168.3.1","127.0.0.1","192.168.3.2"]



# update ..\kibana-6.4.2\config\kibana.yml

elasticsearch.url: "http://192.168.3.1:9200"

# start run

elasticsearch-6.4.2\bin\elasticsearch.bat

kibana-6.4.2\bin\kibana.bat

http://192.168.3.1:9200/index_name/_search?q=username:%E4%B8%AD%E5%9B%BD

http://localhost:5601

# java maven project pom.xml

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
go to see ESManager.java
after add index,wait several seconds,then you can search that.

# elasticsearch6.2.2+ansj5.1.6+spring3.1.2

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
			<version>6.2.2</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.elasticsearch.client/transport -->
		<dependency>
			<groupId>org.elasticsearch.client</groupId>
			<artifactId>transport</artifactId>
			<version>6.2.2</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.elasticsearch.plugin/transport-netty4-client -->
		<dependency>
			<groupId>org.elasticsearch.plugin</groupId>
			<artifactId>transport-netty4-client</artifactId>
			<version>6.2.2</version>
		</dependency>


		<!-- https://mvnrepository.com/artifact/org.springframework.data/spring-data-elasticsearch -->
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-elasticsearch</artifactId>
			<version>3.1.2.RELEASE</version>
		</dependency>
