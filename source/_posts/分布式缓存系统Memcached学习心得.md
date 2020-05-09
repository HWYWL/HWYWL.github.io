---
title: 分布式缓存系统Memcached学习心得
date: 2019-08-18 10:19:36
tags: 
 - 缓存
 - Memcached
categories: 
 - 缓存
keywords: "缓存,Memcached"
description: 分布式缓存系统Memcached学习心得。
---

```
package com.danga.MemCached.test;    
   
import java.util.Date;    
   
import com.danga.MemCached.MemCachedClient;    
import com.danga.MemCached.SockIOPool;    
   
   
public class Test {        
    protected static MemCachedClient mcc = new MemCachedClient();       
       
    static {       
        String[] servers ={"192.168.40.4:12000"};       
       
        Integer[] weights = { 3 };       
       
        //创建一个实例对象SockIOPool     
        SockIOPool pool = SockIOPool.getInstance();       
       
        // set the servers and the weights    
        //设置Memcached Server    
        pool.setServers( servers );       
        pool.setWeights( weights );       
       
        // set some basic pool settings       
        // 5 initial, 5 min, and 250 max conns       
        // and set the max idle time for a conn       
        // to 6 hours       
        pool.setInitConn( 5 );       
        pool.setMinConn( 5 );       
        pool.setMaxConn( 250 );       
        pool.setMaxIdle( 1000 * 60 * 60 * 6 );       
       
        // set the sleep for the maint thread       
        // it will wake up every x seconds and       
        // maintain the pool size       
        pool.setMaintSleep( 30 );       
       
        // Tcp的规则就是在发送一个包之前，本地机器会等待远程主机    
                  // 对上一次发送的包的确认信息到来；这个方法就可以关闭套接字的缓存，    
                  // 以至这个包准备好了就发；    
                  pool.setNagle( false );       
        //连接建立后对超时的控制    
                  pool.setSocketTO( 3000 );    
        //连接建立时对超时的控制    
                  pool.setSocketConnectTO( 0 );       
       
        // initialize the connection pool       
        //初始化一些值并与MemcachedServer段建立连接    
                  pool.initialize();    
               
       
        // lets set some compression on for the client       
        // compress anything larger than 64k       
        mcc.setCompressEnable( true );       
        mcc.setCompressThreshold( 64 * 1024 );       
    }       
           
    public static void bulidCache(){       
        //set(key,value,Date) ,Date是一个过期时间，如果想让这个过期时间生效的话，这里传递的new Date(long date) 中参数date，需要是个大于或等于1000的值。    
        //因为java client的实现源码里是这样实现的 expiry.getTime() / 1000 ，也就是说，如果 小于1000的值，除以1000以后都是0，即永不过期    
        mcc.set( "test", "This is a test String" ,new Date(11211));   
    //十秒后过期    
              
    }       
      
    public static void output() {       
        //从cache里取值    
        String value = (String) mcc.get( "test" );       
        System.out.println(value);        
    }       
           
    public static void main(String[] args){       
        bulidCache();      
        output();           
    }     
       
} 
```