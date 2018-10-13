---
title: 一次RabbitMQ线上故障记录
date: 2018-04-19 21:08:49
tags: mq
---
今天在线上环境中出现了一个很奇怪的RabbitMQ故障，特意记录下。
* 出现现象
* 排查过程
* 事后分析
---
<!-- more -->
## 出现原因
由于公司有一个老版本系统在运维，是部署在Windows Server2012的环境上。今天这个系统出现了异常，关于队列有关的功能出现了不可用。因为该系统架构以及代码都比较老旧，而且正在逐步停止运维，定位问题难度比较高。

具体现象如下：当运维人员发现系统出现异常后，重启了系统，但是还是没有效果。然后查看消息队列RabbitMQ的状态，发现处于停止状态，随后启动RabbitMQ，回到系统一看发现还是不可用。查看系统日志后发现还是无法连接队列，多次启动RabbitMQ，最后发现RabbitMQ无法启动。

## 排查过程
### 查找日志
由于多次启动RabbitMQ失败，所以去查看RabbitMQ日志，由于安装RabbitMQ时并未改变日志路径，所以日志输出在默认路径	%APPDATA%\RabbitMQ。
https://www.rabbitmq.com/relocate.html
{% asset_img 一次RabbitMQ线上故障记录-1.png 一次RabbitMQ线上故障记录 %}

### 错误定位
看日志时发现日志文件已经有600M，无法用Notepad++等工具打开，然后暂时删除日志文件，重启RabbitMQ希望重现问题抓取日志，但是操作几次后从新的日志文件看不出问题。
偶然查看磁盘空间，发现剩余0%（0字节可用），因为这台服务器是租用阿里云的资源，磁盘空间有限，可能由于一些不恰当的操作，导致服务器磁盘。
删除一些无效文件，使磁盘有空余空间，然后重启RabbitMQ服务，但是查看5672端口还是没有监听。

因为从Windows服务启动信息不够，暂时先从命令行启动查看，发现错误信息如下：
{% asset_img 一次RabbitMQ线上故障记录-2.png 一次RabbitMQ线上故障记录 %}

### 处理办法
由于recovery.dets文件无法加载，导致RabbitMQ无法启动。为了能让系统可用，暂时先删除recovery.dets文件，然后启动RabbitMQ恢复正常。
{% asset_img 一次RabbitMQ线上故障记录-3.png 一次RabbitMQ线上故障记录 %}

## 事后分析
查看官方文档，关于RabbitMQ的数据存储：
RabbitMQ从3.7.0版本开始所有的消息数据到放在msg_stores/vhosts目录下，每个vhost单独放在一个子目录，每个子目录以 (vhost名字+哈希值).vhost命名，每个vhost分开存储。
在3.7.0以前，所有的消息都存在以下几个目录中：queues, msg_store_persistent和msg_store_transient。而且如果正常关机的话，是由recovery.dets文件来恢复元数据。 
{% asset_img 一次RabbitMQ线上故障记录-4.png 一次RabbitMQ线上故障记录 %}

下载之前太大的日志，仔细查看ERROR日志，发现如下错误：
{% asset_img 一次RabbitMQ线上故障记录-5.png 一次RabbitMQ线上故障记录 %}
{% asset_img 一次RabbitMQ线上故障记录-6.png 一次RabbitMQ线上故障记录 %}

应该是RabbitMQ在关闭时，将队列中的数据写入recovery.dets文件时由于磁盘空间不足，导致文件出现损坏（被删除的recovery.dets文件大小为0），然后RabbitMQ启动时无法从recovery.dets文件获取元数据，所以启动失败。
