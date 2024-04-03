---
title: npm发布
toc: true
tags: npm
---


## 配置package.json



## 发布到Nexus

[将npm本地包上传到nexus私服的实践](https://wiki.eryajf.net/pages/18ca89)

> 发布Nexus服务器，需要注意不同仓库地址的使用

```shell

# 添加账号
npm adduser registry http://localhost:8081/repository/npm-group/

# 登录账号
npm login

# 发布
npm publish --registry=http://localhost:8081/repository/npm-hosted/
```
