# Facebook ios 事件上报测试踩坑

## 重点提要

- fbsdk至少要12.2.1
- 测试过程中，测试事件页面不能关闭
- 测试设备要安装fb app，允许跟踪（idfa权限），并且登录跟后台一样的fb账号
- xcode工程配置要开启FacebookAutoLogAppEventsEnabled

## fb后台

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/0500b8a827be746c7e23db21b6345abe-117151.jpg)

- 事件上报接入请查看[官方文档](https://developers.facebook.com/docs/app-events/getting-started-app-events-ios/)
- 打开测试事件页面，测试期间不能关闭，上报成功会自动刷新页面
- 后台登录的fb账号要跟测试设备上登录fb账号一致



## 接入问题汇总

- Facebook SDK的版本至少要到12.2.1，[原因请看](https://github.com/facebook/facebook-ios-sdk/issues/1937)
- 接入文档中的配置不能漏，除了fb id之外，autoLog也要开启，如下所示

```xml
<key>FacebookAutoLogAppEventsEnabled</key>
	<true/>
```

- 如果想看fb事件上报的客户端日志，在应用初始化的时候调用

```objective-c
[FBSDKSettings.sharedSettings enableLoggingBehavior:FBSDKLoggingBehaviorAppEvents];
```

- 接入事件的时候，尽量使用fb定义好的标准事件，标准事件的上报能帮助广告优化，[标准事件](https://developers.facebook.com/docs/app-events/reference)
- 自定义事件名称只允许英文字母、数字、中划线、下划线
- fb初始化的时候呀开启广告跟踪

```objective-c
[[FBSettings sharedSettings] setAdvertiserIDCollectionEnabled:YES];
[[FBSettings sharedSettings] setAdvertiserTrackingEnabled:YES];
```





## 测试问题汇总

- 测试设备需要安装fb应用，并登陆后台登录中的fb账号
- fb要开启广告追踪（首次安装一般打开app就会询问，直接同意即可），手机设置->隐私->跟踪->fb，如果没发现fb，可能是fb版本比较久，去appstore更新即可
- 测试步骤
  - 打开fb应用，允许跟踪，并登陆fb账号
  - 启动你要测试的app（如果是已启动，请先强退再启动），进行对应的时间上报测试
  - 等待后台刷新页面刷新，一般几秒钟就能刷新了

