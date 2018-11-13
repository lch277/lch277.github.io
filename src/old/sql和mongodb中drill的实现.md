---
title: sql和mongodb中drill的实现
date: 2017-02-10 16:45:10
tags:
- drill
- sql
- mongodb
---



drill是多维分析中一个很典型的操作

通常多维数据包含：

+ 用户表，包含id、name、性别、年龄、年级、位置以及各种偏好等多维度信息
+ 事实表，包含用户行为的多维信息表



drill操作通常包含：

- 选定用户群体（按用户的维度信息筛选）
- 用户群体的特定行为（按行为的维度信息组合筛选）
- 对上述行为按维度分组（用户维度、行为维度、时间维度）
- 对分组的数据进行聚合计算（max、min、mean、count）



mysql实现

```sql
select * from users a join events b on a.uid=b.uid where a.xxxxx and b.xxxx group by a.xxx, b.xxx order by a.xxx, b.xxx limit 10
```



mongodb实现

```javascript
db.users.aggregate([
    {$match: {reg_time: {$gte: ISODate("2017-01-14T01:40:25.000Z")}}},
    {$lookup: {from: 'events', localField: 'uid', foreignField: 'uid', as: 'events'}},
  	{...}, // 打散
    {...}, // 分组聚合
])
```

