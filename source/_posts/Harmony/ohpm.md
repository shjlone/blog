---
title: ohpm
tags: Harmony 
toc: true
---


ohpm作为OpenHarmony三方库的包管理工具，支持OpenHarmony共享包的发布、安装和依赖管理。



## oh-package.json5


```json
{
  "modelVersion": "5.0.0",
  "description": "Please describe the basic information.",
  "dependencies": {
  },
  "devDependencies": {
    "@ohos/hypium": "1.0.19",
    "@ohos/hamock": "1.0.0"
  },
  "overrides":{

  }
}


```



## ohpm命令


```shell

# 展示所有配置
ohpm config list

# 展示单个属性配置
ohpm config get <key>

ohpm config set <key> <value>

ohpm config delete <key>

# 查询某个库的信息
ohpm info @ohos/lottie

# 创建oh-package.json5
ohpm init [options]

# 安装某个库
ohpm install @ohos/lottie

# 列出所有库
ohpm list

ohpm uninstall lottie

# 清理 ohpm 缓存文件夹。
ohpm cache clean

# 执行用户自定义脚本
ohpm run script_name -- -agr1=1 --arg2=2 -arg3 3 --arg4 4

# 清理工程下所有模块的ohpm安装产物
ohpm clean 

```

