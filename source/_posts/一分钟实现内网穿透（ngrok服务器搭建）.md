---
title: 一分钟实现内网穿透（ngrok服务器搭建）
date: 2019-08-28 16:16:08
tags: 
 - 文件系统
categories: 
 - 文件系统
keywords: "文件系统,go-fastdfs"
description: 一分钟实现内网穿透（ngrok服务器搭建）。
---


简单来说内网穿透的目的是：让外网能访问你本地的应用，例如在外网打开你本地http://127.0.0.1指向的Web站点。

最近公司的花生壳到期了，要续费，发现价格一直在涨，都是5年以上的老用户，旗舰版都没有实现内网完全穿透，打算自己动手替换这个服务，中间走了不少的弯路，这里记录一些文字为大家提供参考。

随着开发与运行移动互联网的应用越来越多对打通内外网的需要也更加迫切，如微信开发、IOS与Android开发等。

虽然租用VPS、ECS等服务器可以解决很多问题但高性能的外网服务器价格非常贵还有数据安全问题，我选择的是公网服务器仅做代理与轻量应用，复杂的应用部署到内网服务器再穿透访问。

# 一、内网穿透概要
为了理解内网穿透我们先来了解几个概念：
### 1.1、IP地址
**网络中唯一定位一台设备的逻辑地址**，类似我们的电话号码

在互联网中我们访问一个网站或使用一个网络服务最终都需要通过IP定位到每一台主机，如访问baidu网站：
![image](8664A94F41774DC99906C452E415CCB2)

其中119.75.213.61就是一个公网的IP地址，他最终指向了一台服务器。

IP地址是IP协议提供的一种统一的地址格式，它为互联网上的每一个网络和每一台主机分配一个逻辑地址，以此来屏蔽物理地址的差异。

内网IP可以同时出现在多个不同的局域网络中，如A公司的U1用户获得了192.168.0.5，B公司的U3用户也可以获得192.168.0.5；但公网IP是唯一的，因为我们只有一个Internet。


```
//局域网可使用的网段（私网地址段）有三大段：
10.0.0.0~10.255.255.255（A类）
172.16.0.0~172.31.255.255（B类）
192.168.0.0~192.168.255.255（C类）
```

### 1.2、域名
**域名是IP的别名，便于记忆，域名最终通过DNS解析成IP地址。**
![image](3E6ACF5E2C4D41F4940ACED7AFB22B47)

IP V4是一个32位的数字，IP V6有128位，要记住一串毫无意义的数字非常困难，域名解决了这个问题。

如www.zhangguo.com.cn就是一个域名，cn表示地区，com表示商业机构，zhangguo是公司名称，www是主机名

![image](9D1A55DD04844185AC7E5452B5D33E06)

DNS查询过程如下，最终将域名变成IP地址

![image](A6376FD5CBA5458DA96E89B782C3A46B)

### 1.3、NAT
**NAT（Network Address Translation）即网络地址转换，NAT能将其本地地址转换成全球IP地址。**

内网的一些主机本来已经分配到了本地IP地址（如局域网DHCP分配的IP），但现在又想和因特网上的主机通信（并不需要加密）时，可使用NAT方法。

通过使用少量的公有IP 地址代表较多的私有IP 地址的方式，将有助于减缓可用的IP地址空间的枯竭。

NAT不仅能解决了lP地址不足与共享上网的问题，而且还能够有效地避免来自网络外部的攻击，隐藏并保护网络内部的计算机。

多路由器可完成NAT功能。

NAT的实现方式：

**静态转换**：是指将内部网络的私有IP地址转换为公有IP地址，IP地址对是一对一。

**动态转换**：是指将内部网络的私有IP地址转换为公用IP地址时，IP地址是不确定的，是随机的。

**端口多路复用（Port address Translation,PAT)**：内部网络的所有主机均可共享一个合法外部IP地址实现对Internet的访问，从而可以最大限度地节约IP地址资源。同时又可隐藏网络内部的所有主机，有效避免来自internet的攻击。因此，目前网络中应用最多的就是端口多路复用方式。

**应用程序级网关技术（Application Level Gateway）ALG**：传统的NAT技术只对IP层和传输层头部进行转换处理，ALG它能对这些应用程序在通信时所包含的地址信息也进行相应的NAT转换。

![image](2900DB8519DA4E409ACFE364F0A978FA)

![image](2732EDAFFCA54EEF9BCFC61277E051F5)

### 1.4、Proxy
Proxy即代理，被广泛应用于计算机领域，主要分为正向代理与反向代理：

### 1.4.1、正向代理
比如X花店代A,B,C,D,E五位男生向Candy女生送匿名的生日鲜花，这里的X花店就是5位顾客的代理，花店代理的是客户，隐藏的是客户。这就是我们常说的代理。

**正向代理隐藏了真实的请求客户端**。服务端不知道真实的客户端是谁，客户端请求的服务都被代理服务器代替来请求，某些科学上网工具扮演的就是典型的正向代理角色。用浏览器访问http://www.google.com时被墙了，于是你可以在国外搭建一台代理服务器，让代理帮我去请求google.com，代理把请求返回的相应结构再返回给我。

![image](68B6D7FE9B274D80921C7818B07A6368)

当多个客户端访问服务器时服务器不知道真正访问自己的客户端是那一台。正向代理中,proxy和client同属一个LAN,对server透明;

![image](EA650A1BACF649A99ACCE08E979B031C)

### 1.4.2、反向代理

拨打10086客服电话，接线员可能有很多个，调度器会智能的分配一个接线员与你通话。这里的调度器就是一个代理，只不过他代理的是接线员，客户端不能确定真正与自己通话的人，隐藏与保护的是目标对象。

**反向代理隐藏了真实的服务端**，当我们请求 ww.baidu.com 的时候，就像拨打10086一样，背后可能有成千上万台服务器为我们服务，但具体是哪一台，你不知道，也不需要知道，你只需要知道反向代理服务器是谁就好了，ww.baidu.com 就是我们的反向代理服务器，反向代理服务器会帮我们把请求转发到真实的服务器那里去。Nginx就是性能非常好的反向代理服务器，用来做负载均衡。

![image](42F044A8FCFF4F958D1360040B858017)

反向代理中,proxy和server同属一个LAN,对client透明。

![image](9EB94628908B4E058157079856FF19D4)

[了解更多关于代理内容请点击这里。](https://www.zhihu.com/question/24723688)

### 1.5、DDNS
DDNS即动态域名解析，是**将用户的动态IP地址映射到一个固定的域名解析服务上**，用户每次连接网络的时候，客户端程序就会通过信息传递把该主机的动态IP地址传送给位于服务商主机上的服务器程序，服务程序负责提供DNS服务并实现动态域名解析。就是说DDNS捕获用户每次变化的IP地址，然后将其与域名相对应，这样域名就可以始终解析到非固定IP的服务器上，互联网用户通过本地的域名服务器获得网站域名的IP地址，从而可以访问网站的服务。

### 1.6、为什么需要内网穿透

**当内网中的主机没有静态IP地址要被外网稳定访问时可以使用内网穿透**

在互联网中唯一定位一台主机的方法是通过公网的IP地址，但固定IP是一种非常稀缺的资源，不可能给每个公司都分配一个，且许多中小公司不愿意为高昂的费用买单，多数公司直接或间接的拨号上网，电信部门会给接入网络的用户分配IP地址，以前上网用户少的时候基本分配的都是临时的静态IP地址，租约过了之后可能会更换成另一个IP地址，这样外网访问就不稳定，因为内网的静态IP地址一直变化，为了解决这个问题可以使用动态域名解析的办法变换域名指向的静态IP地址。但是现在越来越多的上网用户使得临时分配的静态IP地址也不够用了，电信部门开始分配一些虚拟的静态IP地址，这些IP是公网不能直接访问的，如以125开头的一些IP地址，以前单纯的动态域名解析就不好用了。

### 1.7、内网穿透的定义与障碍

**简单来说实现不同局域网内的主机之间通过互联网进行通信的技术叫内网穿透。**

**障碍一**：位于局域网内的主机有两套 IP 地址，一套是局域网内的 IP 地址，通常是动态分配的，仅供局域网内的主机间通信使用；一套是经过网关转换后的外网 IP 地址，用于与外网程序进行通信。

![image](69A07B79D4664CD6855F990F0C909E06)

**障碍二**：位于不同局域网内的两台主机，即使是知道了对方的 IP 地址和端口号，“一厢情愿”地将数据包发送过去，对方也是接收不到的。

因为出于安全起见，除非是主机主动向对方发出了连接请求（这时会在该主机的数据结构中留下一条记录），否则，当主机接收到数据包时，如果在其数据结构中查询不到对应的记录，那些不请自来的数据包将会被丢弃。

![image](827A49D0A8554955AD7545F87F9C00C5)

**解决办法：要想解决以上两大障碍，我们需要借助一台具有公网 IP 的服务器进行桥接。**

# 二、常见的内网穿透产品
### 2.1、花生壳
花生壳既是内网穿透软件、内网映射软件,也是端口映射软件。规模最大，较正规，完善。

收费高，使用简单

官网：http://www.oray.com/

![image](5B96B046290A43D8863388092F3372FF)

### 2.2、Nat123

nat123是内网端口映射与动态域名解析软件，在内网启动映射后，可在外网访问连接内网网站等应用。整个网站我都没有找到客服电话，网友发了一些反面的评价

收费，使用简单

官网：http://www.nat123.com

![image](4B0F818B2E3F4E62972B350D4314602F)

###2.3、NATAPP
NATAPP基于ngrok的国内内网穿透服务，免费版会强制更换域名，临时用一下可以

收费，使用简单

官网：https://natapp.cn/

![image](44E177A4213040C694C0D0A1B2BC8457)

### 2.4、frp与其它
frp 是一个高性能的反向代理应用，可以帮助您轻松地进行内网穿透，对外网提供服务，支持 tcp, http, https 等协议类型，并且 web 服务支持根据域名进行路由转发。

开源免费

使用相对复杂，需要代理服务器支持

官网：https://github.com/fatedier/frp

文档：查看帮助文档，简书示例

利用处于内网或防火墙后的机器，对外网环境提供 http 或 https 服务。

对于 http, https 服务支持基于域名的虚拟主机，支持自定义域名绑定，使多个域名可以共用一个80端口。

利用处于内网或防火墙后的机器，对外网环境提供 tcp 和 udp 服务，例如在家里通过 ssh 访问处于公司内网环境内的主机。

![image](0B56DA9B6F234F9994895B56DCA3731B)

因为frp 仍然处于前期开发阶段，未经充分测试与验证，不推荐用于生产环境，所有我选择了ngrok，资料比较多。

还有如圣剑内网通、ngrok（开源免费）、[更多办法](https://post.smzdm.com/p/564494/)

# 三、ngrok
ngrok是一个反向代理，通过在公共的端点和本地运行的Web服务器之间建立一个安全的通道。ngrok可捕获和分析所有通道上的流量，便于后期分析与响应。

开源免费

官网：https://ngrok.com/

源码：https://github.com/inconshreveable/ngrok

![image](E968976674174F4FA0CFB4B00BFDF2C3)

ngrok1.x开源，ngrok2.x不开源

ngrok使用go语言开发，源代码分为客户端与服务器端。

国内免费服务器：http://ngrok.ciqiuwl.cn/，更多免费服务器请大家挖掘，资源共享，我随时更新：）

如果有服务器，仅客户端的使用是不复杂的，以上面的免费服务器为示例完成内网穿透

现在假定我的本地已成功部署了一个网站，访问地址为127.0.0.1，想内网穿透后被公网上的用户访问，一般步骤如下：

**步骤1、下载windows版本的客户端，解压。一般在为你提供代理服务器的网站上找你要下载的客户端**：

![image](3A2C462626574DBAB583D9E268EB75B7)

**步骤2、在命令（cmd）行下进入到ngrok客户端目录下**

![image](7E69F7DFF2D24E01B550D4DD87E4CEDA)

**步骤3、执行 ngrok -config=ngrok.cfg -subdomain xxx 80 //(xxx 是你自定义的域名前缀)，建议批处理**

![image](9685F6FB19094A9BA68AEC600E271C6F)

如果连接成功，会提示如下信息：

![image](07351B0A640041CA91D262D0753AF557)

**这一步如果你认为太麻烦，可以直接运行目录下的start.bat批处理文件就不用进DOS环境了。运行start.bat直接跳过2，3步**

**步骤4、如果开启成功 你就可以使用 xxx.ngrok.xiaomiqiu.cn 来访问你本机的 127.0.0.1:80 的服务了，当然你必须确定的是你本机的Web是可以正常访问的。**

![image](655B1027C1C64EA69D7084952CB263A0)

注意：

如果你自己有顶级域名，想通过自己的域名来访问本机的项目，那么先将自己的顶级域名解析到120.25.161.137(域名需要已备案哦，80端口必须备案)，然后执行 ngrok -config=ngrok.cfg -hostname xxx.xxx.xxx 80 //(xxx.xxx.xxx是你自定义的顶级域名)

# 四、ubuntu下生成ngrok服务器主程序

### 4.1、步骤与先决条件
如果你只是临时穿透或调试用，到第三步基本就可以了，但如果想作为稳定的商业服务，用别人的服务器还是受制于人，这里我们准备搭建自己的ngrok服务器。大致的步骤如下：

![image](962FF6F12F72406884DD234CB2A7BC46)


ngrok服务器可以是多种平台，如windows、Linux（CentOS、Debian、Ubuntu等）、Mac OS等。

**编译源代码生成应用强烈建议大家使用linux环境，windows肯定可以成功，但非常麻烦，我在windows操作系统上兜了一个大圈圈。**

**先决条件**：

a)、您有一台公网上的服务器，如阿里云的ECS

b)、您有一个域名，最好ICP备案成功，不然80端口没有办法使用，不过像微信开发是不使用80端口的，可以用nginx代理转换。

### 4.2、安装ubuntu操作系统
在linux环境下编译ngrok的源代码比windows下 方便很多，这里我们选择使用ubuntu，获得ubuntu的方法有如下几种：

1）、全新安装ubuntu系统

2）、申请VPS服务器， 阿里云、腾讯云、华为云、百度云、新浪云等，仅编译一下这种方法不错

3）、在虚拟机中安装ubuntu系统

综合考虑我选择了在虚拟机中安装ubuntu操作系统

![image](E2AB70E76091449BAE331FE9C37392CF)

### 4.2.1、安装VMware虚拟机
VMware Workstation是一款功能强大的虚拟机软件，在不影响本机操作系统的情况下，用户可以在虚拟机中同时运行不同版本的操作系统，用于开发、测试以及部署工作。

VMware Workstation 12 pro下载：VMware-workstation-full-12.1.0-3272444.exe

序列号：5A02H-AU243-TZJ49-GTC7K-3C61N（商业应用请购买正式版权，这里仅为学习使用）

1）、双击VMware Workstation 12安装文件，或者右键管理员身份打开，提示是否允许更改，点击是；

2）、打开VMware安装向导，点击下一步；

![image](5D1DDC4380914E949ED507F3F975746E)

3）、VMware Workstation 12激活步骤：

- 　　方法一、首次开启直接输入上文密钥，即可激活；
- 　　方法二、首次开启选择试用，进入试用后按一下步骤激活：

　　a、打开虚拟机主界面，点击“帮助”—“输入许可证密钥”；
　　
![image](0DBAE0E80B3A44B3BEA056A3E912E893)

b、在密钥输入框输入永久许可证密钥5A02H-AU243-TZJ49-GTC7K-3C61N，确定；[更多](http://www.mamicode.com/info-detail-1509515.html)

### 4.2.2、安装ubuntu到虚拟机

1)、下载ubuntu操作系统镜像

下载地址：https://www.ubuntu.com/download/desktop

这里我下载的是ubuntu-16.04.3-desktop-amd64.iso

![image](752C59929F2F4478A49D794CDDD0F85F)

2)、在VMware中安装ubuntu

打开VMware点击“创建新的虚拟机”

![image](4EEFA54B30624B5A8A0E7DA78615CC33)

向导选择自定义

![image](D6A423A2F49A48EC800E5208F289171B)

然后下一步再下一步，直到这里，稍后再安装系统

![image](B18AED600BC74E7EA71A0FFEDD7A7C2C)

后面设置处理器和内存的，电脑配置好的可以试试，否则采用默认的，博主这里是采用默认的，然后下一，直到这里，选择将虚拟机存储为单个磁盘：

![image](5C6A9F1634C74BFBA75C5505D8606B22)

个人建议至少20G硬盘空间，内存建议给1.5G，当然也要看电脑本身的配置，1G的内存跑起来比较卡。

其它的步骤比较简单，更多细节可以参考这里，《[VMware Ubuntu安装详细过程](http://blog.csdn.net/u013142781/article/details/50529030)》。

### 4.2.3、配置ubuntu系统

当ubuntu系统安装成功后，在虚拟机中可以启动ubuntu系统，启动后的系统如下：

![image](5255A726ABB64A549CFAB8762302F3A8)

ubuntu系统的使用还是有许多内容的，这里需要设置的内容如下：

a)、设置上网

就是在ubuntu中可以访问外网，可以使用多种形式

![image](DBA023A822174FB0A6FA077452F7A904)

b)、设置语言

可以选择使用中文版的ubuntu语言环境

![image](B45A03098DF24217AEF6F47D535AC43A)

c)、设置屏幕分辨率

如果不设置默认的屏幕比较小

![image](9316E79558094B0DBCCC7AF5C7AFD4C6)

d)、设置以root超级管理员的身份登录

许多操作要求管理身份

![image](1F474D045AD5422C89E31D64FBB04ECE)

e)、安装VMware Tools工具

只有在VMware虚拟机中安装好了VMware Tools，才能实现主机与虚拟机之间的文件共享，同时可支持自由拖拽的功能，鼠标也可在虚拟机与主机之间自由移动（不用再按ctrl+alt），且虚拟机屏幕也可实现全屏化。
VMware Tools是VMware虚拟机中自带的一种增强工具，相当于VirtualBox中的增强功能（Sun VirtualBox Guest Additions），是VMware提供的增强虚拟显卡和硬盘性能、以及同步虚拟机与主机时钟的驱动程序。

![image](A9292B0AAFC545A8BF228E7D427D925A)

注意如果这里是灰色的需要您将linux.iso镜像加载到虚拟光驱中，一般在VM的安装目录下有，如果没有您需要自行下载。

说明：ubuntu的使用不是本文的重点，相关操作请大家自行查找。

### 4.3、生成ngrok服务器与客户端应用程序
**4.3.1. 导出源代码**
ngrok的源代码托管在github上，可以先在ubuntu下安装git再将ngrok的源代码克隆到本地。

![image](F46F4B31CEDC4F9E856A0E3001EF335C)

其实也可以直接下载到本地后解压，这里使用命令行完成。

启动ubuntu，开打命令行（终端），如下所示：

![image](043040F649504CD3833B7C37E6B28A15)

以root身份执行如下命令：

```
mkdir ngrok #创建名称为ngrok的目录

apt-get update #更新包管理器

apt-get install git  #安装git

git clone https://github.com/inconshreveable/ngrok.git ngrok2 #将ngrok源代码克隆回本地
```

成功执行后如下所示：

![image](80A794FED4194D09983E837F856EB4A1)

导出成功后的源代码：

![image](7290BDE28DF946168DF88D47F93DA344)

PS. 直接在服务器上下载的话实在太慢，可以先在本地下载好，然后用ftp放到服务器上去直接用，如果安装了VMware tools直接拖进去就可以了。

### 4.3.2. 安装Go语言开发环境

直接在命令模式下执行如下指令：
```
apt-get install golang #安装go语言
```

执行结果如下：

![image](76E39DDD873F45C6A03421E8681F5EC1)

### 4.3.3. 更改ngrok域名

在自己的域名管理中添加解析A记录，如下所示：

![image](A79F311B536C48EDB3153E1462DC1743)

将*.ngrok与ngrok都指向您的主机IP。

默认的域名是ngrok自己的，要替换成您自己的域名


```
export GOPATH=/usr/local/ngrok/  #设置环境变量，Go语言的安装位置
export NGROK_DOMAIN="ngrok.yourdomain.com"  #设置环境变量，ngrok域名
```

PS. ngrok名称可以任意，推荐名称为ngrok或者tunnel 

### 4.3.4. 为域名生成证书


```
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
openssl x509 -req -in server.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out server.crt -days 5000
```

生成后的结果如下：

![image](A927F6A6069F4EF486F1B7B623FA87C8)

证书如下：

![image](CD9D906ADA7F472788EC35C10E58D13C)

### 4.3.5. 拷贝证书到指定位置


```
cp rootCA.pem assets/client/tls/ngrokroot.crt  #复制rootCA.pem到assets/client/tls/并更名为ngrokroot.crt
cp server.crt assets/server/tls/snakeoil.crt #复制server.crt到assets/server/tls/并更名为snakeoil.crt
cp server.key assets/server/tls/snakeoil.key #复制server.key到assets/server/tls/并更名为snakeoil.key
```

运行结果：

![image](7DB5BD0F5DC7441A8758163E2BEAE9D4)

### 4.3.6. 编译

 由于go语言的特性，在编译时直接生成机器码，所以在运行过程中并不需要go的环境（非托管应用）。在ngrok目录下，运行一下命令分别生成对应的客户端与服务端。
 
 
```
#win服务端
GOOS=windows GOARCH=386 make release-server 
#win客户端
GOOS=windows GOARCH=386 make release-client
#linux服务端
GOOS=linux GOARCH=386 make release-server
#linux客户端
GOOS=linux GOARCH=386 make release-client
```

生成完成后，在工作目录的bin文件夹下，产生对应的文件。以编译windows平台为例，会产生“ngrok.exe”与“ngrokd.exe”这两个文件，前者客户端，后者需要运行在公网服务器上。

因为项目中引用了一些外部资源，生成会耗费一些时间，对网络也有一定的要求，太慢会中短，命令执行下如：

![image](852E47DFAED64958918332181FF8BD32)

生成结果：

![image](9625496141394A5CBE2CB7C7352793DB)

这里我还生成了两个运行在windows服务器与客户端的应用：

![image](0CDDE2E58BE14672958D63057DB36AFC)

ngrok.exe是客户端，ngrokd.exe是服务端，下面是比较连续的操作结果。

![image](0768FC08675D42D1BF3F5ACDC572DB18)

![image](E3C2971EA9C645F791F77F82AD75023F)

# 五、部署服务器端主程序
### 5.1、部署到Windows Server服务器
将生成的ngrokd.exe文件复制到windows服务器中，当然如果要部署到linux中也是没有问题的。

这里我将ngrokd.exe放在c:\grokeServer目录下：

![image](D6AF149E6E96414B99EBBBB0809DBDF6)

为了方便，我编写了一个批处理文件：ngrokserver2.bat


```
ngrokd.exe -tlsKey="snakeoil.key" -tlsCrt="snakeoil.crt" -domain="ngrok.你的域名.com" -httpAddr=":801" -httpsAddr=":802"
```

点击批处理运行结果如下：

![image](E511D596E9BA4C55BD999E63EBE01BF2)

绑定的域名换成自己的域名，http使用801端口，https使用802端口，供客户端连接的管道端口设置为4443端口，必须前面的域名相同。

为了安全许多服务器会将端口屏蔽，我使用的是ECS服务器，默认801，802都是关闭的，需要手动开启，在阿里云的后台添加开放的端口就可了：

![image](0E85CEFC8ABB4AAC9DCC0FBC7D687BB7)

### 5.2、一键部署ngrok服务器（CentOS、Debian、Ubuntu）

如果编译生成ngrok的源代码生成应用太麻烦，你可以选择网友写的工具，支持一键部署到安装平台：CentOS、Debian、Ubuntu。

https://github.com/clangcn/ngrok-one-key-install

### 六、部署ngrok客户端

这里的客户端就是您的web应用程序所运行的主机，将ubuntu生成的ngrok.exe客户端应用复制到您的系统中：

![image](34E8DA7BF67A4AA0B329B39F379A86C9)

添加配置文件ngrok.cfg:


```
server_addr: "ngrok.你的域名.com:4443"
trust_host_root_certs: false
```

 添加批处理start.bat，如果只运行一次直接在命令行下输入命令也是一样的效果，内容如下：
 
 
```
ngrok.exe -subdomain kyt -config=ngrok.cfg 8987
```

其中8987为端口号，运行成功的结果如下所示： 

![image](A4489CA0156C4DF7A64510A39BE3EF59)

看到这个界面时说明已成功了。

# 七、启动客户端并测试

打开浏览器，输入您映射后的域名就可以穿透内网访问您的web服务器了。

![image](4EA074CBDAE0428FA977F246F098EDDB)

# 八、总结
一开始选择错了平台，在windows花了不少时间，在ubuntu下顺利完成。

无论是客户端还是服务器端最好都做成服务，更方便与稳定。

由于服务器上同时运行着IIS，故服务端Ngrok启动时无法使用80端口，所以在上面，我使用了801作为Ngrok服务器的http端口，使用IIS的代理功能可以解决这个问题，点击这里。当然也可以使用nginx将80转换成其它端口。

许多内容都参考了网友的文章。

**如果服务器搭建好了，只运行客户端穿透内网一分钟够了(手动狗头)。**

欢迎您提供更加好的解决方案，欢迎您提供更多的免费代理服务器，我随时更新，谢谢！


