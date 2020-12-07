只是一点草稿，记录一下自己捣鼓 Push Service 时候遇到的问题。

## iOS Platform

iOS 平台上，需要根据阿里移动推送的[官方文档](https://help.aliyun.com/document_detail/30072.html?spm=a2c4g.11186623.6.585.1c8742ffXLuQbK)来进行一些配置。

### Apple 相关

主要是配置证书，参考[这里](https://help.aliyun.com/document_detail/181292.html)，就不多赘述，只补充一些细节。

1. 从 Keychain Access 里导出证书的时候注意一次导出一个，不要搞错了。一定要先在左下角选择 Certificate（证书），不然没有导出 .p12 的选项。
2. 配置的证书所添加到的 App 的 `Bundle Identification` 必须和 Xcode 里用来测试的 Project 里的配置一致，否则会报错。Pusher 只会报 Invalid Token，但是阿里的平台在测试证书的时候会正确地报 bundle id 不匹配的错误
3. Xcode 必须设置 Push Notifications 的 Capability，在 Xcode 里找到 Target 的 `Signing & Capabilities`，在里面检查（和检查 `Bundle Identifier` 在同一个页面）
4. [集成](https://help.aliyun.com/document_detail/30072.html)的时候直接选择 EMAS 统一接入，把 `AliyunEmasServices-Info.plist` 拖进 Xcode 的 App Target 下。如果还有问题的话，参照 Pod 集成方式再额外配置一下
5. 从 APNS 服务器获取 deviceToken 要求连接到的服务器不能过代理([出处](https://developer.apple.com/forums/thread/660180))。据说可以在规则里排出 `17.0.0.0/8` （但我没搞定，我最后用了实机）
6. 在无法直连 APNS 服务器时，`didFailToRegisterForRemoteNotificationsWithError` 和 `didFailToRegisterForRemoteNotificationsWithError` 回调函数都不会被调用，这是 expected 的现象，参见[这里](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html)，在 Obtaining a Device Token in iOS and tvOS 一节中倒数第二段
  - 顺便，我中间尝试了抓包。抓取 Simulator 好像有点麻烦（我没找到 Simulator 对应的 UDID，也没找到可能使用的虚拟网卡），可以参考[这里](https://blog.csdn.net/james_1010/article/details/42397927)和[这里](https://developer.apple.com/documentation/network/recording_a_packet_trace）

### 阿里相关

这里建议直接使用 [Rammus](https://github.com/OpenFlutter/rammus)，暂时跑通了 demo。
