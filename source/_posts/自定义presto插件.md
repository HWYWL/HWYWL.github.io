---
title: 自定义Presto插件
date: 2021-09-01 17:49:36
tags: 
 - 大数据
 - Presto
categories: 
 - 大数据
keywords: "大数据,Presto"
description: 自定义Presto插件。
---

### 遇到的问题
根据国家发布的规定，我们需要把用户隐私数据进行加密存储，例如手机号、IP地址。
我们的数仓是保存在AWS的S3存储系统中，数据通过Presto进行查询，很多时候找数据是需要精确到具体用户的。
但此时我们的用户数据已加密，而Presto又不支持解密，如果使用程序解密费时费力，所以自定义插件就闪亮登场了。

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/1630490684685-20191111446739_lMhbGS.png)

### 基础版本
我使用了AWS的EMR版本如下：
```
JDK：1.8.0_92
EMR版本：emr-5.30.1
Presto版本：0.232
```

```
JDK：1.8.0_201
EMR版本：emr-5.24.0
Presto版本：0.219
```
对了如果抄我的代码，请确认自己的是Presto维护作者，现在网上有两个版本的Presto，互相是不兼容的。
一个是我现在的使用的**com.facebook.presto**，还有一个是**io.prestosql**，注意注意。
我的程序的Presto SDK版本是0.232，测试是可以兼容低版本的，例如0.219。

### 编写程序
文件目录
```
G:.
├─.idea
└─src
    ├─main
    │  ├─java
    │  │  └─com
    │  │      └─yi
    │  │          └─udfs
    │  │              ├─model
    │  │              ├─scalar
    │  │              │  ├─cryptography
    │  │              │  ├─date
    │  │              │  ├─geo
    │  │              │  ├─json
    │  │              │  └─other
    │  │              └─utils
    │  └─resources
    │      └─META-INF
    │          └─services
    └─test
        └─java
            └─com
                └─yi
```

pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.yi</groupId>
    <artifactId>presto-third-udfs</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <java.version>1.8</java.version>
        <presto.version>0.232</presto.version>
        <slice.version>0.38</slice.version>
        <joda.version>2.8.2</joda.version>
        <airlift.version>0.189</airlift.version>
        <commons-codec.version>1.15</commons-codec.version>
        <junit.version>4.12</junit.version>
        <slf4j.version>1.7.25</slf4j.version>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/com.facebook.presto/presto-spi -->
        <dependency>
            <groupId>com.facebook.presto</groupId>
            <artifactId>presto-spi</artifactId>
            <version>${presto.version}</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>io.airlift</groupId>
            <artifactId>slice</artifactId>
            <version>${slice.version}</version>
        </dependency>

        <dependency>
            <groupId>joda-time</groupId>
            <artifactId>joda-time</artifactId>
            <version>${joda.version}</version>
        </dependency>

        <dependency>
            <groupId>com.facebook.airlift</groupId>
            <artifactId>json</artifactId>
            <version>${airlift.version}</version>
        </dependency>

        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>${commons-codec.version}</version>
        </dependency>

        <!-- Logger -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>log4j-over-slf4j</artifactId>
            <version>${slf4j.version}</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
        </dependency>
    </dependencies>

    <build>
        <finalName>presto-third-udfs</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-release-plugin</artifactId>
                <version>2.5.1</version>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <artifactSet>
                                <includes>
                                    <include>io.airlift:log</include>
                                    <include>joda-time:joda-time</include>
                                </includes>
                            </artifactSet>
                            <relocations>
                                <relocation>
                                    <pattern>io.airlift.log</pattern>
                                    <shadedPattern>io.airlift.log.shaded</shadedPattern>
                                </relocation>
                            </relocations>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

PrestoPlugin，继承Presto SPI的Plugin类，这样Presto启动才能加载,
程序会扫描com.yi.udfs.scalar包下的所有类，将带有特定注解的类加载在函数中。
```java
package com.yi.udfs;

import com.facebook.presto.spi.Plugin;
import com.google.common.collect.ImmutableSet;
import com.google.common.collect.Lists;
import com.google.common.collect.Sets;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.FileInputStream;
import java.io.IOException;
import java.net.URL;
import java.util.List;
import java.util.Objects;
import java.util.Set;
import java.util.zip.ZipEntry;
import java.util.zip.ZipInputStream;

/**
 * @author YI
 * @description
 * @date create in 2021/8/17 9:38
 */
public class PrestoPlugin implements Plugin {
    private static Logger logger = LoggerFactory.getLogger(PrestoPlugin.class);

    /**
     * 加载类
     * @return 返回类对象
     */
    public Set<Class<?>> getFunctions() {
        try {
            List<Class<?>> classes = getFunctionClasses();
            Set<Class<?>> set = Sets.newHashSet();
            for (Class<?> clazz : classes) {
                if (clazz.getName().startsWith("com.yi.udfs.scalar")) {
                    logger.info("加载函数: " + clazz);
                    set.add(clazz);
                }
            }
            return ImmutableSet.<Class<?>>builder().addAll(set).build();
        } catch (IOException e) {
            logger.error("无法从jar文件加载类!", e);
            return ImmutableSet.of();
        }
    }

    /**
     * 通过反射获取类对象
     * @return 类对象
     * @throws IOException
     */
    private List<Class<?>> getFunctionClasses() throws IOException {
        List<Class<?>> classes = Lists.newArrayList();
        String classResource = this.getClass().getName().replace(".", "/") + ".class";
        String jarURLFile = Objects.requireNonNull(Thread.currentThread().getContextClassLoader().getResource(classResource)).getFile();
        int jarEnd = jarURLFile.indexOf('!');
        // 这是URL格式，再次转换以获得实际的文件位置
        String jarLocation = jarURLFile.substring(0, jarEnd);
        jarLocation = new URL(jarLocation).getFile();

        ZipInputStream zip = new ZipInputStream(new FileInputStream(jarLocation));
        for (ZipEntry entry = zip.getNextEntry(); entry != null; entry = zip.getNextEntry()) {
            if (entry.getName().endsWith(".class") && !entry.isDirectory()) {
                String className = entry.getName().replace("/", ".");
                // 删除.class后缀
                className = className.substring(0, className.length() - 6);
                try {
                    classes.add(Class.forName(className));
                } catch (ClassNotFoundException e) {
                    logger.error("无法加载类{}，异常: {}", className, e);
                }
            }
        }
        return classes;
    }
}
```

AESFunctions，一个负责aes加解密的类库，即我们需要的可以在Presto使用的函数库。
```java
package com.yi.udfs.scalar.cryptography;

import com.facebook.presto.spi.function.Description;
import com.facebook.presto.spi.function.ScalarFunction;
import com.facebook.presto.spi.function.SqlType;
import com.facebook.presto.spi.type.StandardTypes;
import com.yi.udfs.utils.AESUtil;
import com.yi.udfs.utils.StringUtil;
import io.airlift.slice.Slice;
import io.airlift.slice.Slices;

/**
 * @author YI
 * @description 使用AES加密解密 AES-128-ECB加密
 * @date create in 2021/8/17 11:00
 */
public class AESFunctions {

    @Description("aes加密")
    @ScalarFunction("aes_encrypt")
    @SqlType(StandardTypes.VARCHAR)
    public static Slice aesEncrypt(@SqlType(StandardTypes.VARCHAR) Slice cSrc, @SqlType(StandardTypes.VARCHAR) Slice cKey) throws Exception {
        if (cSrc == null || StringUtil.empty(cSrc.toStringUtf8())) {
            return cSrc;
        }

        // 加密
        String enString = AESUtil.Encrypt(cSrc.toStringUtf8(), cKey);
        return Slices.utf8Slice(enString);
    }

    @Description("aes解密")
    @ScalarFunction("aes_decrypt")
    @SqlType(StandardTypes.VARCHAR)
    public static Slice aesDecrypt(@SqlType(StandardTypes.VARCHAR) Slice cSrc, @SqlType(StandardTypes.VARCHAR) Slice cKey) {
        if (cSrc == null || StringUtil.empty(cSrc.toStringUtf8())) {
            return cSrc;
        }
        // 解密
        String deString = AESUtil.Decrypt(cSrc.toStringUtf8(), cKey);
        return Slices.utf8Slice(deString);
    }
}
```
这里解释一下注解，
@Description：自定义函数的注释，让人知道这个函数干什么用的。
@ScalarFunction：我们写SQL时写这个函数才能调用我们自定义的方法
@SqlType：方法的返回值
在入参的@SqlType可以用来表示参数的类型，自定义函数的Slice就是我们Java的String。

### 部署

1.如果你也是使用AWS的EMR，并且集群已经在运行了，这个只能一个一个节点替换很麻烦。
```
1.我们把我们生成的jar文件复制到我们节点，我们先复制到临时目录，再从临时目录sudo复制到目标目录
	不能一步到位，sudo下载的文件没有权限复制到目标目录，当然如果你文件不放在S3，就比较简单了，直接第二条命令。
	/usr/bin/aws s3 cp s3://kylin-data/presto-third-udfs.jar /home/hadoop/
	sudo mv presto-third-udfs.jar /usr/lib/presto/plugin/hive-hadoop2/
2.重启presto，每个节点都需要重启，重启的文档如下：
	https://aws.amazon.com/cn/premiumsupport/knowledge-center/restart-service-emr/
```

2.如果你想集群启动时自动加载插件，启动完就可以使用，你可以如下操作，还是已AWS的EMR为例。
我写了一个脚本放在S3文件系统，如下：
```bash
#!/bin/bash
aws s3 cp s3://kylin-data/presto-third-udfs.jar /home/hadoop/
sudo mkdir -p /usr/lib/presto/plugin/hive-hadoop2/
sudo cp /home/hadoop/presto-third-udfs.jar /usr/lib/presto/plugin/hive-hadoop2/
exit 0
```
在网页控制台启动，需要在引导加入此脚本
![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/1630493337411-image.png)

程序启动集群(Java)
```
RunJobFlowRequest request = new RunJobFlowRequest()
    //集群名
    .withName(emrName)
    //emr的版本，指定之后不需要再指定上面组件的版本号(文件合并必须使用这个版本)
    .withReleaseLabel("emr-5.30.1")
    //集群工作步骤，集群启动之 后会自动执行，执行完可以自动关闭或者保持运行集群
    .withSteps(stepConfigs)
    //集群需要安装的组件,hive,hadoop
    .withApplications(applications)
    //emr日志路径
    .withLogUri("s3://apm-event-source-data/")
    //集群相关aws服务的iam角色
    .withServiceRole("EMR_DefaultRole")
    //组成集群的Ec2对应的iam角色
    .withJobFlowRole("EMR_EC2_DefaultRole")
    .withBootstrapActions(new BootstrapActionConfig()
        .withName("安装自定义插件")
        .withScriptBootstrapAction(new ScriptBootstrapActionConfig()
            .withPath("s3://kylin-data/copyJarFile.sh")))
	......
```
就是加入引导，在引导阶段吧我们插件复制到目标目录
```
.withBootstrapActions(new BootstrapActionConfig()
	.withName("安装自定义插件")
	.withScriptBootstrapAction(new ScriptBootstrapActionConfig()
        .withPath("s3://kylin-data/copyJarFile.sh")))
```

### 使用
公共加解密密钥：J4NwAAAAAAAAAAAA

Presto加解密使用
```
加密函数：select aes_encrypt('你好金科', 'J4NwAAAAAAAAAAAA');
	输出：KbAC1EtPwRSbNmS9oaBSuA==

解密函数：select aes_decrypt('KbAC1EtPwRSbNmS9oaBSuA==', 'J4NwAAAAAAAAAAAA');
	输出：你好世界
```

**小知识：**Hive是自带AES加解密函数的，使用如下：
```
加密函数：SELECT base64(aes_encrypt('你好世界', 'J4NwAAAAAAAAAAAA'));
	输出：KbAC1EtPwRSbNmS9oaBSuA==

解密函数：SELECT aes_decrypt(unbase64('KbAC1EtPwRSbNmS9oaBSuA=='), 'J4NwAAAAAAAAAAAA');
	输出：你好世界
```