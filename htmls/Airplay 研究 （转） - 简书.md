\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.jianshu.com\](https://www.jianshu.com/p/36ed158eea49)

[![](https://upload.jianshu.io/users/upload_avatars/9516479/f9089e9e-9d4e-4c17-b75a-92d99e5de8db.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/11a418c4372d)

0.1882018.06.13 15:02:37 字数 4,735 阅读 4,322

```
之前有使用DLNA实现投屏功能 ，但或许此功能对用户体验来讲有限，达不到用户的满意度，所以开始研镜像功能，但镜像的实现实际上只有两种解决方案 miracast以及airplay。前者 并不能实时使用，需要引导用户对电视端与手机端进行一定的设置开启后方可使用，与airplay的体验相差甚远，so 打算入坑airplay。查找多方资料，做以处总结,方便后继接着调研，开发。以下内容大多为我转载，都会标注来源处 ~
```

> 多屏互动技术研究（三）之 Airplay 研究（原创）  
> 来源：[https://blog.csdn.net/u011897062/article/details/79446001](https://blog.csdn.net/u011897062/article/details/79446001)

*   [Airplay 技术研究](https://blog.csdn.net/u011897062/article/details/79446001#airplay%E6%8A%80%E6%9C%AF%E7%A0%94%E7%A9%B6)
    *   [1\. Airplay 简介](https://blog.csdn.net/u011897062/article/details/79446001#1-airplay%E7%AE%80%E4%BB%8B)
        *   [1.1 Airplay 协议构成](https://blog.csdn.net/u011897062/article/details/79446001#11-airplay%E5%8D%8F%E8%AE%AE%E6%9E%84%E6%88%90)
    *   [1.2 Airplay 难点分析](https://blog.csdn.net/u011897062/article/details/79446001#12-airplay%E9%9A%BE%E7%82%B9%E5%88%86%E6%9E%90)
        *   [1.2.1 Airplay 协议文档缺失](https://blog.csdn.net/u011897062/article/details/79446001#121-airplay-%E5%8D%8F%E8%AE%AE%E6%96%87%E6%A1%A3%E7%BC%BA%E5%A4%B1)
        *   [1.2.2 Airplay 敏感数据加密的破解](https://blog.csdn.net/u011897062/article/details/79446001#122-airplay-%E6%95%8F%E6%84%9F%E6%95%B0%E6%8D%AE%E5%8A%A0%E5%AF%86%E7%9A%84%E7%A0%B4%E8%A7%A3)
    *   [1.3 Airplay 握手建连](https://blog.csdn.net/u011897062/article/details/79446001#13-airplay-%E6%8F%A1%E6%89%8B%E5%BB%BA%E8%BF%9E)
    *   [1.4 Airplay Screen Mirroring 视频数据传输](https://blog.csdn.net/u011897062/article/details/79446001#14-airplay-screen-mirroring-%E8%A7%86%E9%A2%91%E6%95%B0%E6%8D%AE%E4%BC%A0%E8%BE%93)
*   [2\. 实现机制](https://blog.csdn.net/u011897062/article/details/79446001#2-%E5%AE%9E%E7%8E%B0%E6%9C%BA%E5%88%B6)
    *   [2.1 设备发现](https://blog.csdn.net/u011897062/article/details/79446001#21-%E8%AE%BE%E5%A4%87%E5%8F%91%E7%8E%B0)
    *   [2.2 设备连接及信令处理](https://blog.csdn.net/u011897062/article/details/79446001#22-%E8%AE%BE%E5%A4%87%E8%BF%9E%E6%8E%A5%E5%8F%8A%E4%BF%A1%E4%BB%A4%E5%A4%84%E7%90%86)
        *   [2.2.1 Mirror 基本流程](https://blog.csdn.net/u011897062/article/details/79446001#221-mirror%E5%9F%BA%E6%9C%AC%E6%B5%81%E7%A8%8B)
        *   [2.2.2 Mirror 交互流程](https://blog.csdn.net/u011897062/article/details/79446001#222-mirror%E4%BA%A4%E4%BA%92%E6%B5%81%E7%A8%8B)
*   [3\. 关键代码实现](https://blog.csdn.net/u011897062/article/details/79446001#3-%E5%85%B3%E9%94%AE%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0)
    *   [3.1 服务发现](https://blog.csdn.net/u011897062/article/details/79446001#31-%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0)
    *   [3.2 RTSP 信令解析](https://blog.csdn.net/u011897062/article/details/79446001#32-rtsp%E4%BF%A1%E4%BB%A4%E8%A7%A3%E6%9E%90)
*   [4\. 总结与展望](https://blog.csdn.net/u011897062/article/details/79446001#4-%E6%80%BB%E7%BB%93%E4%B8%8E%E5%B1%95%E6%9C%9B)
*   [5\. 参考文档](https://blog.csdn.net/u011897062/article/details/79446001#5-%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3)

1\. Airplay 简介
--------------

AirPlay 是苹果开发的一种无线技术，可以通过 WiFi 将 iPhone 、iPad、iPod touch 等 iOS 设备上的包括图片、音频、视频及镜像通过无线的方式传输到支持 AirPlay 设备（IOS8 后，AirPlay 可使用 P2P 直连，绕过了 WIFI，具体有待深入）。  
AirPlay 支持如下几种使用场景:  
• 从 iOS 设备上传输并显示照片、幻灯片；  
• 从 iOS 设备或者 Itunes 软件中传输并播放音频；  
• 从 iOS 设备或者 Itunes 软件中传输并播放视频；  
• 对 iOS 设备或者 OS X Mountain Lion 进行屏幕镜像。由于此功能需要硬件的硬解码支持，所以只能在 iPad 2、iPhone 4S、带 Sandy Bridge CPU 的 Mac 电脑（或更新的设备）上支持。  
最初这套协议名字叫 AirTunes，只支持音频流播放。 后来苹果开发 Apple TV 时，对此协议进行了扩充和改进，加入了视频支持，并改名叫做 AIRPLAY。

### 1.1 Airplay 协议构成

AirPlay 并不是完全重新开始写的一个协议, 是好几个现有的协议的组合, 其中有的协议是完全标准的, 有一部分协议进行了一些修改, 有的则是完全私有的。  
• Multicast DNS (aka Bonjour) 用于发布服务, 启动后, 在 iOS 的控制中心菜单中就能看到对应的设备；  
• HTTP / RTSP / RTP 用于流媒体服务, 传输音视频数据, 进行播放控制等；  
• NTP 时间同步；  
• FairPlay DRM 加密 完全私有的加密协议。

1.2 Airplay 难点分析
----------------

### 1.2.1 Airplay 协议文档缺失

AirPlay 是 Apple 的私有协议族, 没有公开的官方文档作为参考. 目前能够参考的是 github 上 nto 大神整理的一份非官方的协议 spec 文档, 但是完全照着文档的描述是无法兼容新的 iOS 系统 (iOS9/10) 的, 新系统对协议有改变, 而这份文档的最后更新日期是 2012 年, 这些变更内容都没有包含。

### 1.2.2 Airplay 敏感数据加密的破解

有人会说 Airplay 毕竟是通过网络交互的协议, 当年 nto 能够整理出一份 spec, 现在也可以通过抓包, 分析的方式得到新的交互过程, 缺少了文档不过是增加一些破解协议的难度而已. 但是以保护用户隐私著称的 Apple, 不会明文传输音视频数据的. Airplay 协议中有着比较完善的 DRM 保护, 传输的音视频都是用 AES 算法加密过的, 而 AES 的 key 则是在握手的过程中通过 Fairplay 协议保护的, 这一部分是 Apple 重点保护的内容, 目前没有人能真正破解。

1.3 Airplay 握手建连
----------------

建连握手部分是经过修改的 RTSP 协议, 与 HTTP 协议比较类似, 都是客户端 (iOS 系统) 发起请求, 接收端针对请求内容进行回应的方式. 每个请求包括: 方法 + 路径 + header+content 等内容。

主要的交互过程如下:  
4 次 POST 请求, 分别对应的路径为 /pair-setup, /pair-verify 和两次 / fp-setup 这是开头于 FairPlay 相关的核心加密部分, 任何一次 POST 的回复内容不对都会导致握手失败。SETUP 请求 不同版本的 iOS 系统中, SETUP 方法的次数可能不同。

在 SETUP 方法中能得到比较多的信息, 比如后面视频 AES 解密需要用到的 ekey 和 eiv. 接收端需要将自己的监听的接收端口 airVideoPort 回复给客户端. 这些信息需要对这次请求的 content 使用 Apple 的 Binary plist 格式进行解码才能提取, 回复内容也要编码为 Binary plist 格式. 后续完成握手后, 客户端会通过 airVideoPort 端口建立 TCP 连接, 然后将屏幕画面通过 H.264 编码后再通过 AES 加密处理后, 向连接持续发送。

一次 GET 请求, 路径为 / info, 这里接收端需要回复一个 Binary plist 格式的数据, 将用户配置的画面分辨率, 最大帧率等信息传递到客户端。

可能有多次 GET\_PARAMETER 和 SET\_PARAMETER 请求用于调整音量大小。

一次 RECORD 请求, 表明握手完成, airplay 镜像开始。

在镜像过程中会每秒都收到 POST 请求, 路径为 / feedback, 相当于心跳, 用于保持 TCP 的长连接。

1.4 Airplay Screen Mirroring 视频数据传输
-----------------------------------

视频数据传输是客户端通过 TCP 单方向的往接收端灌加密后的数据. 数据内容分为头部和载荷. 头部包含了载荷的类型, 长度, 时间戳等信息. 载荷部分有两种, 一种是 H.264 的参数集, sps, pps 等; 另一种就是 AES 加密后的 H.264 裸流, 没有填充到封装格式中, 用 SETUP 请求中得到的 key 和 iv 解密后, 即可送到视频解码器中解码。

实现 Airplay 主要分为以下几个步骤：  
1\. 设备发现，也就是实现音视频服务发现功能；  
2\. 设备连接及信令处理；  
3\. 音视频数据包解密及解码处理。

2.1 设备发现
--------

实现 AIRPLAY 协议的软件不需要再做任何配置就能发现同一网络中的相关设备，这主要得益于 Bonjour（基于 M-DNS 协议实现）。Bonjour：苹果为基于组播域名服务 (multicast DNS) 的开放性 Zeroconf 标准所起的名字。Zeroconf (零设置网络标准)：全称为 Zero configuration networking，中文名则为零配置网络服务标准，是一种用于自动生成可用 IP 地址的网络技术，不需要额外的手动配置和专属的配置服务器。具体例子为：用户拥有一台 apple tv 和一台 iPhone5s，那之只要都连入到同一个无线局域网内，iphone4s 就会自动找出 apple tv，那么在播放音乐或者视频时候，用户只要点击推送，就可以讲音乐和视频推送到 apple tv 上播放。  
关于 mDNS 部分，苹果已经开源了一个工程：mDNSResponder，可以进行参考，另外 JAVA 下也有一个开源实现 jmdns，而在 Android 平台中，自 Android4.1 开始官方实现了一套 Nsd API(网络服务发现)，也实现了此功能。  
服务发布后的查询响应包如下：

![](http://upload-images.jianshu.io/upload_images/9516479-5484b76a4eb6120c)

服务发布后的查询响应包

图 1 服务发布后的查询响应包  
在这个查询包中有下面几个关键字段，Name、Service、Port，AirPlay 服务名称格式如下：AS-XX.\_airplay.\_tcp.local, RAOP 服务名称格式如下：000B828B571E@AS-XX.\_raop.\_tcp.local. Service 字段也就是. 号分割的第一项，第二项说明了服务的类别：\_airplay 是视频服务，\_raop 是音频服务，第三项说明数据传输的协议，可以通过 tcp 或者 udp 传输。Port 声明了 RTSP 及 HTTP 信令交互的端口，也就是接收端以这两个端口分别创建 ServerSocket，Client 可以通过这两个端口与服务端建立起来连接。所以在发布 airplay 及 raop 服务之前必须准备好这两个 ServerSocket, 当 iphone/ipad 发现服务并发起连接时，信令便通过这两个端口来与接收端进行交互。在镜像流程中，信令会随机选择一个通道进行信令交互。  
下面是 mDNS 中 txt 相关字段的抓包及各个字段的含义，在最新的 IOS 系统中增加了比较多的字段，有些字段的含义暂时还不明确，但是这个并不影响服务被发现，只要声明了比较关键的字段服务就会被发现。  
raop 中比较关键的字段有 ch, cn,et, sr,ss, tp 等，各字段的具体含义及取值在下面的表格中已写明。  
airplay 中比较关键的字段有 devicesid,features 等，各个字段具体含义及取值在下面的表格中。

![](http://upload-images.jianshu.io/upload_images/9516479-61afcacff5b38ba4)

mDNS 查询包

图 2 mDNS 查询包

The name is formed using the MAC address of the device and the name of the remote speaker which will be shown by the clients.  
The following fields appear in the TXT record:

<table><thead><tr><th>name</th><th>value</th><th>description</th></tr></thead><tbody><tr><td>txtvers</td><td>1</td><td>TXT record version 1</td></tr><tr><td>ch</td><td>2</td><td>audio channels: stereo</td></tr><tr><td>cn</td><td>0,1,2,3</td><td>audio codecs</td></tr><tr><td>et</td><td>0,3,5</td><td>supported encryption types</td></tr><tr><td>md</td><td>0,1,2</td><td>supported metadata types</td></tr><tr><td>pw</td><td>false</td><td>does the speaker require a password?</td></tr><tr><td>sr</td><td>44100</td><td>audio sample rate: 44100 Hz</td></tr><tr><td>ss</td><td>16</td><td>audio sample size: 16-bit</td></tr><tr><td>tp</td><td>UDP</td><td>supported transport: TCP or UDP</td></tr><tr><td>vs</td><td>220.68</td><td>server version 220.68</td></tr><tr><td>am</td><td>AppleTV2,1</td><td>device model</td></tr><tr><td>fp</td><td>0xA7FFFF7</td><td>supported features</td></tr></tbody></table>

Audio codecs

<table><thead><tr><th>cn</th><th>description</th></tr></thead><tbody><tr><td>0</td><td>PCM</td></tr><tr><td>1</td><td>Apple Lossless (ALAC)</td></tr><tr><td>2</td><td>AAC</td></tr><tr><td>3</td><td>AAC ELD (Enhanced Low Delay)</td></tr></tbody></table>

Encryption Types

<table><thead><tr><th>et</th><th>description</th></tr></thead><tbody><tr><td>0</td><td>no encryption</td></tr><tr><td>1</td><td>RSA (AirPort Express)</td></tr><tr><td>3</td><td>FairPlay</td></tr><tr><td>4</td><td>MFiSAP (3rd-party devices)</td></tr><tr><td>5</td><td>FairPlay SAPv2.5</td></tr></tbody></table>

Metadata Types

<table><thead><tr><th>md</th><th>description</th></tr></thead><tbody><tr><td>0</td><td>text</td></tr><tr><td>1</td><td>artwork</td></tr><tr><td>2</td><td>progress</td></tr></tbody></table>

The following fields are available in the TXT record:

<table><thead><tr><th>name</th><th>value</th><th>description</th></tr></thead><tbody><tr><td>model</td><td>AppleTV2,1</td><td>device model</td></tr><tr><td>deviceid</td><td>58:55:CA:1A:E2:88</td><td>MAC address of the device</td></tr><tr><td>features</td><td>0x39f7</td><td>bitfield of supported features</td></tr><tr><td>pw</td><td>1</td><td>server is password protected</td></tr></tbody></table>

The pw field appears only if the AirPlay server is password protected. Otherwise it is not included in the TXT record.  
The features bitfield allows the following features to be defined:

<table><thead><tr><th>bit</th><th>name</th><th>description</th></tr></thead><tbody><tr><td>0</td><td>Video</td><td>video supported</td></tr><tr><td>1</td><td>Photo</td><td>photo supported</td></tr><tr><td>2</td><td>VideoFairPlay</td><td>video protected with FairPlay DRM</td></tr><tr><td>3</td><td>VideoVolumeControl</td><td>volume control supported for videos</td></tr><tr><td>4</td><td>VideoHTTPLiveStreams</td><td>http live streaming supported</td></tr><tr><td>5</td><td>Slideshow</td><td>slideshow supported</td></tr><tr><td>7</td><td>Screen</td><td>mirroring supported</td></tr><tr><td>8</td><td>ScreenRotate</td><td>screen rotation supported</td></tr><tr><td>9</td><td>Audio</td><td>audio supported</td></tr><tr><td>11</td><td>AudioRedundant</td><td>audio packet redundancy supported</td></tr><tr><td>12</td><td>FPSAPv2pt5_AES_GCM</td><td>FairPlay secure auth supported</td></tr><tr><td>13</td><td>PhotoCaching</td><td>photo preloading supported</td></tr></tbody></table>

2.2 设备连接及信令处理
-------------

当 iPhone 手机端发现接收端以后就会通过 TCP 连接上来，然后会通过 RTSP 协议进行交互，不过经过苹果的数次升级和变更，现在虽然表面上还是基于 RTSP 协议，不过已经和标准的 RTSP 协议区别很大了，如果想学习标准的 RTSP 协议，有个非常好的开源项目 live555 值得参考。接下来重点来了，我们的接收端需要处理来自手机端发来的各种请求命令，并且做出正确的响应。

### 2.2.1 Mirror 基本流程

流程简图如下：  
1.| <————–fp-setup————> |  
| |  
2.| <————–fp-setup————> |  
| |  
| |  
3.| <————–SETUP—————> |  
| |  
4.| <————–GET /info———–> |  
| |  
5.| <————–GET\_PARAMMETER——> |  
| | 6 | <————–SET\_PARAMMETER——> |  
| |  
7 | <————–POST /feedback——> | (every 2 seconds)  
streaming data to port 7100 128bytes headers | H264 DATA | 128bytes headers | H264 DATA | …  
伪代码大致如下：

```
if (!strcmp(method,"POST") && !strcmp(uri,"/pair-setup")) {
res = request\_handle\_pairsetup(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method,"POST") && !strcmp(uri,"/pair-verify")) {
res = request\_handle\_pairverify(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method, "POST") && !strcmp(uri,"/fp-setup")) {
res = request\_handle\_fpsetup(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method, "POST") && !strcmp(uri, "/auth-setup")) {
res = request\_handle\_authsetup(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method, "GET") && !strcmp(uri, "/info")) {
res = request\_handle\_info(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method, "GET") && !strcmp(uri, "/stream.xml")) {
res = request\_handle\_streamxml(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method, "OPTIONS")) {
res = request\_handle\_options(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method, "ANNOUNCE")) {
res = request\_handle\_announce(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method, "SETUP")) {
res = request\_handle\_setup(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method, "SET\_PARAMETER")) {
res = request\_handle\_setparameter(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method, "FLUSH")) {
res = request\_handle\_flush(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method, "TEARDOWN")) {
res = request\_handle\_teardown(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method, "RECORD")) {
res = request\_handle\_record(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method, "GET\_PARAMETER")) {
res = request\_handle\_getparameter(conn, request, res, &responseData, &responseDataLen);
} else if (!strcmp(method,"POST") && !strcmp(uri,"/feedback")) {
res = request\_handle\_feedback(conn, request, res, &responseData, &responseDataLen);
}
```

其中有些命令是为了兼容老版本协议而存在的。

### 2.2.2 Mirror 交互流程

下面以一次完整的镜像流程抓包分析相关的交互流程。  
大致交互流程如下：  
一. POST /pair-setup POST /pair-verify 等命令用于配对验证，其中会用到 ed25519，SHA-512，AES 等算法。

PS. 目前这一步相关的加密流程比较复杂，还有找到破解的方法，但是在发布 Airplay 服务时候可以设置特定的 features 跳过这一步的验证流程。

二. POST /fp-setup 命令是 FairPlay 相关，FairPlay 是苹果公司开发的一种 DRM(数字版权管理) 技术，苹果的视频和音频传输都在这种技术的保护之下被 AES 加密后传输，这个 FairPlay 技术也是整个 AirPlay 中最难的部分，真的很难。

三. 第一个 SETUP 命令，手机端会传输大量的参数给到接收端，其中最重要的两个参数是 ekey 和 eiv，它们经过一些变换之后要用于 AES 解密的。要重点说的是参数是用 plist 格式保存的，与标准的 RTSP 协议完全不同。

四. GET /info 命令 手机端会通过此命令获取接收端的一些参数，比如接收端能接受的音频格式，能播放的视频长和宽，分辨率等。

五. GET\_PARAMETER 和 SET\_PARAMETER 命令用于调整音量大小，比较简单。

六. RECORD 命令好像没什么具体要做的，因为重要的事情都在 SETUP 命令里面做了。

七. 第二个 SETUP 命令，手机端通过这个命令通知接收端，准备建立屏幕镜像传输通道，命令中的 type 参数等于 110，代表是屏幕镜像。接收端会在应答报文中将自己的接收端口告诉手机端，也就是 dataPort 这个参数，默认一般是 7100。手机端会通过 TCP 连接上这个端口，然后将屏幕镜像通过 h264 编码以后再通过 AES 加密处理后通过这个端口源源不断的传给接收端，而具体的承载协议是苹果自定义的。

八. POST /feedback 命令 每秒钟手机端都会向接收端发一次这个命令，我理解为定时保活的意思。

每次推送音频及视频的时候会有一个二次 SETUP 的交互，音频和视频的交互内容稍有不同，主要是端口、数据类型及数据格式等的交互。

Client 相关信令的相关内容主要通过 plist 这种格式，这是苹果专门的一种 xml 形式的内容交换格式，目前 java 端可以通过 dd-plist.jar 完成数据解析。

完整的 Mirror 交互抓包如下：

POST /pair-setup RTSP/1.0  
Content-Length: 32  
Content-Type: application/octet-stream  
CSeq: 0  
DACP-ID: E28CCF9054EDE3B9  
Active-Remote: 3016615115  
User-Agent: AirPlay/320.20

1t<..\*….i1”m,.. g.|h.Y…t..M.RTSP/1.0 200 OK  
Date: Thu, 19 Oct 2017 02:16:35 GMT  
CSeq: 0  
Content-Type: application/octet-stream  
Server: AirTunes/220.68  
Content-Length: 32

8.5…p.\*…..A.%.Yn…..w  
…$.

POST /pair-verify RTSP/1.0  
X-Apple-PD: 1  
X-Apple-AbsoluteTime: 530072180  
Content-Length: 68  
Content-Type: application/octet-stream  
CSeq: 1  
DACP-ID: E28CCF9054EDE3B9  
Active-Remote: 3016615115  
User-Agent: AirPlay/320.20

…………..k……)oWL..t.%s..(.b1t<..\*….i1”m,.. g.|h.Y…t..M.RTSP/1.0 200 OK  
Date: Thu, 19 Oct 2017 02:16:36 GMT  
CSeq: 1  
Content-Type: application/octet-stream  
Server: AirTunes/220.68  
Content-Length: 96

.^{…O. j..Xa+..y….+.^.!…p…nF.!.g\*.W….. .U…L…b\[bz.tf………sm..”…..,A.=….g..

POST /pair-verify RTSP/1.0  
X-Apple-PD: 1  
X-Apple-AbsoluteTime: 530072180  
Content-Length: 68  
Content-Type: application/octet-stream  
CSeq: 2  
DACP-ID: E28CCF9054EDE3B9  
Active-Remote: 3016615115  
User-Agent: AirPlay/320.20

……H.u6.Fq\`.(V?..+6.4.7…^..&…..K.L………….T1Wv7I.\*ke.m..RTSP/1.0 200 OK  
Date: Thu, 19 Oct 2017 02:16:36 GMT  
CSeq: 2  
Content-Type: application/octet-stream  
Server: AirTunes/220.68  
Content-Length: 0

POST /fp-setup RTSP/1.0  
X-Apple-ET: 32  
Content-Length: 16  
Content-Type: application/octet-stream  
CSeq: 3  
DACP-ID: E28CCF9054EDE3B9  
Active-Remote: 3016615115  
User-Agent: AirPlay/320.20

FPLY…………RTSP/1.0 200 OK  
Date: Thu, 19 Oct 2017 02:16:36 GMT  
CSeq: 3  
Content-Type: application/octet-stream  
X-Apple-ET: 32  
Server: AirTunes/220.68  
Content-Length: 142

FPLY………..2.W..RO…z.d.{.D$…~.  
.z..\]..’0.Y….:.M…….M…{V…….C….e.N.9.\[..d..\]..>.j.~.V.+..@Bu.ZD.Y.rV…Q8…’r..W.P.\*.Fh.

POST /fp-setup RTSP/1.0  
X-Apple-ET: 32  
Content-Length: 164  
Content-Type: application/octet-stream  
CSeq: 4  
DACP-ID: E28CCF9054EDE3B9  
Active-Remote: 3016615115  
User-Agent: AirPlay/320.20

FPLY………….p..gO.;……%jXB…..\*…..T….x….#.E.^.\]|…..s….  
j……….q..pT….y.Gq<..r.  
.+..ZW..\_..Z……H…>…k’..k..4…M;r..XU.vc…z,…..O.RTSP/1.0 200 OK  
Date: Thu, 19 Oct 2017 02:16:36 GMT  
CSeq: 4  
Content-Type: application/octet-stream  
X-Apple-ET: 32  
Server: AirTunes/220.68  
Content-Length: 32

FPLY……….XU.vc…z,…..O.

SETUP rtsp://192.168.123.47/6108990559123033009 RTSP/1.0  
Content-Length: 425  
Content-Type: application/x-apple-binary-plist  
CSeq: 5  
DACP-ID: E28CCF9054EDE3B9  
Active-Remote: 3016615115  
User-Agent: AirPlay/320.20

bplist00……….  
..  
………RetTname\]sourceVersionZtimingPortXdeviceIDUmodelZmacAddress^osBuildVersion\[sessionUUIDTekeySeiv. _..GrandStream-6plusV320.20..&_..[D8:BB:2C:1F:28:94YiPhone7,1\_..D8:BB:2C:1F:28:92U14G60\_.$54C77E8B-F4BE-4BB1-A9D7-D246D6672D8BO.HFPLY](d8:BB:2C:1F:28:94YiPhone7,1_..D8:BB:2C:1F:28:92U14G60_.%2454C77E8B-F4BE-4BB1-A9D7-D246D6672D8BO.HFPLY)…….<….?z.(…K\`…..}…..’……W…B…6j..w.{^..d…,…!.O..Ds\*..i#6…D.T.!…..”.’.5.@.I.O.Z.i.u.z.~……………….H………………………….\[

二进制的 plist 转换成 xml 格式如下：

```
<?xml version="1.0" encoding="UTF-8"?>
                                                <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
                                                <plist version="1.0">
                                                <dict>
                                                   <key>et</key>
                                                   <integer>32</integer>
                                                   <key>name</key>
                                                   <string>GrandStream-6plus</string>
                                                   <key>sourceVersion</key>
                                                   <string>320.20</string>
                                                   <key>timingPort</key>
                                                   <integer>51149</integer>
                                                   <key>deviceID</key>
                                                   <string>D8:BB:2C:1F:28:94</string>
                                                   <key>model</key>
                                                   <string>iPhone7,1</string>
                                                   <key>macAddress</key>
                                                   <string>D8:BB:2C:1F:28:92</string>
                                                   <key>osBuildVersion</key>
                                                   <string>14G60</string>
                                                   <key>sessionUUID</key>
                                                   <string>0DBDA1BF-5083-46E7-9F25-37143854A861</string>
                                                   <key>ekey</key>
                                                   <data>
                                                      RlBMWQECAQAAAAA8AAAAAF5xvfwQNcVLHM2KurVtQCMAAAAQC8/GGZ7i56Hu5wMkHYQ0Bh/SImp90UX8SzQltjNHayrNtwmX
                                                   </data>
                                                   <key>eiv</key>
                                                   <data>
                                                      SdqQ7TXGEvz+R80Xi4IoaA==
                                                   </data>
                                                </dict>
                                                </plist>
```

关键字段已通过红色字体标出，et，加密类型；timingPort， NTP 同步端口， ekey/eiv 主要用 AES 解密，用于后期视频帧的解密处理。

RTSP/1.0 200 OK  
Date: Thu, 19 Oct 2017 02:16:37 GMT  
CSeq: 5  
Server: AirTunes/220.68  
Content-Length: 284

目前能跑通的流程为发现流程、连接流程，相关代码实现如下，音视频数据解密及解码流程还没有仔细去研究。

3.1 服务发现
--------

raop 服务发布，相关 txt 设置如下：

```
protected ServiceInfo onCreateServiceInfo(byte\[\] hwAddr, String name, int port) {
                String identifier = getStringHardwareAdress(hwAddr);
                Map<String, String> txt = new HashMap<String, String>();
                txt.put("tp", "UDP"); /\*传输方式\*/
                txt.put("sm", "false");
                txt.put("sv", "false");/\*不清楚\*/
                txt.put("ek", "0");
                txt.put("et", "0,1"); /\*supported encryption types, 0, no encryption; 1,RSA (AirPort Express);3, FairPlay;4,MFiSAP (3rd-party devices); 5,FairPlay SAPv2.5\*/
                txt.put("md", "0,1,2"); /\*metadata: text, artwork, progress\*/
                txt.put("cn", "0,1");/\*audio codecs 0   PCM，1   Apple Lossless (ALAC)， 2        AAC， 3  AAC ELD (Enhanced Low Delay) \*/
                txt.put("sr", "44100");/\*audio sample rate: 44100 Hz\*/
                txt.put("ch", "2"); /\*audio channels: stereo\*/
                txt.put("ss", "16"); /\*audio sample size: 16-bit\*/
                txt.put("pw", "false");         /\*does the speaker require a password?\*/
                txt.put("vn", "3");  
                txt.put("txtvers", "1");  /\*TXT record version 1\*/
                txt.put("vs", AirPlay.SRCVERS);  /\*server version 130.14\*/
                txt.put("am",AirPlay.MODEL);  /\*device model\*/
                //txt.put("pk", AirTunes.PKSTR);  还不知道这个字段的用处
                //txt.put("da", "true");还不知道怎么用
                //txt.put("ft", AirPlay.FEATURES);  support function
                txt.put("w", "1");  //unknown
                //txt.put("vn", "65537");  //unknown
                //txt.put("sf", "0x4");  //unknown

                return ServiceInfo
        .create(AirTunes.AIR\_TUNES\_SERVICE\_TYPE,
                        identifier + "@" + name ,
                port, 0, 0, txt);
        }
```

airplay 服务发布：

```
protected ServiceInfo onCreateServiceInfo(byte\[\] hwAddr, String name, int port) {
                String addr = getStringHardwareAdress(hwAddr);
                Map<String, String> txt = new HashMap<String, String>();
                txt.put("deviceid", addr);
                txt.put("features", String.format("0x%04x", AirPlay.FEATURES));
                txt.put("model", AirPlay.MODEL);
                txt.put("srcvers", AirPlay.SRCVERS);
                txt.put("vv", "1");
                //txt.put("rhd", "1.9.7");
                txt.put("flags", "0x44");
                txt.put("pi", AirPlay.PISTR);
                txt.put("pk", AirPlay.PKSTR);
                return ServiceInfo.create(AirPlay.TYPE, name, port, 0, 0, txt);
        }
```

3.2 RTSP 信令解析
-------------

信令相关的代码太多，只截取 SETUP 命令的处理

```
if(contentType.equals("application/x-apple-binary-plist")){
    try {
        NSDictionary dict = (NSDictionary) BinaryPropertyListParser.parse(packet.getContent());
        Log.d(TAG,"\[SETUP\], dict  is "+dict.toXMLPropertyList());
        //process ekey && eiv
        NSData ekeyData = (NSData)dict.get("ekey");
        NSData eivData = (NSData)dict.get("eiv");
        if(ekeyData != null && eivData != null){
        response.append("Audio-Jack-Status", "connected; type=analog");
        String ekey =  (ekeyData).getBase64EncodedData();
        String eiv = (eivData).getBase64EncodedData();
        mAudioSession = AudioSession.parse(ekey, eiv, mKey);
        NSNumber timingportObj = (NSNumber)dict.get("timingPort");
        Log.d(TAG,"\[SETUP\], timingportObj  is "+timingportObj.toString());
        int  timingPort =timingportObj.intValue();
        // Address
        Pattern p = Pattern.compile("SETUP\\\\s(rtsp://.\*?)\\\\s");
        Matcher m = p.matcher(packet.getRawPacket());
        if ( m.find() && mAudioSession != null) {
            String uri = m.group(1);
            Log.d(TAG, "\[SETUP\] uri is "+uri+", timingPort is "+timingPort);
            InetAddress iaddr = InetAddress.getByName(Uri.parse(uri).getHost());
            // Creaet Audio Server
            mAudioServer = AudioServer.create(mAudioSession, iaddr, 0, timingPort);
            }
         if(mAudioServer != null ){
             NSDictionary reponsedict = new NSDictionary();
             Log.d(TAG,"local event Port is "+mAudioServer.getServerPort()+
             ",  time port is "+mAudioServer.getTimingPort());
             reponsedict.put("eventPort",  AirPlay.PORT);
             reponsedict.put("timingPort", mAudioServer.getTimingPort());
             Log.d(TAG,"\[SETUP\]response content is"+reponsedict.toXMLPropertyList());
             byte\[\] contents = BinaryPropertyListWriter.writeToArray(reponsedict);
             response.append("Content-Length", contents.length+"");
             response.setContent(contents, 0, contents.length);
          }
     }else {
           //second SETUP
            NSDictionary reponsedict = new NSDictionary();
            NSDictionary mirrorDic = new NSDictionary();
            mirrorDic.put("dataPort",  7100);
            mirrorDic.put("type", 110);
            reponsedict.put("streams", NSArray.wrap(new NSObject\[\]{mirrorDic}));
            Log.d(TAG,"\[SETUP\]response content is "+reponsedict.toXMLPropertyList());
            byte\[\] contents = BinaryPropertyListWriter.writeToArray(reponsedict);
            //byte\[\] contents = reponsedict.toXMLPropertyList().getBytes();
            response.append("Content-Length", contents.length+"");
            response.finalizeHeader();
            response.setContent(contents, 0, contents.length);
          }

   } catch (IOException e) {
                        // TODO Auto-generated catch block
                           e.printStackTrace();
                         } catch (PropertyListFormatException e) {
                             // TODO Auto-generated catch block
                             e.printStackTrace();
                         } catch (Exception e){
                           e.printStackTrace();
       }
}
```

目前实现了设备发现、设备连接功能。但是相关音频及视频解码流程还没有跑通，涉及的内容比较多，能力及精力有限，还没能进一步去研究相关实现及解密解码流程。在前面的步骤中虽然实现了发现和连接，但是有很多字段的含义及用途不清楚，网上能找到的一份协议并非官方并且是比较老的版本，12 年以前的，很多流程都已经变更了，参考意义有限。这些字段对后期音视频解码是否影响暂不清楚。后期如果需要进步研究的话，首先需要理清楚前面协议的字段与后面音视频流参数之间的关系，其次，需要搞清楚后期发送的数据包相关头部字段的信息及数据加密类型。最后，最关键的是能够找到相关的解密方法及解码器。如果这些问题解决掉了，最终能够实现镜像功能。

\[1\] Unofficial AirPlay Protocol Specification——[http://blog.csdn.net/cyshuxin/article/details/39694111](http://blog.csdn.net/cyshuxin/article/details/39694111)  
\[2\] AirPlay 协议浅析——[http://phonegap.me/post/42.html](http://phonegap.me/post/42.html)  
\[3\] 关于 airplay 协议实现镜像功能研究——[http://blog.csdn.net/bxjie/article/details/39581565](http://blog.csdn.net/bxjie/article/details/39581565)  
\[4\] ios 9.3 POST /pair-setup,POST /pair-verify ——[https://github.com/juhovh/shairplay/issues/61](https://github.com/juhovh/shairplay/issues/61)

* * *

* * *

* * *

> 尝试过的 github 代码参照于：[https://github.com/jamesdlow/open-airplay](https://github.com/jamesdlow/open-airplay)

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![](https://upload.jianshu.io/users/upload_avatars/9516479/f9089e9e-9d4e-4c17-b75a-92d99e5de8db.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)](https://www.jianshu.com/u/11a418c4372d)

总资产 3 (约 0.31 元) 共写了 3.1W 字获得 39 个赞共 16 个粉丝

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

*   手游直播是直播行业中非常重要的一个垂直领域. 手游直播与其他移动直播相比主要是画面的来源不同, 手游直播其实是一种...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4469440/12404b19-12f8-4fa8-a396-b682630fd4b1.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)金山视频云](https://www.jianshu.com/u/b2227c3472fd)阅读 14
    
    [![](https://upload-images.jianshu.io/upload_images/4469440-44be66aadb277f41.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/a100901ff544)
*   RFC 2326RTSP Spec 中文版 (1-11)RTSP Spec 中文版 (12-16)RTSP Spec 中文版...
    
*   Spring Cloud 为开发人员提供了快速构建分布式系统中一些常见模式的工具（例如配置管理，服务发现，断路器，智...
    
    [![](https://upload-images.jianshu.io/upload_images/7328262-54f7992145380c10.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/46fd0faecac1)
*   一、HTTP（WebService） 基于 HTTP 的渐进下载 Progressive Download 流媒体播放仅是...
    
    [![](https://upload-images.jianshu.io/upload_images/121149-5d5388a2722ec028.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/5b0fa403b3ce)
*   RTSP SDP RTP/RTCP 介绍应用层 RTSP、SDP； 传输层 RTP、TCP、UDP； 网络层 IP...
    
    [![](https://upload-images.jianshu.io/upload_images/9196774-3dd91df0807ec20f.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/346d2fbaed7b)