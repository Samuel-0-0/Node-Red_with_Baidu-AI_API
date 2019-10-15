# 第一章 序

## Node-Red基础
Node-Red基础教程略，可以参考萝卜的教程。

## 百度AI开放平台介绍
目前为止，各大服务商中，也只有百度比厚道，提供了很多免费的AI服务供我们玩耍，百度AI开放平台的地址是https://ai.baidu.com/docs#/。

# 第二章 百度AI服务基础

## 在Node-Red中怎么使用
首先你需要已经了解Node-Red，并会使用function等基础节点。

在Node-Red中，可以有2种方式使用百度AI服务。
1. 安装百度AI服务的nodejs-sdk
2. 使用网络API接口

本次主要介绍网络API接口的使用

## 百度AI服务网络API接口调用
向API服务地址使用POST发送请求，必须在URL中带上参数：access_token，如

```
https://aip.baidubce.com/rest/2.0/image-classify/v2/dish?access_token=24.f9ba9c5241b67688bb4adbed8bc91dec.2592000.1485570332.282335-8574074
```
具体的请求内容在后面章节介绍。

## access_token的获取
要获取access_token需要百度云应用的AK、SK，
以图像识别为例，首先登陆https://console.bce.baidu.com/ai/#/ai/imagerecognition/overview/index并点击创建应用。

![image](https://raw.githubusercontent.com/Samuel-0-0/Node-Red_with_Baidu-AI_API/master/picture/01.png)

接下去你要做的是
1. 填写应用名称
2. 选择应用类型
3. 接口选择（除了智能呼叫中心能选的都选上）
4. 文字识别包名，选择不需要
5. 语音包名，选择不需要
6. 填写应用描述
7. 点击立即创建
8. 查看应用详情
9. 此时就可以看到应用的API Key和Secret Key，即AK、SK

接着，我们打开Node-Red，添加1个inject节点、2个function节点、1个http request节点、1个debug节点，按照下图顺序连接起来，

![image](https://raw.githubusercontent.com/Samuel-0-0/Node-Red_with_Baidu-AI_API/master/picture/02.png)

打开第一个function节点，输入以下内容

```
msg.headers = {
    "Content-Type" : "application/x-www-form-urlencoded"
}
msg.payload = {
    "grant_type" : "client_credentials",
    "client_id" : "百度云应用的AK",
    "client_secret" : "百度云应用的SK"
}
return msg;

```
打开http request节点，按照下图设置好

![image](https://raw.githubusercontent.com/Samuel-0-0/Node-Red_with_Baidu-AI_API/master/picture/03.png)

填好后，我们先测试一下，先点击右上角的部署，只后点击inject，这时在调试窗口可以看到有信息输出，如下图

![image](https://raw.githubusercontent.com/Samuel-0-0/Node-Red_with_Baidu-AI_API/master/picture/04.png)

其中的access_token就是我们需要的，
expires_in表示失效的实际，单位为秒，换算后是30天。
为了便于使用access_token，我们再对第二个function就行修改，

```
flow.set('access_token', msg.payload.access_token); //将access_token储存
return msg;
```
因为access_token具有时效性，为了防止失效，修改inject，改成间隔执行，间隔1小时（或者自定义）
