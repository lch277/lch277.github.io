---
title: vuex的理解
date: 2016-12-27 20:42:06
tags:
- vuex
- vue
---

vue2的状态管理模式vuex类似react的redux，都是提供一个单一状态树的模型store。

数据操作部分：redux是分别编写action和reducer，然后将state和action映射为组件的props，reducer部分则与组件没有直接关系，只负责修改state；vuex的store中可以定义一些数据操作的方法，getters是读取过滤数据的公共操作，mutations是修改数据的公共操作，actions是异步修改数据的公共操作，然后定义组件时可以映射这些公共操作。

数据模块化：redux可以按store的数据结构自行拆分模块，然后分模块组织action和reducer；vuex则提供了modules，每个module可以拥有自己的state、mutations、actions、getters，甚至是子module——从上到下的分割。

整体来看，vuex和redux提供的功能是类似的，redux把action和reducer分开的做法逻辑上和代码上都更为分散，写起来相对繁琐，而vuex把state和相关操作整合在一起，用户的关注点聚集在数据上，写起来简洁，看起来直观，理解上也较容易。
