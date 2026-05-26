---
layout:     post
title:      SpringMVC 的常用注解
subtitle:   Java学习笔记
date:       2024-10-17
author:     DM
header-img: img/post-bg-debug.png
catalog:    true
tags:       Java Spring
---

## 一、组件型注解
- Component 在类定义之前添加 Component注解，他会被spring 容器识别，并转为bean。
- Repository对Dao实现类进行注解（特殊的Component）
- Service用于对业务逻辑层进行注解（特殊的Component）
- Controller用于控制层注解 (特殊的Component)
以上四种注解都是注解在类上的，被注解的类将被spring初始化为一个bean,然后统一管理
