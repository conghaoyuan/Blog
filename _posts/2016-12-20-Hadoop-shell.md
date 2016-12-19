---
layout: post
title: Hadoop Shell 相关操作
categories:
- 大数据
---

## 1、HDFS Shell

既然 `HDFS` 是存取数据的分布式文件系统,那么对 `HDFS`的操作,就是文件系统的基本操作,比如文件的创建、修改、删除、修改权限等,文件夹的创建、删除、重命名等。对 HDFS 的操作命令类似于Linux 的 shell对文件的操作,如 ls、mkdir、rm 等。

执行：`hadoop fs`，可以查看hdfs 的指令：

| 选项名称       | 使用格式      | 含义  |
| ------------- |:----------------:| --------:|
| -ls	| -ls <路径> | 查看指定路径的当前目录结构 |
|-lsr	|-lsr <路径>	| 递归查看指定路径的目录结构|
|-du	| -du <路径> | 统计目录下个文件大小|
|-dus	|-dus <路径>	| 汇总统计目录下文件(夹)大小|
|-count	|-count [-q] <路径>	| 统计文件(夹)数量|
|-mv	|-mv <源路径> <目的路径>	| 移动|
|-cp	|-cp <源路径> <目的路径>	|复制|
|-rm	|-rm [-skipTrash] <路径>	|删除文件/空白文件夹|
|-rmr	|-rmr [-skipTrash] <路径>	|递归删除|
|-put	|-put <多个 linux 上的文件> <hdfs 路径>	|上传文件|
|-copyFromLocal|-copyFromLocal <多个 linux 上的文件> <hdfs 路径>|从本地复制|
|-moveFromLocal|-moveFromLocal <多个 linux 上的文件> <hdfs 路径>|从本地移动|
|-getmerge|-getmerge <源路径> <linux 路径>|合并到本地|
|-cat|-cat <hdfs 路径>|查看文件内容|
|-text|-text <hdfs 路径>|查看文件内容|
|-copyToLocal|-copyToLocal [-ignoreCrc] [-crc] [hdfs 源路 径] [linux 目的路径]|从本地复制|
|-moveToLocal|-moveToLocal [-crc] <hdfs 源路径> <linux 目的路径>|从本地移动|
|-mkdir|-mkdir <hdfs 路径>|创建空白文件夹|
|-setrep|-setrep [-R] [-w] <副本数> <路径>|修改副本数量|
|-touchz|-touchz <文件路径>|创建空白文件|
|-stat|-stat [format] <路径>|显示文件统计信息|
|-tail|-tail [-f] <文件>|查看文件尾部信息|
|-chmod|-chmod [-R] <权限模式> [路径]|修改权限|
|-chown|-chown [-R] [属主][:[属组]] 路径|修改属主|
|-chgrp|-chgrp [-R] 属组名称 路径|修改属组|
|-help|-help [命令选项]|帮助 |




