---
title: SPA前后端分离下的webpack工作流   
date: 2017-01-19 09:49:39   
tags:   
- SPA
- webpack
- 全栈
---

### 前后端同构
+ 直接在后端集成webpack热加载模块（express、koa均有类似中间件）
+ 优势，无需开启两个服务，无需前端服务代理中转，易实现spa的服务器端渲染
+ 劣势，适用性差，增加后端开发复杂度，不适合初学者，易出错
+ 实现
    + 使用`webpack、webpack-dev-middleware、webpack-hot-middleware`等模块

### 前后端异构
+ spa前端单独开启一个服务，处理静态页面请求和中转后端数据api
+ 后端可以处理静态页面和数据api，逻辑清晰简单，易于集成发布
+ 后端可以微服务化，可以应对大型复杂spa
+ 实现
    + 使用`webpack-dev-server`前端服务，配置devServer和proxy
    + 自行实现dev server，使用`http-proxy-middleware`