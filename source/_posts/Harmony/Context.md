---
title: Context
tags: Harmony 
toc: true
---


- UIAbilityContext：UIAbility对应的context，继承自Context，提供UIAbility的相关方法
- AbilityStageContext：AbilityStage对应的context，提供AbilityStage的相关方法，比如获取ModuleInfo对象、环境变化对象
- ApplicationContext：应用级上下文，提供注册组件生命周期等接口
- BaseContext
- Context：继承自BaseContext，提供ability或application的上下文能力，包括访问资源、启动ability等
- ExtensionContext：提供访问特定Extension的资源的能力
- FormExtensionContext
- AbilityResult