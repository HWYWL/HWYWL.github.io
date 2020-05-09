---
title: 一个小小的RPC框架
date: 2020-05-01 17:41:19
tags: 
 - 高并发
 - 分布式
categories: 
 - 分布式
keywords: "分布式,高并发"
description: 一个小小的RPC框架
---

# 一个mini的RPC框架

### 说明
这是一个实验性的RPC框架，麻雀虽小但是五脏俱全，基本上RPC的有的功能基础功能都实现了，比如注册发现，反向代理调用等。

在使用上也非常简单只需几行的代码：
我们先定义一个接口是实现
```java
public interface CalcService {
    /**
     * 获取IP地址
     *
     * @return
     */
    String getIp();

    /**
     * 打招呼
     *
     * @param name 名称
     * @return
     */
    String hi(String name);
}
```
然后实现它
```java
public class CalcServiceImpl implements CalcService {

    @Override
    public String getIp() {
        String ip = null;
        try {
            InetAddress ip4 = Inet4Address.getLocalHost();
            ip = ip4.getHostAddress();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }

        return ip;
    }

    @Override
    public String hi(String name) {
        return "Hi " + name;
    }
}
```

现在就可以写我们的生产者和消费者了
**生产者**
```java
public class Server {
    public static void main(String[] args) {
        RpcServerConfig config = new RpcServerConfig();
        config.setPort(8080);

        RpcServer server = new RpcServer(config);
        server.register(CalcService.class, new CalcServiceImpl());
        server.start();
    }
}
```

**消费者**
```java
public class Client {
    public static void main(String[] args) {
        List<Peer> servers = Collections.singletonList(new Peer("127.0.0.1", 8080));
        RpcClientConfig config = new RpcClientConfig();
        config.setServers(servers);

        RpcClient client = new RpcClient(config);
        CalcService service = client.getProxy(CalcService.class);

        String ip = service.getIp();
        String hi = service.hi("美女");
        System.out.println(hi + " 我顺着 " + ip + " 去找你！");
    }
}
```

启动生产者
![JX3dVH.png](https://s1.ax1x.com/2020/05/01/JX3dVH.png)

运行消费者
![JX32qg.png](https://s1.ax1x.com/2020/05/01/JX32qg.png)

具体实现可以参考源码。
GitHub：https://github.com/HWYWL/mini-rpc

### 问题建议

- 联系我的邮箱：ilovey_hwy@163.com
- 我的博客：https://www.hwy.ac.cn
- GitHub：https://github.com/HWYWL