---
title: Flink入门
date: 2019-07-15 02:33:08
tags: 
 - 大数据
 - Flink
categories: 
 - 大数据
keywords: "大数据,Flink"
description: Flink入门。
---

### 说明
这篇文章只是让你入门接触到流计算的强大。

### 下载安装Apache Flink
```
http://www.mirrorservice.org/sites/ftp.apache.org/flink/flink-1.8.0/flink-1.8.0-bin-scala_2.12.tgz
```

这里以windows为例子，解压后双击start-cluster.bat启动集群模式

![alt](https://i.loli.net/2019/05/14/5cda90be4116957755.png)

正常情况下，现在访问**http://localhost:8081**就能进入管理界面
![alt](https://i.loli.net/2019/05/14/5cda911a6bd4919200.png)

好，写下来我们编写我们的代码，我们可以从官方的工程开始，
生成Java项目的工程有两种方式，一种是maven生成
```
mvn archetype:generate								\
  -DarchetypeGroupId=org.apache.flink				\
  -DarchetypeArtifactId=flink-quickstart-java		\
  -DarchetypeVersion=${1:-1.8.0}							\
  -DgroupId=org.myorg.quickstart					\
  -DartifactId=$PACKAGE								\
  -Dversion=0.1										\
  -Dpackage=org.myorg.quickstart					\
  -DinteractiveMode=false
```
还有一种是使用官方提供的脚本：
```
curl https://flink.apache.org/q/quickstart.sh | bash
```

推荐使用第二种，方面快捷，官方负责维护，当然你执行的时候要看看自己flink的版本，因为脚本生成的工程pom.xml中flink的依赖会是最新的，可能不兼容你下载的flink版本。

我们把生成的工程导入IDEA，她的工程目录如下：
![alt](https://i.loli.net/2019/05/14/5cda92aa36bab37458.png)

好了我们选择StreamingJob来编写我们的代码，实现监控维基百科词条的更新，不过在此之前我们需要引入维基百科的详情资源。
```
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-wikiedits_2.11</artifactId>
    <version>${flink.version}</version>
</dependency>
```

然一顿骚操作之后编写好我们的代码
```java
public class StreamingJob {

    public static void main(String[] args) throws Exception {
        // 环境信息
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<WikipediaEditEvent> edits = env.addSource(new WikipediaEditsSource());

        //以用户名为key分组
        KeyedStream<WikipediaEditEvent, String> keyedStream = edits.keyBy((KeySelector<WikipediaEditEvent, String>) WikipediaEditEvent::getUser);

        //时间窗口为5秒
        SingleOutputStreamOperator<Tuple3<String, Integer, StringBuilder>> operator = keyedStream.timeWindow(Time.seconds(15)).aggregate(new AggregateFunction<WikipediaEditEvent, Tuple3<String, Integer, StringBuilder>, Tuple3<String, Integer, StringBuilder>>() {
            @Override
            public Tuple3<String, Integer, StringBuilder> createAccumulator() {
                //创建ACC
                return new Tuple3<>("", 0, new StringBuilder());
            }

            @Override
            public Tuple3<String, Integer, StringBuilder> add(WikipediaEditEvent wikipediaEditEvent, Tuple3<String, Integer, StringBuilder> stringIntegerStringBuilderTuple3) {
                StringBuilder sb = stringIntegerStringBuilderTuple3.f2;

                //如果是第一条记录，就加个"修改字符数量 ："作为前缀，
                //如果不是第一条记录，就用空格作为分隔符
                if(StringUtils.isBlank(sb.toString())){
                    sb.append("修改字符数量 : ");
                }else {
                    sb.append(" ");
                }

                return new Tuple3<>(wikipediaEditEvent.getUser(),
                        wikipediaEditEvent.getByteDiff() + stringIntegerStringBuilderTuple3.f1,
                        sb.append(wikipediaEditEvent.getByteDiff()));
            }

            @Override
            public Tuple3<String, Integer, StringBuilder> getResult(Tuple3<String, Integer, StringBuilder> stringIntegerStringBuilderTuple3) {
                return stringIntegerStringBuilderTuple3;
            }

            @Override
            public Tuple3<String, Integer, StringBuilder> merge(Tuple3<String, Integer, StringBuilder> stringIntegerStringBuilderTuple3, Tuple3<String, Integer, StringBuilder> acc1) {
                //合并窗口的场景才会用到
                return new Tuple3<>(stringIntegerStringBuilderTuple3.f0,
                        stringIntegerStringBuilderTuple3.f1 + acc1.f1, stringIntegerStringBuilderTuple3.f2.append(acc1.f2));
            }
        });

        // 将每个key的聚合结果单独转为字符串，实际应用中这里可以发送到kafka、mysql或者redis中
        operator.map((MapFunction<Tuple3<String, Integer, StringBuilder>, String>) Tuple3::toString).print();

        // 执行
        env.execute("Flink Streaming Java API Skeleton Start");
    }
}
```

然后我们使用maven的命令打包
```
mvn clean package -U
```
打包好之后回到我们Flink的管理界面上传jar包

![alt](https://i.loli.net/2019/05/14/5cda942cb5d0e74644.png)

填写类路径，然后提交
![alt](https://i.loli.net/2019/05/14/5cda945ae8a1b20114.png)

到这我们已经看到程序已经开始执行了
![alt](https://i.loli.net/2019/05/14/5cda947adf63983987.png)

然后打开我们启动start-cluster.bat是弹出的控制台窗口，是不是发现程序已经完美运行了呢
![alt](https://i.loli.net/2019/05/14/5cda95010b83d12061.png)

具体的API使用请查看官方文档。