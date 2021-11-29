**配置**

appid wxf50efe73475465fb

secret 9e05b081d22377ad7152f40ba23a3dee

Token lilaiqun

EncodingAESKey jjCJ8xOsNBad23m4N0l4weST0cOR4UQevNkWjn2mfG2

![image](../../images/image.png)



**第三方sdk**

https://www.easywechat.com/

```
# thinkphp5.1
composer create-project topthink/think=5.1.* tp5
 
# easywechat.com
composer require overtrue/wechat:~4.0 -vvv
```



**消息格式**

```xml
<xml>
  <ToUserName><![CDATA[toUser]]></ToUserName>
  <FromUserName><![CDATA[fromUser]]></FromUserName>
  <CreateTime>1348831860</CreateTime>
  <MsgType><![CDATA[text]]></MsgType>
  <Content><![CDATA[this is a test]]></Content>
  <MsgId>1234567890123456</MsgId>
</xml>
```



| 参数         | 描述                     |
| ------------ | ------------------------ |
| ToUserName   | 开发者微信号             |
| FromUserName | 发送方帐号（一个OpenID） |
| CreateTime   | 消息创建时间 （整型）    |
| MsgType      | 消息类型，文本为text     |
| Content      | 文本消息内容             |
| MsgId        | 消息id，64位整型         |