---
layout:     post
author:     zjh
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - 学习记录
---
## 一、镜像下载篇 
### 1、推荐国内华为镜像源 
https://mirrors.huaweicloud.com/
### 2、示例 推荐下载压缩版本
```
#elasticSearch 
https://mirrors.huaweicloud.com/elasticsearch/5.6.16/elasticsearch-5.6.16.tar.gz
#logstash
https://mirrors.huaweicloud.com/logstash/5.6.16/logstash-5.6.16.tar.gz
#kibana
https://mirrors.huaweicloud.com/kibana/5.6.16/kibana-5.6.16-darwin-x86_64.tar.gz
```
### 3、解压
```
tar -zxvf  logstash-5.6.16.tar.gz
```
## 二、配置启动篇

### 1、elasticsearch的启动，不能用root用户，需要建立一个其他用户，并授予权限
```
#打开Elasticsearch的配置文件：

vim config/elasticsearch.yml
#修改配置:

network.host=0.0.0.0
network.port=9200

#它默认就是这个配置，没有特殊要求，在本地不需要修改。

#启动Elasticsearch

./bin/elasticsearch

#启动成功，访问localhost:9200,网页显示：

{
  "name" : "56IrTCM",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "e4ja7vS2TIKI1BsggEAa6Q",
  "version" : {
    "number" : "5.2.2",
    "build_hash" : "f9d9b74",
    "build_date" : "2017-02-24T17:26:45.835Z",
    "build_snapshot" : false,
    "lucene_version" : "6.4.1"
  },
  "tagline" : "You Know, for Search"
}
#如果你的内存太小试试这个启动命令
ES_JAVA_OPTS="-Xms256M -Xmx256M -XX:ParallelGCThreads=1" ./bin/elasticsearch -d
#如果说内存太小，切换到root
sysctl -w vm.max_map_count=262144

```
### 2、logstash 启动
```
#在 logstash的主目录下：

vim config/logback.conf 

#修改 logback.conf 如下：

input {
#这里是log4j的整合方式
  log4j {
    mode => "server"
    host => "0.0.0.0"
    port => 4560
  }
#这里是logback的整合方式
    tcp {
        port => 4561
        codec => "json"
    }
}
filter {
  #Only matched data are send to output.
}
output {
    elasticsearch {
    action => "index"          #The operation on ES
    hosts  => "localhost:9200"   #ElasticSearch host, can be array.
    index  => "%{[appname]}"         #The index to write data to.
  }
}

#启动
nohup ./bin/logstash -f config/logback.conf  &
```
### 3、kibana杀进程和重启
```

#启动
nohub ./bin/kibana &
#杀进程 
ps -ef|grep node

```
## 三、整合篇
1、### pom.xml引入依赖
```
 <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>4.11</version>
        </dependency>
```
2、### logback.xml配置器  记得替换ip和端口，替换的是上面logstash内tcp的部分的端口
```
<!-- logstash 配置部分 appanme 根据实际情况修改 -->
	<appender name="stash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
		<destination>IP:Port</destination>
		<encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder" >
                        <!-- 此处的appname值，换成你自己的应用名，作为logstash的索引 -->
			<customFields>{"appname":"volunteersystem"}</customFields>
		</encoder>
	</appender>


#需要在哪里就在哪里加ref注入
<root level="debug">
		<appender-ref ref="stash" />
	</root>
```
## 四、测试效果篇
### 1、访问kibana面板
### 2、添加索引 management=》index partterns =》
<a href='http://img.zjhwork.xyz/elk/elk1.jpg' target='_blank'><img src='http://img.zjhwork.xyz/elk/elk1.jpg'></a>
### 3、查看日志信息  ps：必须让你的应用先产生日志信息，在elasticSearch内有索引记录才能记录到elk中
<a href='http://img.zjhwork.xyz/elk/elk2.jpg' target='_blank'><img src='http://img.zjhwork.xyz/elk/elk2.jpg'></a>
