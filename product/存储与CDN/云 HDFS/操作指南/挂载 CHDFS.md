
创建 CHDFS 及挂载点后，可以通过挂载点挂载 CHDFS，本文为您详细介绍如何挂载 CHDFS。

## 前提条件
- 确保挂载的机器或者容器内安装了 Java 1.8。
- 确保挂载的机器或者容器其 VPC，与挂载点指定 VPC 相同。
- 确保挂载的机器或者容器其 VPC IP，与挂载点指定权限组中有一条权限规则授权地址匹配。

## 操作步骤
1. 下载 [CHDFS-Hadoop](https://github.com/tencentyun/chdfs-hadoop-plugin) JAR 包。
2. 将 JAR 包放置对应的目录下，对于 EMR 集群，可同步到所有节点的`/usr/local/service/hadoop/share/hadoop/common/lib/`目录下。
3. 编辑 core-site.xml 文件，新增以下基本配置：
```
<!--chdfs 的实现类-->
<property>
		 <name>fs.AbstractFileSystem.ofs.impl</name>
		 <value>com.qcloud.chdfs.fs.CHDFSDelegateFSAdapter</value>
</property>

<!--chdfs 的实现类-->
<property>
		 <name>fs.ofs.impl</name>
		 <value>com.qcloud.chdfs.fs.CHDFSHadoopFileSystemAdapter</value>
</property>

<!--本地 cache 的临时目录, 对于读写数据, 当内存 cache 不足时会写入本地硬盘, 这个路径若不存在会自动创建-->
<property>
		 <name>fs.ofs.tmp.cache.dir</name>
		 <value>/data/chdfs_tmp_cache</value>
</property>

<!--用户的appId, 可登录腾讯云控制台(https://console.cloud.tencent.com/developer)查看-->      
<property>
		 <name>fs.ofs.user.appid</name>
		 <value>1250000000</value>
</property>

<!--flush语义, 默认false(异步刷盘), 请参考下图数据可见性与持久化详细说明 -->      
<property>
		 <name>fs.ofs.upload.flush.flag</name>
		 <value>false</value>
</property>

```
4. 将 core-site.xml 同步到所有 hadoop 节点上。
>?对于 EMR 集群，以上步骤3、4可在 EMR 控制台的组件管理中，修改 HDFS 配置即可。
>
5. 使用 hadoop fs 命令行工具，运行`hadoop fs -ls ofs://${mountpoint}/`命令，这里 mountpoint 为挂载地址。如果正常列出文件列表，则说明已经成功挂载 CHDFS。
6. 用户也可使用 hadoop 其他配置项，或者 mr 任务在 CHDFS 上运行数据任务。对于 mr 任务，可以通过`-Dfs.defaultFS=ofs://${mountpoint}/`将本次任务的默认输入输出 FS 改为 CHDFS。

## 场景说明

### 写入数据可见性与持久化

CHDFS 是云端分布式文件系统，通过 CHDFS 客户端打开一个文件，会返回一个输出流；通过输出流写入数据后， CHDFS 客户端会在满足一定大小后(通常为4MB)，异步刷盘。如果上层主动调用输出流的 flush 接口, **默认的行为是忽略（和通常的 flush 同步刷盘语义不同）**。等待后续满足写入量达到一定大小后异步刷盘。或者最后调用 close 的时候，也才会强制同步刷盘。 close 成功的数据, 表示已经持久化成功，可保证后续读取到。

CHDFS 客户端默认异步 flush，因为相比起本地磁盘，云端的访问时延较大，异步 flush 可以减少与云端的交互，避免 flush 卡主用户写入，显著提升写入性能，减少作业时间。风险点是如果最终没有主动调用 close，会存在未刷盘的数据丢失的风险。
 
 **对于需要 flush 后需要立刻可见以及持久化的场景，例如使用 CHDFS 存储 Hbase Wal Log，Journal 日志等，请开启同步刷盘**。请参考配置项 fs.ofs.upload.flush.flag 说明。

## 配置项说明

|        配置项      |                             说明                             |  默认值   | 是否必填 | 版本要求 |
| :------------------------------| :----------------------------------------------------| :-------| :------ | :-------|
|       fs.ofs.tmp.cache.dir        |   存放临时数据    |    无     |    是    | 无 |
|       fs.ofs.map.block.size       | chdfs 文件系统的 block 大小，单位为字节。默认为128MB（只对 map 切分有影响，和 chdfs 底层存储切块大小无关） | 134217728 |    否    | 无 |
| fs.ofs.data.transfer.thread.count |               chdfs 传输数据时的并行线程数                |    32     |    否    |  无 |
| fs.ofs.block.max.memory.cache.mb  | chdfs 插件使用的内存 buffer 的大小，单位为 MB。(对读写都有加速作用) |    16     |    否    | 无 |
|  fs.ofs.block.max.file.cache.mb   |  chdfs 插件使用的磁盘 buffer 的大小，单位为 MB。（对写有加速作用）  |    256    |    否    |  无  |
|   fs.ofs.prev.read.block.count    | 读取时，预读的 chdfs block 数量（chdfs 的底层 block 大小一般为4MB）|     4     |    否    |  chdfs_hadoop_plugin-x.x.x-shaded 1.1.6(含) 版本后默认为 16  |
|      fs.ofs.plugin.info.log       |          是否打印插件的调试日志，日志以 info 级别打印。可选值为 true、false |   false   |    否    |   无  |
|      fs.ofs.bucket.region       |          文件系统或者元数据加速器 bucket 所在的地域，如 ap-shanghai，ap-beijing|   false   |    否    | chdfs_hadoop_plugin_network 版本V2.7及其以上   |
| fs.ofs.meta.server.port | 元数据端口 | 443 | 否 | 无 |
| fs.ofs.meta.transfer.tls | 元数据是否使用 tls | true | 否 | 无  |
| fs.ofs.data.transfer.https | 数据流是否使用 tls | false | 否 |  无 |
| fs.ofs.meta.send.max.retry | 元数据请求重试次数 | 10 | 否 |  无 |
| fs.ofs.data.transfer.endpoint.suffix | 数据流自定义域名 | 无 | 否 |  无 |
| fs.ofs.prev.read.block.release.enable | 开启预读 | true | 否 | 无  |
| fs.ofs.meta.endpoint.suffix | 元数据自定义域名 | 无 | 否 | chdfs_hadoop_plugin-x.x.x-shaded 1.1.2(含)后支持 |
| fs.ofs.data.transfer.distinguish.host | 拆分endpoint 和 Host，使用未备案域名时须配置为 true | false | 否 | chdfs_hadoop_plugin-x.x.x-shaded 1.1.2(含)后支持 |



