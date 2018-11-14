---
title: 自建mongodb集群增加用户鉴权    
date: 2017-02-21 11:21:13   
tags:   
- mongodb
---

# 副本集的鉴权
+ 副本成员之间的内部鉴权internal authentication
+ 副本集与客户端之间的基于角色的用户鉴权role-based user authentication

# 流程
### 创建一个keyfile
```shell
openssl rand -base64 756 > <path-to-keyfile>
chmod 400 <path-to-keyfile>
```
> keyfile的文件权限需要严格遵守，400是必须的，所以最好`chown mongod:mongod <path-to-keyfile>`

### 拷贝keyfile到各个副本集成员
> 同样注意权限问题

### 在各副本成员上启用访问控制
```
security:
  keyFile: <path-to-keyfile>
replication:
  replSetName: <replicaSetName>
```
> 注意，启用keyfile之后，security下的`authentication: enabled`可以去掉了
> net下的`bindIp: localhost,<lan-ip>`确保绑定到内网IP，多个IP之间用英文逗号隔开且不要加额外的空格

### 使用localhost连接到副本集成员
因为副本集刚启动，还没有创建用户，此时只能通过localhost连接副本成员创建用户

### 初始化副本集
副本集名必须与config文件中保持一致，member的host必须为内网ip
```
rs.initiate(
  {
    _id : <replicaSetName>,
    members: [
      { _id : 0, host : "mongo1.example.net:27017" },
      { _id : 1, host : "mongo2.example.net:27017" },
      { _id : 2, host : "mongo3.example.net:27017" }
    ]
  }
)
```

### 创建管理员用户
使用db.createUser()在admin数据库上创建一个权限至少为userAdminAnyDatabase的用户
```
admin = db.getSiblingDB("admin")
admin.createUser(
  {
    user: "fred",
    pwd: "changeme1",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
```

### 其他
参考官方文档 [deploy-replica-set-with-keyfile-access-control](https://docs.mongodb.com/manual/tutorial/deploy-replica-set-with-keyfile-access-control/)