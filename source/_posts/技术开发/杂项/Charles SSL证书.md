---
title: Windows下Charles支持HTTPS
tags: charles
toc: true
---


## SSL握手过程

### 客户端发出加密通信请求

提供：

- 协议版本(如 TSL1.0)
- 随机数 1(用于生成对话密钥) 
- 支持的加密方法(如 RSA 公钥加密) 
- 支持的压缩方法

### 服务器回应

回应内容: 

- 确认使用的加密通信协议版本(TSL1.0) 
- 随机数 2(用于生成对话密钥)
- 确认加密方法(RSA) 
- 服务器证书(包含非对称加密的公钥) 
- (可选)要求客户端提供证书的请求

### 客户端验证证书

如果证书不是可信机构颁布，或证书域名与实际域名不符，或者证书已经过期，就会向访问者显示一个警告，是否继续通信

### 客户端回应

证书没有问题，就会取出证书中的服务器公钥
然后发送:

- 随机数 3(pre-master key，此随机数用服务器公钥加密，防止被窃听) 
- 编码改变通知(表示随后的信息都将用双方商定的方法和密钥发送) 
- 客户端握手结束通知

### 双方生成会话密钥

双方同时有了三个随机数，接着就用事先商定的加密方法，各自生成同一把“会 话密钥” 服务器端用自己的私钥(非对称加密的)获取第三个随机数，
会计算生成本次所 用的会话密钥(对称加密的密钥)，如果前一步要求客户端证书，会在这一步验证。

### 服务器最后响应

服务器生成会话密钥后，向客户端发送: 

- 编码改变通知(后面的信息都用双方的加密方法和密钥来发送) 
- 服务器握手结束通知


## Charles 设置

### SSL设置

点击 `Proxy` -> `SSL Proxy Settings` -> `SSLProxy` -> `Add`

添加SSL代理规则

    Host:*
    Port:443

### 证书配置(不用设置)

点击 `Help` -> `SSL Proxying` -> `Install Charles Root Certificate` -> `安装证书`

选择将所有的证书都放入`受信任的根证书颁发机构`，钥匙链中双击证书，选择`信任`，使用此证书时，选择`始终信任`


### 手机配置

连接跟电脑一样的网络，配置代理连接charles（在wifi中配置IP、port）。浏览器访问`chls.pro/ssl`，下载安装证书


## 注意事项

Android7及以上无法抓取https，因为"Network Security Configuration"的新安全功能。这个功能运行开发人员在不修改应用程序的情况下
自定义他们的网络安全设置。如果应用程序运行的系统版本高于或等于24，并且targetSdkVersion>=24，则只有系统证书才被信任。


## 参考

- [https://www.charlesproxy.com/documentation/using-charles/ssl-certificates/](https://www.charlesproxy.com/documentation/using-charles/ssl-certificates/)
- [https://juejin.im/post/5b4f005ae51d45191c7e534a](https://juejin.im/post/5b4f005ae51d45191c7e534a)
