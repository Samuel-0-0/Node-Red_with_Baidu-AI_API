# 第三章 案例1 图像识别
## 接口能力分析
图像识别具体是在图像技术分类下，地址为：https://ai.baidu.com/docs#/ImageClassify-API/top

![image](https://raw.githubusercontent.com/Samuel-0-0/Node-Red_with_Baidu-AI_API/master/picture/05.png)

打开页面，依次打开并找到API文档，首先分析接口能力，以判断是否符合我们预期的需求。假设我们想要识别一个不确定的物体（可能是纸箱，可能是瓶子，可能是椅子），根据接口能力的描述，我们选择通用物体和场景识别高级版作为本次需要调用的API。

## 使用API
根据请求说明中请求示例的描述，

> HTTP 方法：POST
> 
> 请求URL： https://aip.baidubce.com/rest/2.0/image-classify/v2/advanced_general
> 
> URL参数：
> 
> 参数 | 值
> ---|---
> access_token | 通过API Key和Secret Key获取的access_token,参考“Access Token获取”
> Header如下：
> 
> 参数 | 值
> ---|---
> Content-Type | application/x-www-form-urlencoded
> Body中放置请求参数，参数详情如下：
> 请求参数
> 
> 参数 | 是否必选 | 类型 | 可选值范围 | 说明
> ---|---|---|---|---
> image | 是 | string | - | Base64编码字符串，以图片文件形式请求时必填。(支持图片格式：jpg，bmp，png，jpeg)，图片大小不超过4M。最短边至少15px，最长边最大4096px。注意：图片需要base64编码、去掉编码头后再进行urlencode。
> baike_num | 否 | integer | 0-5 | 返回百科信息的结果数，默认为0，不返回；2为返回前2个结果的百科信息，以此类推。

从上面的描述中，我们得知，需要发送一个POST的http请求到

```
https://aip.baidubce.com/rest/2.0/image-classify/v2/advanced_general
```

请求需要URL参数access_token，于是经过拼装后的请求地址为：

```
https://aip.baidubce.com/rest/2.0/image-classify/v2/advanced_general?access_token=【我们在第二章获得的access_token】
```

因为我们已经将access_token保存到flow变量了，所以重新修改一下请求地址：

```
https://aip.baidubce.com/rest/2.0/image-classify/v2/advanced_general?access_token=${flow.get('access_token')}
```

和第二章获取access_token的请求类似，最终我们配置一个function节点，内容为：

```
msg.headers = {
    "Content-Type" : "application/x-www-form-urlencoded"
}
msg.url = `https://aip.baidubce.com/rest/2.0/image-classify/v2/advanced_general?access_token=${flow.get('access_token')}`;
msg.payload = {
    image:image_base64, //图片Base64编码
    baike_num:5 //返回百科信息的结果数量
}

return msg;

```

前面的请求参数中提到image是需要Base64编码字符串，那么怎么获得这个字符串呢？

假设我们需要检查一张来自网络的图像，只需要在前面增加1个http request节点和1个base64节点即可。

最终，我们配置这样一条流来进行图像识别。

![image](https://raw.githubusercontent.com/Samuel-0-0/Node-Red_with_Baidu-AI_API/master/picture/06.png)

可以直接复制导入到Node-Red中测试：
```
[{"id":"5faf7164.60b2a","type":"function","z":"196a5cf2.9b52f3","name":"通用物体识别","func":"msg.headers = {\n    \"Content-Type\" : \"application/x-www-form-urlencoded\"\n}\nmsg.url = `https://aip.baidubce.com/rest/2.0/image-classify/v2/advanced_general?access_token=${flow.get('access_token')}`;\nmsg.payload = {\n    image:msg.payload, //图片Base64编码\n    baike_num:5 //返回百科信息的结果数量\n}\n\nreturn msg;\n","outputs":1,"noerr":0,"x":600,"y":260,"wires":[["7303ff7c.0ef9a"]]},{"id":"7303ff7c.0ef9a","type":"http request","z":"196a5cf2.9b52f3","name":"","method":"POST","ret":"obj","paytoqs":false,"url":"","tls":"","persist":false,"proxy":"","authType":"","x":760,"y":260,"wires":[["bfb46f16.6ab98"]]},{"id":"47a2bcd6.d4a644","type":"base64","z":"196a5cf2.9b52f3","name":"","action":"","property":"payload","x":440,"y":260,"wires":[["5faf7164.60b2a"]]},{"id":"bfb46f16.6ab98","type":"debug","z":"196a5cf2.9b52f3","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload","targetType":"msg","x":910,"y":260,"wires":[]},{"id":"de360e93.54772","type":"inject","z":"196a5cf2.9b52f3","name":"","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":170,"y":260,"wires":[["6fcd0d9d.4ba3a4"]]},{"id":"6fcd0d9d.4ba3a4","type":"http request","z":"196a5cf2.9b52f3","name":"","method":"GET","ret":"bin","paytoqs":false,"url":"https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1571140367297&di=c89684813fd816c843313217e07feb0d&imgtype=0&src=http%3A%2F%2Fn.sinaimg.cn%2Fsinacn20%2F666%2Fw400h266%2F20180604%2Fb404-hcmurvh7196381.jpg","tls":"","persist":false,"proxy":"","authType":"","x":300,"y":260,"wires":[["47a2bcd6.d4a644","e7009a1c.201908"]]},{"id":"e7009a1c.201908","type":"image","z":"196a5cf2.9b52f3","name":"","width":160,"x":460,"y":300,"wires":[]}]
```
