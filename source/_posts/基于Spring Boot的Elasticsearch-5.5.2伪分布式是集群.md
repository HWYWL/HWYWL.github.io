---
title: 基于Spring Boot的Elasticsearch-5.5.2伪分布式是集群
date: 2018-05-25 17:11:31
tags: 
 - Spring Boot
 - Elasticsearch
categories: 
 - 搜索引擎
keywords: "Spring Boot,Elasticsearch"
description: 基于Spring Boot的Elasticsearch-5.5.2伪分布式是集群。
---

这就是我踩过的坑

我们先去官网下载elasticsearch

Windows：https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.0.zip

Linux：https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.0.tar.gz

<!--more-->
### 这就是我踩过的坑

我们先去官网下载elasticsearch

    Windows：https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.0.zip

    Linux：https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.0.tar.gz

解压后我们复制三份出来分别命名为：elasticsearch-5.5.2-noed1、elasticsearch-5.5.2-noed2、elasticsearch-5.5.2-noed3  
然后我们进入各个里面的config文件夹里面找到 elasticsearch.yml 文件，在最后添加上如下配置，IP地址换为自己电脑的IP地址。

elasticsearch-5.5.2-noed1：

    #节点1的配置信息：  
    cluster.name: elasticsearch   #集群名称，保证唯一  
    node.name: Elasticsearch-node1   #节点名称，必须不一样  
    network.host: 192.168.0.103   #必须为本机的ip地址  
    http.port: 9200   #服务端口号，在同一机器下必须不一样  
    transport.tcp.port : 9300   #集群间通信端口号，在同一机器下必须不一样  
    #设置集群自动发现机器ip集合  
    discovery.zen.ping.unicast.hosts: ["192.168.0.103:9300", "192.168.0.103:9301", "192.168.0.103:9302"]

elasticsearch-5.5.2-noed2：

    #节点1的配置信息：  
    cluster.name: elasticsearch   #集群名称，保证唯一  
    node.name: Elasticsearch-node1   #节点名称，必须不一样  
    network.host: 192.168.0.103   #必须为本机的ip地址  
    http.port: 9201   #服务端口号，在同一机器下必须不一样  
    transport.tcp.port : 9301   #集群间通信端口号，在同一机器下必须不一样  
    #设置集群自动发现机器ip集合  
    discovery.zen.ping.unicast.hosts: ["192.168.0.103:9300", "192.168.0.103:9301", "192.168.0.103:9302"]

elasticsearch-5.5.2-noed3：

    #节点1的配置信息：  
    cluster.name: elasticsearch   #集群名称，保证唯一  
    node.name: Elasticsearch-node1   #节点名称，必须不一样  
    network.host: 192.168.0.103   #必须为本机的ip地址  
    http.port: 9202   #服务端口号，在同一机器下必须不一样  
    transport.tcp.port : 9302   #集群间通信端口号，在同一机器下必须不一样  
    #设置集群自动发现机器ip集合  
    discovery.zen.ping.unicast.hosts: ["192.168.0.103:9300", "192.168.0.103:9301", "192.168.0.103:9302"]

配置好之后就可以依次启动了，在bin目录下有启动脚本。

## 注意：IP地址别用localhost 或者 127.0.0.1 这是一个深坑w(ﾟДﾟ)w

启动之后我们安装ElasticSearch Head这个插件，如果你用谷歌浏览器又能翻墙的话建议直接到谷歌商店去下载，这样很方便，如果你不具备以上条件请自行百度安装方法（以下截图为谷歌商店）：

![](http://www.hwy.ac.cn/upload/2018/05/1c23o36pbag4uon0nulpcmjt3d.png)

以下是测试安装是否正确：  
![](http://www.hwy.ac.cn/upload/2018/05/s16kodfgjgjpoq7sqr7mpe7a63.png)

接下来我们通过Spring Boot编写操作这个集群的Java代码，先看看我代码的目录结构：

![](http://www.hwy.ac.cn/upload/2018/05/35di1ipmniif2r029a8q47eqaq.png)

以下是我们的代码和配置文件，使用maven工程构建，我的maven版本为：apache-maven-3.3.9，JDK版本：jdk1.8.0_111  
代码如下：

pom.xml
```js
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.yi</groupId>
  <artifactId>spring-boot-elasticsearch</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-boot-elasticsearch</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <spring-boot.version>2.0.0.RELEASE</spring-boot.version>
    <hutool-all.version>4.0.2</hutool-all.version>
    <log4j.version>2.11.0</log4j.version>
    <spring-data-elasticsearch.version>3.0.3.RELEASE</spring-data-elasticsearch.version>
    <jna.version>4.5.1</jna.version>
    <junit.version>4.12</junit.version>
  </properties>

  <dependencies>
    <!--spring boot框架-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>${spring-boot.version}</version>
    </dependency>

    <!--Spring Boot视图模板-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-freemarker</artifactId>
      <version>${spring-boot.version}</version>
    </dependency>

    <!--配置文件读取-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-configuration-processor</artifactId>
      <version>${spring-boot.version}</version>
    </dependency>

    <!--elasticsearch -->
    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-elasticsearch</artifactId>
      <version>${spring-data-elasticsearch.version}</version>
    </dependency>

    <dependency>
      <groupId>net.java.dev.jna</groupId>
      <artifactId>jna</artifactId>
      <version>${jna.version}</version>
    </dependency>

    <!--elasticsearch 必须依赖日志才能启动-->
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-api</artifactId>
      <version>${log4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
      <version>${log4j.version}</version>
    </dependency>

    <!--hutool 工具类-->
    <dependency>
      <groupId>cn.hutool</groupId>
      <artifactId>hutool-all</artifactId>
      <version>${hutool-all.version}</version>
    </dependency>

    <!--单元测试工具-->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>${junit.version}</version>
    </dependency>

  </dependencies>
</project>

```  

application.properties

    # ELASTICSEARCH (ElasticsearchProperties})
    #集群名。(默认值: elasticsearch)
    spring.data.elasticsearch.cluster-name=elasticsearch
    #集群其他节点地址列表，用逗号分隔。如果没有指定，就启动一个客户端节点。默认 9300 是 Java 客户端的端口。
    spring.data.elasticsearch.cluster-nodes=192.168.0.103:9300
    #是否开启本地（我本地测试用就启用本地了）
    spring.data.elasticsearch.local=true
    #开启 Elasticsearch 仓库。(默认值:true。)
    spring.data.elasticsearch.repositories.enabled=true
    #存储索引的位置
    #spring.data.elasticsearch.properties.path.home=data/elasticsearch
    #日志存储目录
    #spring.data.elasticsearch.properties.path.logs=./elasticsearch/log
    #数据存储目录
    #spring.data.elasticsearch.properties.path.data=./elasticsearch/data
    #连接超时的时间
    #spring.data.elasticsearch.properties.transport.tcp.connect_timeout=120s

Book.java

    package com.yi.model;

    import org.springframework.data.annotation.Id;
    import org.springframework.data.elasticsearch.annotations.Document;

    @Document(indexName = "book_index", type = "books")
    public class Book {

        @Id
        private String id;

        private String title;

        private String content;

        private String author;

        private String releaseDate;

        public Book() {
        }

        public Book(String id, String title, String content, String author, String releaseDate) {
            this.id = id;
            this.title = title;
            this.content = content;
            this.author = author;
            this.releaseDate = releaseDate;
        }

        忽略get和set方法，自己加上
    }

BookRepository.java

    package com.yi.repository;

    import com.yi.model.Book;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.Pageable;
    import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

    import java.util.List;

    /**
     * 描述:
     *
     * @author YI
     * @date 2018-3-21 12:08:48
     **/
    public interface BookRepository extends ElasticsearchRepository<Book, String> {

        Page<Book> findByAuthor(String author, Pageable pageable);

        List<Book> findByTitle(String title);

        Book save(Book book);
    }

BookService.java

    package com.yi.service;

    import com.yi.model.Book;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.PageRequest;

    import java.util.List;

    /**
     * 描述:
     *
     * @author YI
     * @date 2018-3-21 12:08:48
     **/
    public interface BookService {

        Book save(Book book);

        void delete(Book book);

        Iterable<Book> findAll();

        Page<Book> findByAuthor(String author, PageRequest pageRequest);

        List<Book> findByTitle(String title);
    }

BookServiceImpl.java

    package com.yi.service.impl;

    import com.yi.model.Book;
    import com.yi.repository.BookRepository;
    import com.yi.service.BookService;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.PageRequest;
    import org.springframework.stereotype.Service;

    import java.util.List;

    /**
     * 描述:
     *
     * @author YI
     * @date 2018-3-21 12:08:48
     **/
    @Service
    public class BookServiceImpl implements BookService {

        @Autowired
        private BookRepository bookRepository;

        @Override
        public Book save(Book book) {
            return bookRepository.save(book);
        }

        @Override
        public void delete(Book book) {
            bookRepository.delete(book);
        }

        @Override
        public Iterable<Book> findAll() {
            return bookRepository.findAll();
        }

        @Override
        public Page<Book> findByAuthor(String author, PageRequest pageRequest) {
            return bookRepository.findByAuthor(author, pageRequest);
        }

        @Override
        public List<Book> findByTitle(String title) {
            return bookRepository.findByTitle(title);
        }

    }

ElasticSearchController.java

    package com.yi.controller;

    import com.yi.model.Book;
    import com.yi.service.BookService;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.PageRequest;
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.ResponseBody;

    import java.util.ArrayList;
    import java.util.List;

    /**
     * ElasticSearch操作
     * @author YI
     * @date 2018-3-21 12:08:48
     */
    @Controller
    @RequestMapping("/elasticsearch")
    public class ElasticSearchController {

        @Autowired
        private BookService bookService;

        /**
         * 保存到elasticsearch中
         */
        @RequestMapping("/save")
        public void save(){
            for (int i = 0; i < getTitle().size(); i++) {
                bookService.save(new Book(String.valueOf(i), getTitle().get(i), getContent().get(i), "黄文懿", i + "-FEB-2017"));
            }
        }

        /**
         * 通过作者查找
         * http://localhost:8080/elasticsearch/findByAuthor?name=黄文懿
         * @param name
         * @return
         */
        @RequestMapping("/findByAuthor")
        @ResponseBody
        public Page<Book> findByAuthor(String name){
            return bookService.findByAuthor(name, new PageRequest(0, 10));
        }

        /**
         * 通过标题查找
         * @return
         */
        @RequestMapping("/findByTitle")
        @ResponseBody
        public List<Book> findByTitle(){
            return bookService.findByTitle("如梦令");
        }

        private List<String> getTitle() {
            List<String> list = new ArrayList<>();

            list.add("《如梦令·常记溪亭日暮》");
            list.add("《醉花阴·薄雾浓云愁永昼》");
            list.add("《声声慢·寻寻觅觅》");
            list.add("《永遇乐·落日熔金》");
            list.add("《如梦令·昨夜雨疏风骤》");
            list.add("《渔家傲·雪里已知春信至》");
            list.add("《点绛唇·蹴[1]罢秋千》");
            list.add("《点绛唇·寂寞深闺》");
            list.add("《蝶恋花·泪湿罗衣脂粉满》");
            list.add("《蝶恋花 离情》");
            list.add("《浣溪沙》");
            list.add("《浣溪沙》");
            list.add("《浣溪沙》");
            list.add("《浣溪沙》");
            list.add("《浣溪沙》");
            list.add("《减字木兰花·卖花担上》");
            list.add("《临江仙·欧阳公作《蝶恋花》");
            list.add("《临江仙·庭院深深深几许》");
            list.add("《念奴娇·萧条庭院》");
            list.add("《菩萨蛮·风柔日薄春犹早》");
            list.add("《菩萨蛮·归鸿声断残云碧》");
            list.add("《武陵春·风住尘香花已尽》");
            list.add("《一剪梅·红藕香残玉蕈秋》");
            list.add("《渔家傲·天接云涛连晓雾》");
            list.add("《鹧鸪天·暗淡轻黄体性柔》");
            list.add("《鹧鸪天·寒日萧萧上锁窗》");
            list.add("《一剪梅·红藕香残玉簟秋》");
            list.add("《如梦令·常记溪亭日暮》");
            list.add("《浣溪沙》");
            list.add("《浣溪沙》");
            list.add("《浣溪沙》");
            list.add("《蝶恋花·泪湿罗衣脂粉满》");
            list.add("《蝶恋花·暖日晴风初破冻》");
            list.add("《鹧鸪天·寒日萧萧上锁窗》");
            list.add("《醉花阴·薄雾浓云愁永昼》");
            list.add("《鹧鸪天·暗淡轻黄体性柔》");
            list.add("《蝶恋花·永夜恹恹欢意少》");
            list.add("《浣溪沙》");
            list.add("《浣溪沙》");
            list.add("《如梦令·谁伴明窗独坐》");

            return list;
        }

        private List<String> getContent() {
            List<String> list = new ArrayList<>();

            list.add("初中 宋·李清照 常记溪亭日暮，沉醉不知归路，兴尽晚回舟，误入藕花深处。争渡，争渡");
            list.add("重阳节 宋·李清照 薄雾浓云愁永昼，瑞脑消金兽。佳节又重阳，玉枕纱厨，半夜凉初透。东");
            list.add("闺怨诗 宋·李清照 寻寻觅觅，冷冷清清，凄凄惨惨戚戚。乍暖还寒时候，最难将息。三杯两");
            list.add("元宵节 宋·李清照 落日熔金，暮云合璧，人在何处。染柳烟浓，吹梅笛怨，春意知几许。元");
            list.add("婉约诗 宋·李清照 昨夜雨疏风骤，浓睡不消残酒，试问卷帘人，却道海棠依旧。知否，知否");
            list.add("描写梅花 宋·李清照 雪里已知春信至，寒梅点缀琼枝腻，香脸半开娇旖旎，当庭际，玉人浴出");
            list.add(" 宋·李清照 蹴罢秋千，起来慵整纤纤手。露浓花瘦，薄汗轻衣透。见客入来，袜刬金");
            list.add("闺怨诗 宋·李清照 寂寞深闺，柔肠一寸愁千缕。惜春春去。几点催花雨。倚遍阑干，只是无");
            list.add("婉约诗 宋·李清照 泪湿罗衣脂粉满。四叠阳关，唱到千千遍。人道山长水又断。萧萧微雨闻");
            list.add("描写春天 宋·李清照 暖雨晴风初破冻，柳眼梅腮，已觉春心动。酒意诗情谁与共？泪融残粉花");
            list.add("寒食节 宋·李清照 淡荡春光寒食天，玉炉沈水袅残烟，梦回山枕隐花钿。海燕未来人斗草，");
            list.add(" 宋·李清照 髻子伤春慵更梳，晚风庭院落梅初，淡云来往月疏疏，玉鸭薰炉闲瑞脑，");
            list.add(" 宋·李清照 莫许杯深琥珀浓，未成沉醉意先融。疏钟已应晚来风。瑞脑香消魂梦断，");
            list.add("闺怨诗 宋·李清照 小院闲窗春已深，重帘未卷影沉沉。倚楼无语理瑶琴，远岫出山催薄暮。");
            list.add("爱情诗 宋·李清照 绣幕芙蓉一笑开，斜偎宝鸭亲香腮，眼波才动被人猜。一面风情深有韵，");
            list.add("描写春天 宋·李清照 卖花担上，买得一枝春欲放。泪染轻匀，犹带彤霞晓露痕。怕郎猜道，奴");
            list.add("》 宋·李清照 欧阳公作《蝶恋花》，有“深深深几许”之句，予酷爱之。用其语作“庭");
            list.add("描写梅花 宋·李清照 庭院深深深几许，云窗雾阁春迟，为谁憔悴损芳姿。夜来清梦好，应是发");
            list.add("寒食节 宋·李清照 萧条庭院，又斜风细雨，重门须闭。宠柳娇花寒食近，种种恼人天气。险");
            list.add("思乡诗 宋·李清照 风柔日薄春犹早，夹衫乍著心情好。睡起觉微寒，梅花鬓上残。故乡何处");
            list.add("描写春天 宋·李清照 归鸿声断残云碧，背窗雪落炉烟直。烛底凤钗明，钗头人胜轻。角声催晓");
            list.add("闺怨诗 宋·李清照 风住尘香花已尽，日晚倦梳头。物是人非事事休，欲语泪先流。闻说双溪");
            list.add(" 宋·李清照 红藕香残玉蕈秋，轻解罗裳，独上兰舟。云中谁寄锦书来？雁字回时，月");
            list.add("豪放诗 宋·李清照 天接云涛连晓雾。星河欲转千帆舞。仿佛梦魂归帝所。闻天语。殷勤问我");
            list.add("描写花 宋·李清照 暗淡轻黄体性柔。情疏迹远只香留。何须浅碧深红色，自是花中第一流。");
            list.add("描写秋天 宋·李清照 寒日萧萧上琐窗，梧桐应恨夜来霜。酒阑更喜团茶苦，梦断偏宜瑞脑香。");
            list.add("闺怨诗 宋·李清照 红藕香残玉簟秋。轻解罗裳，独上兰舟。云中谁寄锦书来？雁字回时，月");
            list.add(" 宋·李清照 常记溪亭日暮。沈醉不知归路。兴尽晚回舟，误入藕花深处。争渡。争渡");
            list.add(" 宋·李清照 莫许杯深琥珀浓。未成沈醉意先融。已应晚来风。瑞脑香消魂梦断，");
            list.add(" 宋·李清照 小院闲窗春色深。重帘未卷影沈沈。倚楼无语理瑶琴。远岫出山催薄暮，");
            list.add(" 宋·李清照 淡荡春光寒食天。玉炉沈水袅残烟。梦回山枕隐花钿。海燕未来人斗草，");
            list.add(" 宋·李清照 泪湿罗衣脂粉满。四叠阳关，唱到千千遍。人道山长山又断。萧萧微雨闻");
            list.add(" 宋·李清照 暖日晴风初破冻。柳眼眉腮，已觉春心动。酒意诗情谁与共。泪融残粉花");
            list.add(" 宋·李清照 寒日萧萧上锁窗。梧桐应恨夜来霜。酒阑更喜团茶苦，梦断偏宜瑞脑香。");
            list.add(" 宋·李清照 薄雾浓云愁永昼。瑞脑消金兽。佳节又重阳，玉枕纱厨，半夜凉初透。东");
            list.add(" 宋·李清照 暗淡轻黄体性柔。情疏迹远只香留。何须浅碧深红色，自是花中第一流。");
            list.add(" 宋·李清照 永夜恹恹欢意少。空梦长安，认取长安道。为报今年春色好。花光月影宜");
            list.add(" 宋·李清照 髻子伤春慵更梳。晚风庭院落梅初。淡云来往月疏疏。玉鸭熏炉闲瑞脑，");
            list.add(" 宋·李清照 绣面芙蓉一笑开。斜飞宝鸭衬香腮。眼波才动被人猜。一面风情深有韵，");
            list.add(" 宋·李清照 谁伴明窗独坐，我共影儿俩个。灯尽欲眠时，影也把人抛躲。无那，无那");

            return list;
        }
    }

Application.java

    package com.yi;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;

    /**
     * Spring boot 启动程序
     * @author YI
     * @date 2018-3-15 15:49:59
     */
    @SpringBootApplication
    public class Application {

        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    }

写好之后启动吧，通过浏览器或者其他工具访问你想访问的接口吧，下图就会访问保存数据接口后elasticsearch中的数据，可以通过elasticsearch-head访问：

![](http://www.hwy.ac.cn/upload/2018/05/3gssim7b6egrkqt0uipknom811.png)