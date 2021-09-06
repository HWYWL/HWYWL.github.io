---
title: 大数据文件Parquet
date: 2021-09-06 16:10:40
tags: 
 - 大数据
 - Parquet
categories: 
 - 大数据
keywords: "大数据,Parquet"
description: 大数据文件Parquet。
---

### 简介
是Hadoop的一种列存储格式；它提供了高效的数据存储和编码。使用Dremel文件中描述的记录分解和组装算法来表示嵌套结构，
下面是官方的解释。
> Apache Parquet is a columnar storage format available to any project in the Hadoop ecosystem, regardless of the choice of data processing framework, data model or programming language.

通俗点就是大数据全家桶的高效存储方式，它主要解决了如下几个问题：
- 把IO只给查询需要用到的数据，只加载需要被计算的列
- 空间节省，列式的压缩效果更好
- 可以针对数据类型进行编码(gzip、snappy)
- 开启矢量化的执行引擎(不再1条1条的处理数据，而是一次处理1024条数据)

### SDK封装
Parquet的具体原理我这边就不再赘述了，这边只讲使用。原生的api很难用，相当繁琐，所以我封装了一层。
只暴露简单的api使用，里面复杂的逻辑全部屏蔽掉

通过反射自动生成MessageType，自动拼装数据结构，遵循简单的原则，你给我数据我给你parquet文件。
目前支持的数据类型如下：
- int、Integer
- long、Long
- float、Float
- double、Double
- String

### 安装
**maven**
```
<dependency>
    <groupId>com.github.hwywl</groupId>
    <artifactId>parquet-plus</artifactId>
    <version>1.0.3-RELEASE</version>
</dependency>
```

**Gradle**
```
implementation 'com.github.hwywl:parquet-plus:1.0.3-RELEASE'
```

### 使用

可以查看github中的源码，不看也行，很简单的，使用如下：
```java
public class ParquetTest {
    String filePath = "F:\\a.parquet";

    /**
     * 写入一个数据到parquet文件
     *
     * @throws IOException
     * @throws CustomException
     */
    @Test
    public void ParquetWriteTest() throws IOException, CustomException {
        TestModel model = getModel();
        ParquetUtil.writerParquet(filePath, model);
    }

    /**
     * 写入多个数据到parquet文件
     *
     * @throws IOException
     * @throws CustomException
     */
    @Test
    public void ParquetWriteListTest() throws IOException, CustomException {
        List<TestModel> models = getModels();
        ParquetUtil.writerParquet(filePath, models);
    }

    /**
     * 将parquet文件转为对象集合
     *
     * @throws IOException
     */
    @Test
    public void ParquetReadBeanTest() throws IOException {
        List<TestModel> models = ParquetUtil.readParquetBean(filePath, TestModel.class);
        System.out.println(models);
    }

    /**
     * 将parquet文件转为Map集合
     *
     * @throws IOException
     */
    @Test
    public void ParquetReadMapTest() throws IOException {
        List<Map<String, Object>> maps = ParquetUtil.readParquetMap(filePath);
        System.out.println(maps);
    }

    /**
     * 将parquet文件转为对象集合,并只拿2条数据
     *
     * @throws IOException
     */
    @Test
    public void ParquetReadBeanMaxLineTest() throws IOException {
        List<TestModel> models = ParquetUtil.readParquetBean(filePath, 2, TestModel.class);
        System.out.println(models);
    }

    /**
     * 测试数据
     *
     * @return
     */
    private TestModel getModel() {
        return new TestModel(2, 3, 6L, "校花", 10);
    }

    /**
     * 测试数据
     *
     * @return
     */
    private List<TestModel> getModels() {
        List<TestModel> arrayList = new ArrayList<>();

        for (int i = 0; i < 10; i++) {
            arrayList.add(new TestModel(2, 3, 6L, "校花", 10));
        }

        return arrayList;
    }
}
```

### 1.0.3-RELEASE 版本更新
1. 开放底层api

### 1.0.2-RELEASE 版本更新
1. 修复bug
2. 增加更简便的API

### 1.0.1-RELEASE 版本更新
1. 增加生成parquet文件
2. 增加parquet文件读取

### 源码地址
GitHub：https://github.com/HWYWL/parquet-plus