---
title: Excel表的读写
date: 2018-07-27 10:25:34
tags: 
 - Docker
 - Linux
categories: 
 - Docker
keywords: "Linux,Docker"
description: docker安装。
---
Docker从1.13版本之后采用时间线的方式作为版本号，分为社区版CE和企业版EE。 社区版是免费提供给个人开发者和小型团体使用的，企业版会提供额外的收费服务，比如经过官方测试认证过的基础设施、容器、插件等。

<!--more-->
## <font color="#9cc6ef">Centos7上安装docker</font>

<div id="cnblogs_post_body" class="blogpost-body" style="margin-bottom: 20px; word-break: break-word; font-family: Verdana, Geneva, Arial, Helvetica, sans-serif; font-size: 14.4px; background-color: rgb(238, 238, 238);">

<span style="font-size: 16px;">Docker从1.13版本之后采用时间线的方式作为版本号，分为社区版CE和企业版EE。</span>

<span style="font-size: 16px;">社区版是免费提供给个人开发者和小型团体使用的，企业版会提供额外的收费服务，比如经过官方测试认证过的基础设施、容器、插件等。</span>

<span style="font-size: 16px;">社区版按照stable和edge两种方式发布，每个季度更新stable版本，如17.06，17.09；每个月份更新edge版本，如17.09，17.10。</span>

##  一、安装docker

1、Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。

通过 **uname -r **命令查看你当前的内核版本

<div class="cnblogs_code" style="background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); padding: 5px; overflow: auto; margin: 5px 0px; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">

<pre style="margin-bottom: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;"> $ <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">uname</span> -r</pre>

</div>

<span style="font-size: 16px;">2、使用 `root` 权限登录 Centos。确保 yum 包更新到最新。</span>

<div class="cnblogs_code" style="background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); padding: 5px; overflow: auto; margin: 5px 0px; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">

<pre style="margin-bottom: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">$ <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">sudo</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">yum</span> update</pre>

</div>

<span style="font-size: 16px;">3、卸载旧版本(如果安装过旧版本的话)</span>

<div class="cnblogs_code" style="background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); padding: 5px; overflow: auto; margin: 5px 0px; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">

<pre style="margin-bottom: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">$ <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">sudo</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">yum</span> <span style="color: rgb(0, 0, 0); font-size: 12px !important; line-height: 1.5 !important;">remove docker  docker</span>-<span style="color: rgb(0, 0, 0); font-size: 12px !important; line-height: 1.5 !important;">common docker</span>-<span style="color: rgb(0, 0, 0); font-size: 12px !important; line-height: 1.5 !important;">selinux docker</span>-engine</pre>

</div>

<span style="font-size: 16px;">4、安装需要的软件包， </span><span style="font-size: 16px;">yum-util 提供yum-config-manager功能，</span><span class="pln" style="font-size: 16px;"><span class="com">另外两个是devicemapper驱动依赖的</span></span>

<div class="cnblogs_code" style="background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); padding: 5px; overflow: auto; margin: 5px 0px; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">

<pre style="margin-bottom: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">$ <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">sudo</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">yum</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">install</span> -y <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">yum</span>-utils device-mapper-persistent-data lvm2</pre>

</div>

5、设置yum源

<div class="cnblogs_code" style="background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); padding: 5px; overflow: auto; margin: 5px 0px; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">

<pre style="margin-bottom: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;"><span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;"><span style="color: rgb(0, 0, 0); font-size: 12px !important; line-height: 1.5 !important;">$</span> sudo</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">yum</span>-config-manager --add-repo https:<span style="color: rgb(0, 128, 0); font-size: 12px !important; line-height: 1.5 !important;">//</span><span style="color: rgb(0, 128, 0); font-size: 12px !important; line-height: 1.5 !important;">download.docker.com/linux/centos/docker-ce.repo</span></pre>

</div>

 ![](https://images2017.cnblogs.com/blog/1107037/201801/1107037-20180128094640209-1433322312.png)

6、可以查看所有仓库中所有docker版本，并选择特定版本安装

<div class="cnblogs_code" style="background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); padding: 5px; overflow: auto; margin: 5px 0px; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">

<pre style="margin-bottom: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">$ <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">yum</span> list docker-ce --showduplicates | <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">sort</span> -r</pre>

</div>

![](https://images2017.cnblogs.com/blog/1107037/201801/1107037-20180128095038600-772177322.png)

7、安装docker

<div class="cnblogs_code" style="background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); padding: 5px; overflow: auto; margin: 5px 0px; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">

<pre style="margin-bottom: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">$ <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">sudo</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">yum</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">install</span> docker-ce  <span style="color: rgb(0, 128, 0); font-size: 12px !important; line-height: 1.5 !important;">#由于repo中默认只开启stable仓库，故这里安装的是最新稳定版17.12.0</span> <span style="color: rgb(0, 0, 0); font-size: 12px !important; line-height: 1.5 !important;">$</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">sudo</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">yum</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">install</span> <FQPN>  <span style="color: rgb(0, 128, 0); font-size: 12px !important; line-height: 1.5 !important;"># 例如：sudo yum install docker-ce-17.12.0.ce</span></pre>

</div>

 ![](https://images2017.cnblogs.com/blog/1107037/201801/1107037-20180128103448287-493824081.png)

<span style="font-size: 16px;">8、启动并加入开机启动</span>

<div class="cnblogs_code" style="background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); padding: 5px; overflow: auto; margin: 5px 0px; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">

<pre style="margin-bottom: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">$ <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">sudo</span> <span style="color: rgb(0, 0, 0); font-size: 12px !important; line-height: 1.5 !important;">systemctl start docker
$</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">sudo</span> systemctl enable docker</pre>

</div>

<span style="font-size: 16px;">9、验证安装是否成功(有client和service两部分表示docker安装启动都成功了)</span>

<div class="cnblogs_code" style="background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); padding: 5px; overflow: auto; margin: 5px 0px; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">

<pre style="margin-bottom: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">$ <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">docker</span> version</pre>

</div>

![](https://images2017.cnblogs.com/blog/1107037/201801/1107037-20180128104046600-1053107877.png)

##  二、问题

<span style="font-size: 16px;">1、因为之前已经安装过旧版本的docker，在安装的时候报错如下：</span>

<div class="cnblogs_code" style="background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); padding: 5px; overflow: auto; margin: 5px 0px; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">

<pre style="margin-bottom: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;"><span style="color: rgb(0, 0, 0); font-size: 12px !important; line-height: 1.5 !important;">Transaction check error:</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">file</span> /usr/bin/docker from <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">install</span> of docker-ce-<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">17.12</span>.<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">0</span>.ce-<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">1</span>.el7.centos.x86_64 conflicts with <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">file</span> from package <span style="font-size: 15px; color: rgb(255, 0, 0); line-height: 1.5 !important;">**docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64**</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">file</span> /usr/bin/docker-containerd from <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">install</span> of docker-ce-<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">17.12</span>.<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">0</span>.ce-<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">1</span>.el7.centos.x86_64 conflicts with <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">file</span> from package <span style="font-size: 15px; color: rgb(255, 0, 0); line-height: 1.5 !important;">**docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64**</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">file</span> /usr/bin/docker-containerd-shim from <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">install</span> of docker-ce-<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">17.12</span>.<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">0</span>.ce-<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">1</span>.el7.centos.x86_64 conflicts with <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">file</span> from package <span style="font-size: 15px; color: rgb(255, 0, 0); line-height: 1.5 !important;">**docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64**</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">file</span> /usr/bin/dockerd from <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">install</span> of docker-ce-<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">17.12</span>.<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">0</span>.ce-<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">1</span>.el7.centos.x86_64 conflicts with <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">file</span> from package **<span style="font-size: 15px; color: rgb(255, 0, 0); line-height: 1.5 !important;">docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64</span>**</pre>

</div>

<span style="font-size: 16px;">2、卸载旧版本的包</span>

<div class="cnblogs_code" style="background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); padding: 5px; overflow: auto; margin: 5px 0px; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">

<pre style="margin-bottom: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">$ <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">sudo</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">yum</span> erase docker-common-<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">2</span>:<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">1.12</span>.<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">6</span>-<span style="color: rgb(128, 0, 128); font-size: 12px !important; line-height: 1.5 !important;">68</span>.gitec8512b.el7.centos.x86_64</pre>

</div>

![](https://images2017.cnblogs.com/blog/1107037/201801/1107037-20180128103145287-536100760.png)

<span style="font-size: 16px;">3、再次安装docker</span>

<div class="cnblogs_code" style="background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); padding: 5px; overflow: auto; margin: 5px 0px; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">

<pre style="margin-bottom: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">$ <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">sudo</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">yum</span> <span style="color: rgb(0, 0, 255); font-size: 12px !important; line-height: 1.5 !important;">install</span> docker-ce</pre>

</div>

<div id="comment_body_3991824" class="blog_comment_body" style="word-wrap: break-word; overflow: hidden;">4、推荐一种删除docker的方法：  

<div class="cnblogs_Highlighter sh-gutter">

<div id="highlighter_844319" class="syntaxhighlighter  csharp" style="width: 1613px; margin: 1em 0px !important; position: relative !important; overflow: auto !important; font-size: 1em !important; background-color: rgb(255, 255, 255) !important;">

<table border="0" cellpadding="0" cellspacing="0" style="width: 1613px; border-radius: 0px !important; background-image: none !important; background-position: initial !important; background-size: initial !important; background-repeat: initial !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; border: 0px !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.1em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; padding: 0px !important; position: static !important; right: auto !important; top: auto !important; vertical-align: baseline !important; box-sizing: content-box !important; font-family: Consolas, &quot;Bitstream Vera Sans Mono&quot;, &quot;Courier New&quot;, Courier, monospace !important; font-size: 12px !important; min-height: auto !important;">

<tbody style="border-radius: 0px !important; background: none !important; border: 0px !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.1em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; padding: 0px !important; position: static !important; right: auto !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 12px !important; min-height: auto !important;">

<tr style="border-radius: 0px !important; background: none !important; border: 0px !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.1em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; padding: 0px !important; position: static !important; right: auto !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 12px !important; min-height: auto !important;">

<td class="gutter" style="border-radius: 0px !important; background: none !important; border: 0px !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.1em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; position: static !important; right: auto !important; top: auto !important; vertical-align: baseline !important; width: 35px !important; box-sizing: content-box !important; font-size: 12px !important; min-height: auto !important; color: rgb(175, 175, 175) !important;">

<div class="line number1 index0 alt2" style="border-radius: 0px !important; background: none rgb(244, 244, 244) !important; border-width: 0px 2px 0px 0px !important; border-top-style: initial !important; border-right-style: solid !important; border-bottom-style: initial !important; border-left-style: initial !important; border-top-color: initial !important; border-right-color: rgb(108, 226, 108) !important; border-bottom-color: initial !important; border-left-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.8em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; padding: 0px 0.5em !important; position: static !important; right: auto !important; text-align: right !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 12px !important; min-height: auto !important; white-space: nowrap !important;">1</div>

<div class="line number2 index1 alt1" style="border-radius: 0px !important; background-image: none !important; background-position: initial !important; background-size: initial !important; background-repeat: initial !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; border-width: 0px 2px 0px 0px !important; border-top-style: initial !important; border-right-style: solid !important; border-bottom-style: initial !important; border-left-style: initial !important; border-top-color: initial !important; border-right-color: rgb(108, 226, 108) !important; border-bottom-color: initial !important; border-left-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.8em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; padding: 0px 0.5em !important; position: static !important; right: auto !important; text-align: right !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 12px !important; min-height: auto !important; white-space: nowrap !important;">2</div>

<div class="line number3 index2 alt2" style="border-radius: 0px !important; background: none rgb(244, 244, 244) !important; border-width: 0px 2px 0px 0px !important; border-top-style: initial !important; border-right-style: solid !important; border-bottom-style: initial !important; border-left-style: initial !important; border-top-color: initial !important; border-right-color: rgb(108, 226, 108) !important; border-bottom-color: initial !important; border-left-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.8em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; padding: 0px 0.5em !important; position: static !important; right: auto !important; text-align: right !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 12px !important; min-height: auto !important; white-space: nowrap !important;">3</div>

<div class="line number4 index3 alt1" style="border-radius: 0px !important; background-image: none !important; background-position: initial !important; background-size: initial !important; background-repeat: initial !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; border-width: 0px 2px 0px 0px !important; border-top-style: initial !important; border-right-style: solid !important; border-bottom-style: initial !important; border-left-style: initial !important; border-top-color: initial !important; border-right-color: rgb(108, 226, 108) !important; border-bottom-color: initial !important; border-left-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.8em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; padding: 0px 0.5em !important; position: static !important; right: auto !important; text-align: right !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 12px !important; min-height: auto !important; white-space: nowrap !important;">4</div>

</td>

<td class="code" style="border-radius: 0px !important; background: none !important; border: 0px !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.1em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; position: static !important; right: auto !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 12px !important; min-height: auto !important;">

<div class="container" style="border-radius: 0px !important; background: none !important; border: 0px !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.1em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; padding: 0px !important; position: relative !important; right: auto !important; top: auto !important; vertical-align: baseline !important; box-sizing: content-box !important; font-size: 12px !important; min-height: auto !important;">

<div class="line number1 index0 alt2" style="border-radius: 0px !important; background: none rgb(244, 244, 244) !important; border: 0px !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.8em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; padding: 0px 1em !important; position: static !important; right: auto !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 12px !important; min-height: auto !important; white-space: nowrap !important;">`yum remove docker docker-common docker-selinux docker-engine -y`</div>

<div class="line number2 index1 alt1" style="border-radius: 0px !important; background-image: none !important; background-position: initial !important; background-size: initial !important; background-repeat: initial !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; border: 0px !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.8em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; padding: 0px 1em !important; position: static !important; right: auto !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 12px !important; min-height: auto !important; white-space: nowrap !important;">`/etc/systemd -name ``'*docker*'` `-exec rm -f {} ;`</div>

<div class="line number3 index2 alt2" style="border-radius: 0px !important; background: none rgb(244, 244, 244) !important; border: 0px !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.8em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; padding: 0px 1em !important; position: static !important; right: auto !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 12px !important; min-height: auto !important; white-space: nowrap !important;">`find /etc/systemd -name ``'*docker*'` `-exec rm -f {} \;`</div>

<div class="line number4 index3 alt1" style="border-radius: 0px !important; background-image: none !important; background-position: initial !important; background-size: initial !important; background-repeat: initial !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; border: 0px !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.8em !important; margin: 0px !important; outline-style: initial !important; outline-width: 0px !important; overflow: visible !important; padding: 0px 1em !important; position: static !important; right: auto !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 12px !important; min-height: auto !important; white-space: nowrap !important;">`find /lib/systemd -name ``'*docker*'` `-exec rm -f {} \;`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

</div>

</div>