---
title: Elastic-Stack-5升级
date: 2017-02-22 11:22:27
tags:
- Elastic Stack

---



[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html#setup-upgrade)

## 迁移插件

[Elasticsearch Migration Plugin](https://github.com/elastic/elasticsearch-migration/) 安装失败，可能是配置失败



## 备份数据

快照需要共享文件系统或者hdfs，配置hdfs需要启用elasticsearch-repository-hdfs插件，插件需要适配elasticsearch版本和hadoop版本，折腾几次还是没有配置成功

直接备份elasticsearch各节点的data目录



## Elasticsearch

### 1. 禁用分片分配

当关闭节点时，分配进程将立即尝试将该节点上的分片复制到集群中的其他节点，从而导致大量浪费的I / O。这可以通过在关闭节点之前禁用分配来避免：

```
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": "none"
  }
}
```

### 2. 执行同步刷新 

如果停止索引并发出同步刷新请求，分片恢复将快得多：

```
POST _flush/synced
```

同步刷新请求是“尽力而为”操作。如果有任何挂起的索引操作，它将失败，但是如果必要，可以安全地多次重新发出请求。

### 3. 关闭并升级所有节点

停止集群中所有节点上的所有Elasticsearch服务。可以按照[upgrade-node](https://www.elastic.co/guide/en/elasticsearch/reference/current/rolling-upgrades.html#upgrade-node)中描述的相同过程升级每个节点。

### 4. 升级单个节点

**在**开始升级**之前**关闭集群中的其中一个节点。

> 使用zip或tarball软件包时，默认情况下`config`，`data`，`logs`和 `plugins`目录位于Elasticsearch主目录中。
>
> 最好将这些目录放在不同的位置，以便在升级Elasticsearch时不会删除它们。这些自定义的路径可以通过设置项`path.conf`， `path.logs`，和`path.data`来配置，并使用`ES_JVM_OPTIONS`指定`jvm.options`文件的位置。
>
> [Debian的](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html)和[RPM](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html)安装包会将这些目录放置在每个操作系统的适当位置。

使用[Debian](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html)或[RPM](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html)软件包进行升级：

- 使用`rpm`或`dpkg`安装新软件包。所有文件应放置在正确的位置，配置文件不应被覆盖。使用zip或压缩tarball进行升级：


- 将zip或tarball解压缩到新目录，以确保不会覆盖`config`或`data`目录。
- 将目录中的文件`config`从旧安装复制到新安装，或将环境变量设置`ES_JVM_OPTIONS`为`jvm.options`文件的位置，并使用`-E path.conf=` 命令行上的选项指向外部config目录。
- 可以将目录中的文件`data`从旧安装复制到新安装，也可以`config/elasticsearch.yml`使用该`path.data`设置配置文件中数据目录的位置。

### 5. 升级任何插件

升级节点时必须升级Elasticsearch插件。使用 `elasticsearch-plugin`脚本安装所需的任何插件的正确版本。

### 6. 坑

注意仔细阅读elasticsearch及其插件的broken changes，在升级前去掉数据中使用的不支持特性，以免升级完成后数据在新版本中出问题

### 7. ik

copy and unzip `target/releases/elasticsearch-analysis-ik-{version}.zip` to `your-es-root/plugins/ik`



## Kibana

下载新版本的rpm包，yum安装，使用之前版本的配置文件即可

> **注意：**
>
> 新版本kibana默认从`/etc/kibana/kibana.yml`加载配置文件，之前版本是在kibana安装目录的config文件夹中加载



## Logstash

下载新版本的rpm包，yum安装，使用之前的配置文件即可

> **注意：**
>
> 新版本使用`initctl start logstash`，之前版本使用 service 命令启动 logstash 服务



## X-PACK

### 安装

1. 下载X-PACK的压缩文件：[`https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-5.2.1.zip`](https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-5.2.1.zip) ([sha1](https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-5.2.1.zip.sha1))
2. 放置压缩文件到节点机器的临时目录
3. 安装为 ElasticSearch 插件`bin/elasticsearch-plugin install file:///path/to/file/x-pack-5.2.1.zip`
4. 安装为 Kibana 插件`bin/kibana-plugin install file:///path/to/file/x-pack-5.2.1.zip`
5. 安装为 Logstash 插件`bin/logstash-plugin install file:///path/to/file/x-pack-5.2.1.zip`

### 配置

X-PACK 默认打开所有功能，可以通过配置 `elasticsearch.yml`和`kibana.yml`来打开关闭功能

| Setting                    | Description                              |
| -------------------------- | ---------------------------------------- |
| `xpack.security.enabled`   | Set to `false` to disable X-Pack security. Configure in both `elasticsearch.yml` and `kibana.yml`. |
| `xpack.monitoring.enabled` | Set to `false` to disable X-Pack monitoring. Configure in both `elasticsearch.yml` and `kibana.yml`. |
| ` xpack.graph.enabled`     | Set to `false` to disable X-Pack graph. Configure in both `elasticsearch.yml` and `kibana.yml`. |
| ` xpack.watcher.enabled`   | Set to `false` to disable Watcher. Configure in `elasticsearch.yml` only. |
| `xpack.reporting.enabled`  | Set to false to disable X-Pack reporting. Configure in kibana.yml only. |

### 监控

x-pack安装完成默认会自动监控elasticsearch和kibana，要监控logstash需要打开如下配置：

```
xpack.monitoring.elasticsearch.url: "http://es-prod-node:9200" 
xpack.monitoring.elasticsearch.username: "logstash_system" 
xpack.monitoring.elasticsearch.password: "changeme"
```



## SearchGuard

todo