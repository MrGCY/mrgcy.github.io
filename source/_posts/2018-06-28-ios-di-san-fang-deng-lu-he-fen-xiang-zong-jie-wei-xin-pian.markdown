---
layout: post
title: "iOS-第三方登录和分享总结(微信篇)"
date: 2018-06-28 14:44:47 +0800
comments: true
categories: 1
---
对于第三方登录和分享,当我们做熟了就会发现三种登录和分享的方式都是大同小异,流程基本上也一样,只要我们掌握其中的一种,其他的只需要看看文档就会很快做完,下面我们就先介绍微信.
###一 微信
####1.1微信登录
具体iOS微信集成指南[点击查看iOS指南](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=1417694084&token=221ba501dec7869e36e52e0a34b1b6fde58fe54c&lang=zh_CN)
######1.1.1 申请账号
向微信的开放平台申请开发账号[点击打开连接申请](https://open.weixin.qq.com/cgi-bin/index?t=home/index&lang=zh_CN&token=221ba501dec7869e36e52e0a34b1b6fde58fe54c)
######1.1.2下载微信SDK
SDK文件包括 libWeChatSDK.a，WXApi.h，WXApiObject.h 三个。
如选用手动集成，请前往[资源下载页](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419319164&lang=zh_CN "iOS资源下载")下载最新SDK包
![sdk文件预览.png](http://upload-images.jianshu.io/upload_images/2517741-add1a5e8c3df536e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

######1.1.3环境搭建
1.新建工程,将sdk全部拖进去.
![工程导入](http://upload-images.jianshu.io/upload_images/2517741-84ac3409bfeaec9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.微信开放平台新增了微信模块用户统计功能，便于开发者统计微信功能模块的用户使用和活跃情况。开发者需要在工程中链接上:SystemConfiguration.framework, libz.dylib, libsqlite3.0.dylib, libc++.dylib, Security.framework, CoreTelephony.framework, CFNetwork.framework。
3.在你的工程文件中选择Build Setting，在"Other Linker Flags"中加入"-Objc -all_load"，在Search Paths中添加 libWeChatSDK.a ，WXApi.h，WXApiObject.h
![QQ20180110-143356.png](http://upload-images.jianshu.io/upload_images/2517741-ca4798793fd79faa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
4.在Xcode中，选择你的工程设置项，选中“TARGETS”一栏，在“info”标签栏的“URL type“添加“URL scheme”为你所注册的应用程序id
![QQ20180110-143650.png](http://upload-images.jianshu.io/upload_images/2517741-74262aa89bea9874.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
5.在Xcode中，选择你的工程设置项，选中“TARGETS”一栏，在“info”标签栏的“LSApplicationQueriesSchemes“添加微信的白名单,主要是为了手机端进行登录可以直接进入微信客户端中
![QQ20180110-143954.png](http://upload-images.jianshu.io/upload_images/2517741-7b665d76c787c4f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这块我把所有需要添加的白名单都添加了,基本上是比较全的,项目中有需要用的的可以直接复制进去
######1.1.3代码实现
这里的代码可能跟微信官方下载下来的demo中的代码位置有区别,是因为我这边做了一些小的封装,但调用的方法没有变(我这边只把实现的流程展示出来,具体代码可以参考我的demo或者官方demo)
1.要使你的程序启动后微信终端能响应你的程序，必须在代码中向微信终端注册你的id,在AppDelegate的- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions中注册微信
需要导头文件
```
#import "WXApi.h"
#import "WXApiObject.h"
```
```
//向微信注册
-(BOOL)WXRegister{
return [WXApi registerApp:KWXAPPID];
}
```
2.重写AppDelegate的handleOpenURL和openURL方法(后面写的QQ,和微博一样的处理)：
```
- (BOOL)applicationOpenURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation{
if ([KTencentSchema isEqualToString:[url scheme]]) {
//QQ
return [TencentOAuth HandleOpenURL:url];

}else if ([KTencentURLSchema isEqualToString:[url scheme]]) {
//QQ
return [QQApiInterface handleOpenURL:url delegate:self.qqUtils];

}else if ([KWXURLSchema isEqualToString:[url scheme]]) {
//微信
return [WXApi handleOpenURL:url delegate:self.wxUtils];

}else if ([KWBURLSchema isEqualToString:[url scheme]]){
//微博
return [WeiboSDK handleOpenURL:url delegate:self.wbUtils];
}
return NO;
}
- (BOOL)applicationHandleOpenURL:(NSURL *)url {

if ([KTencentSchema isEqualToString:[url scheme]]) {
//QQ
return [TencentOAuth HandleOpenURL:url];

}else if ([KTencentURLSchema isEqualToString:[url scheme]]) {
//QQ
return [QQApiInterface handleOpenURL:url delegate:self.qqUtils];

}else if ([KWXURLSchema isEqualToString:[url scheme]]) {
//微信
return [WXApi handleOpenURL:url delegate:self.wxUtils];

}else if ([KWBURLSchema isEqualToString:[url scheme]]){
//微博
return [WeiboSDK handleOpenURL:url delegate:self.wbUtils];
}
return NO;
}
```
3.你的程序要实现和微信终端交互的具体请求与回应，因此需要实现WXApiDelegate协议的两个方法：
```
#pragma mark- WXApiDelegate
/*! @brief 收到一个来自微信的请求，第三方应用程序处理完后调用sendResp向微信发送结果
*
* 收到一个来自微信的请求，异步处理完成后必须调用sendResp发送处理结果给微信。
* 可能收到的请求有GetMessageFromWXReq、ShowMessageFromWXReq等。
* @param req 具体请求内容，是自动释放的
*/
-(void) onReq:(BaseReq*)req{
if ([req isKindOfClass:[GetMessageFromWXReq class]]) {
//微信终端向第三方程序请求提供内容的消息结构体
if (self.wxDelegate
&& [self.wxDelegate respondsToSelector:@selector(WXApiUtilsDidRecvGetMessageReq:)]) {
GetMessageFromWXReq *getMessageReq = (GetMessageFromWXReq *)req;
[self.wxDelegate WXApiUtilsDidRecvGetMessageReq:getMessageReq];
}
} else if ([req isKindOfClass:[ShowMessageFromWXReq class]]) {
//微信通知第三方程序，要求第三方程序显示的消息结构体
if (self.wxDelegate
&& [self.wxDelegate respondsToSelector:@selector(WXApiUtilsDidRecvShowMessageReq:)]) {
ShowMessageFromWXReq *showMessageReq = (ShowMessageFromWXReq *)req;
[self.wxDelegate WXApiUtilsDidRecvShowMessageReq:showMessageReq];
}
} else if ([req isKindOfClass:[LaunchFromWXReq class]]) {
//微信终端打开第三方程序携带的消息结构体
if (self.wxDelegate
&& [self.wxDelegate respondsToSelector:@selector(WXApiUtilsDidRecvLaunchFromWXReq:)]) {
LaunchFromWXReq *launchReq = (LaunchFromWXReq *)req;
[self.wxDelegate WXApiUtilsDidRecvLaunchFromWXReq:launchReq];
}
}
}
/*! @brief 发送一个sendReq后，收到微信的回应
*
* 收到一个来自微信的处理结果。调用一次sendReq后会收到onResp。
* 可能收到的处理结果有SendMessageToWXResp、SendAuthResp等。
* @param resp具体的回应内容，是自动释放的
*/
-(void) onResp:(BaseResp*)resp{
if ([resp isKindOfClass: [PayResp class]]){
//支付结果
if (self.wxDelegate
&& [self.wxDelegate respondsToSelector:@selector(WXApiUtilsDidRecvPayResponse:)]) {
PayResp *payResp = (PayResp *)resp;
[self.wxDelegate WXApiUtilsDidRecvPayResponse:payResp];
}
}else if ([resp isKindOfClass:[SendMessageToWXResp class]]) {
//第三方程序向微信终端发送SendMessageToWXReq后，微信发送回来的处理结果，该结果用SendMessageToWXResp表示。
if (self.wxDelegate
&& [self.wxDelegate respondsToSelector:@selector(WXApiUtilsDidRecvMessageResponse:)]) {
SendMessageToWXResp *messageResp = (SendMessageToWXResp *)resp;
[self.wxDelegate WXApiUtilsDidRecvMessageResponse:messageResp];
}
} else if ([resp isKindOfClass:[SendAuthResp class]]) {
//微信处理完第三方程序的认证和权限申请后向第三方程序回送的处理结果
if (self.wxDelegate
&& [self.wxDelegate respondsToSelector:@selector(WXApiUtilsDidRecvAuthResponse:)]) {
SendAuthResp *authResp = (SendAuthResp *)resp;
[self.wxDelegate WXApiUtilsDidRecvAuthResponse:authResp];
}
} else if ([resp isKindOfClass:[AddCardToWXCardPackageResp class]]) {
//微信返回第三方添加卡券结果
if (self.wxDelegate
&& [self.wxDelegate respondsToSelector:@selector(WXApiUtilsDidRecvAddCardResponse:)]) {
AddCardToWXCardPackageResp *addCardResp = (AddCardToWXCardPackageResp *)resp;
[self.wxDelegate WXApiUtilsDidRecvAddCardResponse:addCardResp];
}
}
}
```
4.具体登录调用
在登录之前我们可以判断用户是否安装微信客户端
```
//是否安装微信
- (BOOL)isWXAppInstalled
{
return [WXApi isWXAppInstalled];
}
```
然后再调用登录方法
```
//登录方法
- (void)WXOauthLogin
{
SendAuthReq* req = [[SendAuthReq alloc] init];
req.scope = @"snsapi_userinfo,snsapi_base";
req.state = @"0744" ;
[WXApi sendReq:req];

}
```
登录成功会在-(void) onResp:(BaseResp*)resp方法中回调成功结果,可以获取登录用户的信息
```
/**
* 登录成功获取用户个人信息回调
*/
- (void)QQApiUtilsGetUserInfoResponse:(APIResponse*) response tencentOAuth:(TencentOAuth *)tencentOAut{
//获取到QQ的用户信息
NSLog(@"===%@",response.jsonResponse);
NSInteger gender = 0;
if ([response.jsonResponse[@"gender"] isEqualToString:@"男"]){
gender = 1;
}
NSMutableDictionary * params = [NSMutableDictionary dictionary];
[params setValue:tencentOAut.openId forKey:@"openid"];//openid【必须】
[params setValue:[response.jsonResponse objectForKey:@"nickname"] forKey:@"nickName"];//QQ昵称【必须】
[params setValue:[response.jsonResponse objectForKey:@"figureurl_qq_2"] forKey:@"avatar"];//头像【必须】
[params setValue:gender == 1 ? @"男":@"女" forKey:@"sex"];//性别【必须】
//登录成功回调
#warning 需要开发调用自己的登录接口
[self loginSuccess:params];
}
```
获取到用户信息之后,我们可以根据自己公司的业务做后续处理
####1.2微信分享(好友分享,朋友圈分享)
分享操作指南官方也有对应的知道文档[微信分享操作指南](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419317332&token=221ba501dec7869e36e52e0a34b1b6fde58fe54c&lang=zh_CN)
微信分享目前支持文字、图片、音乐、视频、网页共五种类型,这块我主要写了四种类型,基本上都一样只是中间调用的类不一样
分享的场景有两种
>发送到聊天界面——WXSceneSession
发送到朋友圈——WXSceneTimeline
######1.2.1分享的几种方式
1.网页
```
//网页类型分享
- (BOOL)sharedLinkToWeChat:(NSString *)title
description:(NSString *)description
detailUrl:(NSString *)detailUrl
image:(UIImage *)image
shareType:(WXShareSceneType)sharedType
{
UIImage *compressedImage = [image imageWithFileSize:32*1024 scaledToSize:CGSizeMake(300, 300)];

WXMediaMessage *message = [WXMediaMessage message];

message.title = title;

message.description = description;

[message setThumbImage:compressedImage];

WXWebpageObject *webpageObject = [WXWebpageObject object];

webpageObject.webpageUrl = detailUrl;

message.mediaObject = webpageObject;

SendMessageToWXReq *req = [[SendMessageToWXReq alloc] init];

req.bText= NO;

req.message = message;

req.scene = (sharedType == WXShareSceneTypeTimeline? WXSceneTimeline:WXSceneSession);
BOOL success = [WXApi sendReq:req];
return success;
}
```
2.图片
```
//分享图片
-(BOOL)shareImageToWeChat:(UIImage *)image
shareType:(WXShareSceneType)sharedType{

UIImage *compressedImage = [image imageWithFileSize:32*1024 scaledToSize:CGSizeMake(300, 300)];
WXMediaMessage *message = [WXMediaMessage message];
[message setThumbImage:compressedImage];

WXImageObject *ext = [WXImageObject object];
ext.imageData = UIImagePNGRepresentation(image);
message.mediaObject = ext;

SendMessageToWXReq* req = [[SendMessageToWXReq alloc] init];
req.bText = NO;
req.message = message;
req.scene = (sharedType == WXShareSceneTypeTimeline? WXSceneTimeline:WXSceneSession);

BOOL success = [WXApi sendReq:req];
return success;
}
```
3.音乐类型
```
//音乐类型分享
- (BOOL)sharedMusicToWeChat:(NSString *)title
description:(NSString *)description
musicUrl:(NSString *)musicUrl
musicDataUrl:(NSString *)musicDataUrl
image:(UIImage *)image
shareType:(WXShareSceneType)sharedType
{
UIImage *compressedImage = [image imageWithFileSize:32*1024 scaledToSize:CGSizeMake(300, 300)];

WXMediaMessage *message = [WXMediaMessage message];
//标题
message.title = title;
//描述
message.description = description;
//缩略图
[message setThumbImage:compressedImage];

WXMusicObject * musicObject = [WXMusicObject object];
//音乐的url
musicObject.musicUrl = musicUrl;
//低分辨率音频
musicObject.musicLowBandUrl = musicUrl;
//音乐数据的url
musicObject.musicDataUrl = musicDataUrl;
musicObject.musicLowBandDataUrl = musicDataUrl;
message.mediaObject = musicObject;

SendMessageToWXReq *req = [[SendMessageToWXReq alloc] init];

req.bText= NO;

req.message = message;

req.scene = (sharedType == WXShareSceneTypeTimeline? WXSceneTimeline:WXSceneSession);
BOOL success = [WXApi sendReq:req];
return success;
}
```
4.视频
```
//视频类型分享
- (BOOL)sharedVideoToWeChat:(NSString *)title
description:(NSString *)description
videoUrl:(NSString *)videoUrl
image:(UIImage *)image
shareType:(WXShareSceneType)sharedType
{
UIImage *compressedImage = [image imageWithFileSize:32*1024 scaledToSize:CGSizeMake(300, 300)];

WXMediaMessage *message = [WXMediaMessage message];
//标题
message.title = title;
//描述
message.description = description;
//缩略图
[message setThumbImage:compressedImage];

WXVideoObject * videoObject = [WXVideoObject object];
//视频的url
videoObject.videoUrl = videoUrl;
videoObject.videoLowBandUrl = videoUrl;
message.mediaObject = videoObject;

SendMessageToWXReq *req = [[SendMessageToWXReq alloc] init];

req.bText= NO;

req.message = message;

req.scene = (sharedType == WXShareSceneTypeTimeline? WXSceneTimeline:WXSceneSession);
BOOL success = [WXApi sendReq:req];
return success;
}
```
######1.2.2分享成功的回调
分享成功的回调也是在-(void) onResp:(BaseResp*)resp方法中处理
```
-(void) onResp:(BaseResp*)resp{
if ([resp isKindOfClass:[SendMessageToWXResp class]]) {
//第三方程序向微信终端发送SendMessageToWXReq后，微信发送回来的处理结果，该结果用SendMessageToWXResp表示。
if (self.wxDelegate
&& [self.wxDelegate respondsToSelector:@selector(WXApiUtilsDidRecvMessageResponse:)]) {
SendMessageToWXResp *messageResp = (SendMessageToWXResp *)resp;
[self.wxDelegate WXApiUtilsDidRecvMessageResponse:messageResp];
}
} 
}
```
以上就是微信登录和分享的整个流程,具体代码[demo](https://github.com/MrGCY/ThirdPartyShareLoginAndPay),也请大家继续关注,后续会分享QQ的登录和分享
