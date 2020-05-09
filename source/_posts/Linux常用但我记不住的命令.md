---
title: Linux常用但我记不住的命令
date: 2019-02-26 13:50:08
tags: 
 - 操作系统
 - Linux
categories: 
 - Linux
keywords: "Linux,分区"
description: 有些命令不常用会经常忘记，所以先给他记下来。
---

Linux如何查看端口

1、lsof -i:端口号 用于查看某一端口的占用情况，比如查看8000端口使用情况，lsof -i:8000

<!--more-->
Linux如何查看端口

1、lsof -i:端口号 用于查看某一端口的占用情况，比如查看8000端口使用情况，lsof -i:8000

<div class="cnblogs_code" style="margin: 5px 0px; padding: 5px; background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); overflow: auto; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">  

<pre style="margin-bottom: 0px; padding: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;"># lsof -i:<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">8000</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);">COMMAND   PID USER   FD   TYPE  DEVICE SIZE</span>/<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);">OFF NODE NAME  
lwfs</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">22065</span> root    <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">6u</span>  IPv4 <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">4395053</span>      0t0  TCP *:irdmi (LISTEN)</pre>

</div>

可以看到8000端口已经被轻量级文件系统转发服务lwfs占用

2、netstat -tunlp |grep 端口号，用于查看指定的端口号的进程情况，如查看8000端口的情况，netstat -tunlp |grep 8000

<div class="cnblogs_code" style="margin: 5px 0px; padding: 5px; background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); overflow: auto; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">  

<div class="cnblogs_code_toolbar" style="margin: 5px 0px 0px; padding: 0px;"><span class="cnblogs_code_copy" style="margin: 0px; padding: 0px 5px 0px 0px; line-height: 1.8;"><a title="复制代码" style="margin: 0px; padding: 0px; color: rgb(0, 0, 0); border: none !important; background-color: rgb(245, 245, 245) !important;">![复制代码](https://common.cnblogs.com/images/copycode.gif)</a></span></div>

<pre style="margin-bottom: 0px; padding: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;"># netstat -<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);">tunlp 
Active Internet connections (only servers)
Proto Recv</span>-Q Send-Q Local Address               Foreign Address             State       PID/<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);">Program name   
tcp</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span>      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">111</span>                 <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:*                   LISTEN      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">4814</span>/<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);">rpcbind        
tcp</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span>      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">5908</span>                <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:*                   LISTEN      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">25492</span>/qemu-<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);">kvm      
tcp</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span>      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">6996</span>                <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:*                   LISTEN      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">22065</span>/<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);">lwfs          
tcp</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span>      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">192.168</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">122.1</span>:<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">53</span>            <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:*                   LISTEN      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">38296</span>/<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);">dnsmasq       
tcp</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span>      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">22</span>                  <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:*                   LISTEN      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">5278</span>/<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);">sshd           
tcp</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span>      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">127.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.1</span>:<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">631</span>               <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:*                   LISTEN      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">5013</span>/<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);">cupsd          
tcp</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span>      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">127.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.1</span>:<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">25</span>                <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:*                   LISTEN      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">5962</span>/<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);">master         
tcp</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span>      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">8666</span>                <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:*                   LISTEN      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">44868</span>/<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);">lwfs          
tcp</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span>      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">8000</span>                <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:*                   LISTEN      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">22065</span>/lwfs        </pre>

<div class="cnblogs_code_toolbar" style="margin: 5px 0px 0px; padding: 0px;"><span class="cnblogs_code_copy" style="margin: 0px; padding: 0px 5px 0px 0px; line-height: 1.8;"><a title="复制代码" style="margin: 0px; padding: 0px; color: rgb(0, 0, 0); border: none !important; background-color: rgb(245, 245, 245) !important;">![复制代码](https://common.cnblogs.com/images/copycode.gif)</a></span></div>

</div>

<div class="cnblogs_code" style="margin: 5px 0px; padding: 5px; background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); overflow: auto; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">  

<pre style="margin-bottom: 0px; padding: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;"># netstat -tunlp | <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 255);">grep</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">8000</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);">tcp</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span>      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0</span> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">8000</span>                <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>.<span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">0.0</span>:*                   LISTEN      <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(128, 0, 128);">22065</span>/lwfs          </pre>

</div>

说明一下几个参数的含义：

<div class="cnblogs_code" style="margin: 5px 0px; padding: 5px; background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); overflow: auto; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">  

<div class="cnblogs_code_toolbar" style="margin: 5px 0px 0px; padding: 0px;"><span class="cnblogs_code_copy" style="margin: 0px; padding: 0px 5px 0px 0px; line-height: 1.8;"><a title="复制代码" style="margin: 0px; padding: 0px; color: rgb(0, 0, 0); border: none !important; background-color: rgb(245, 245, 245) !important;">![复制代码](https://common.cnblogs.com/images/copycode.gif)</a></span></div>

<pre style="margin-bottom: 0px; padding: 0px; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;"> <span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(255, 0, 0);">-t (tcp) 仅显示tcp相关选项
                                 -u (udp)仅显示udp相关选项
                                 -n 拒绝显示别名，能显示数字的全部转化为数字
                                 -l 仅列出在Listen(监听)的服务状态
                                 -</span><span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(0, 0, 0);"><span style="margin: 0px; padding: 0px; line-height: 1.8; color: rgb(255, 0, 0);">p 显示建立相关链接的程序名</span></span> </pre>

<div class="cnblogs_code_toolbar" style="margin: 5px 0px 0px; padding: 0px;"><span class="cnblogs_code_copy" style="margin: 0px; padding: 0px 5px 0px 0px; line-height: 1.8;"><a title="复制代码" style="margin: 0px; padding: 0px; color: rgb(0, 0, 0); border: none !important; background-color: rgb(245, 245, 245) !important;">![复制代码](https://common.cnblogs.com/images/copycode.gif)</a></span></div>

</div>

后台启动一个jar：

<div class="cnblogs_code" style="margin: 5px 0px; padding: 5px; background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); overflow: auto; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">  

<pre style="padding: 0px; margin-bottom: 0px; line-height: 1.42857; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">nohup java -jar xxx.jar &    </pre>

</div>

<span style="font-family: &quot;Helvetica Neue&quot;, Helvetica, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, &quot;Noto Sans CJK SC&quot;, &quot;WenQuanYi Micro Hei&quot;, Arial, sans-serif; font-size: 13px;">tail 命令可用于查看文件的内容，有一个常用的参数</span> <span class="marked" style="border: 0px; margin: 0px; padding: 0.2em; background-color: rgb(236, 234, 230); border-radius: 3px; font-weight: bold; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; font-size: 13px;">-f</span> <span style="font-family: &quot;Helvetica Neue&quot;, Helvetica, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, &quot;Noto Sans CJK SC&quot;, &quot;WenQuanYi Micro Hei&quot;, Arial, sans-serif; font-size: 13px;">常用于查阅正在改变的日志文件。</span>

<div class="cnblogs_code" style="margin: 5px 0px; padding: 5px; background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); overflow: auto; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">  

<pre style="padding: 0px; margin-bottom: 0px; line-height: 1.42857; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">  

**命令格式：**  

<pre class="prettyprint prettyprinted" style="border-left-width: 4px; border-color: rgb(221, 221, 221); margin: 15px auto; padding: 10px 15px; font-variant-numeric: normal; font-variant-east-asian: normal; font-stretch: normal; font-size: 12px; line-height: 20px; font-family: Menlo, Monaco, Consolas, &quot;Andale Mono&quot;, &quot;lucida console&quot;, &quot;Courier New&quot;, monospace; white-space: pre-wrap; background: url(&quot;/images/codecolorer_bg.gif&quot;) center top rgb(251, 251, 251);"><span class="pln" style="border: 0px; margin: 0px; padding: 0px; color: rgb(0, 0, 0);">tail</span> <span class="pun" style="border: 0px; margin: 0px; padding: 0px; color: rgb(102, 102, 0);">[参数]</span> <span class="pln" style="border: 0px; margin: 0px; padding: 0px; color: rgb(0, 0, 0);"></span> <span class="pun" style="border: 0px; margin: 0px; padding: 0px; color: rgb(102, 102, 0);">[文件]</span> <span class="pln" style="border: 0px; margin: 0px; padding: 0px; color: rgb(0, 0, 0);"></span> </pre>

</pre>

</div>

**参数：**

<div class="cnblogs_code" style="margin: 5px 0px; padding: 5px; background-color: rgb(245, 245, 245); border: 1px solid rgb(204, 204, 204); overflow: auto; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">  

<pre style="padding: 0px; margin-bottom: 0px; line-height: 1.42857; white-space: pre-wrap; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">  

 _-f 循环读取_   -q 不显示处理信息  
 _-v 显示详细的处理信息_   -c<数目> 显示的字节数  
 _-n<行数> 显示行数_   --pid=PID 与-f合用,表示在进程ID,PID死掉之后结束.  
 _-q, --quiet, --silent 从不输出给出文件名的首部_   -s, --sleep-interval=S 与-f合用,表示在每次反复的间隔休眠S秒  

</pre>

</div>

**实例**

要显示 notes.log 文件的最后 10 行，请输入以下命令：

<pre class="prettyprint prettyprinted" style="border-left-width: 4px; border-color: rgb(221, 221, 221); margin: 15px auto; padding: 10px 15px; font-variant-numeric: normal; font-variant-east-asian: normal; font-stretch: normal; font-size: 12px; line-height: 20px; font-family: Menlo, Monaco, Consolas, &quot;Andale Mono&quot;, &quot;lucida console&quot;, &quot;Courier New&quot;, monospace; white-space: pre-wrap; background: url(&quot;/images/codecolorer_bg.gif&quot;) center top rgb(251, 251, 251);"><span class="pln" style="border: 0px; margin: 0px; padding: 0px; color: rgb(0, 0, 0);">tail notes</span><span class="pun" style="border: 0px; margin: 0px; padding: 0px; color: rgb(102, 102, 0);">.</span><span class="pln" style="border: 0px; margin: 0px; padding: 0px; color: rgb(0, 0, 0);">log</span></pre>

要跟踪名为 notes.log 的文件的增长情况，请输入以下命令：

<pre class="prettyprint prettyprinted" style="border-left-width: 4px; border-color: rgb(221, 221, 221); margin: 15px auto; padding: 10px 15px; font-variant-numeric: normal; font-variant-east-asian: normal; font-stretch: normal; font-size: 12px; line-height: 20px; font-family: Menlo, Monaco, Consolas, &quot;Andale Mono&quot;, &quot;lucida console&quot;, &quot;Courier New&quot;, monospace; white-space: pre-wrap; background: url(&quot;/images/codecolorer_bg.gif&quot;) center top rgb(251, 251, 251);"><span class="pln" style="border: 0px; margin: 0px; padding: 0px; color: rgb(0, 0, 0);">tail</span> <span class="pun" style="border: 0px; margin: 0px; padding: 0px; color: rgb(102, 102, 0);">-</span><span class="pln" style="border: 0px; margin: 0px; padding: 0px; color: rgb(0, 0, 0);">f notes</span><span class="pun" style="border: 0px; margin: 0px; padding: 0px; color: rgb(102, 102, 0);">.</span><span class="pln" style="border: 0px; margin: 0px; padding: 0px; color: rgb(0, 0, 0);">log</span></pre>

此命令显示 notes.log 文件的最后 10 行。当将某些行添加至 notes.log 文件时，tail 命令会继续显示这些行。 显示一直继续，直到您按下（Ctrl-C）组合键停止显示。

vmstat - 虚拟内存统计

vmstat 命令报告有关进程、内存、分页、块 IO、中断和 CPU 活动等信息。


```
# vmstat 3
```
输出示例：

```
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu------
 r b swpd free buff cache si so bi bo in cs us sy id wa st
 0 0 0 2540988 522188 5130400 0 0 2 32 4 2 4 1 96 0 0
 1 0 0 2540988 522188 5130400 0 0 0 720 1199 665 1 0 99 0 0
 0 0 0 2540956 522188 5130400 0 0 0 0 1151 1569 4 1 95 0 0
 0 0 0 2540956 522188 5130500 0 0 0 6 1117 439 1 0 99 0 0
 0 0 0 2540940 522188 5130512 0 0 0 536 1189 932 1 0 98 0 0
 0 0 0 2538444 522188 5130588 0 0 0 0 1187 1417 4 1 96 0 0
 0 0 0 2490060 522188 5130640 0 0 0 18 1253 1123 5 1 94 0 0

```

2.找出占用内存资源最多的前 10 个进程

```
ps -auxf | sort -nr -k 4 | head -10
```

3.找出占用 CPU 资源最多的前 10 个进程


```
ps -auxf | sort -nr -k 3 | head -10
```

### 常用私活
查找文件**all-server.log**中含**deviceId**有的内容并显示出来
```
find -type f -name 'all-server.log'|xargs grep "deviceId"
```
统计**aaa**在文件**all-server.log**出现的次数
```
cat all-server.log |grep "aaa"|wc -l
```