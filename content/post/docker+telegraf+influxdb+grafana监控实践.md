+++
tags = []
categories = []
description = ""
date = "2017-07-21T13:20:09"
menu = ""
banner = ""
images = []
title = "docker+telegraf+influxdb+grafana监控实践"
+++

<!--more-->
服务监控是运维中必不可少的一部分，下面来教大家搭建出一款高效灵活并且颜值高的监控系统，所有服务会用docker搭建。

先简单对各个服务简单介绍以及是如何配合输出监控数据：

Telegraf > InfluxDB > Grafana 对应 采集 > 存储 > 显示

1.Telefraf
> Telefraf是Go语言编写的信息采集代理，负责把监控信息数据收集、处理、汇总和写入到指定存储中，本文中就是利用Telefraf把需要监控的数据采集到InfluxDB中。

2.InfluxDB
> InfluxDB是Go语言编写的一个开源分布式时序、事件和指标数据库，负责收集监控数据，提供了类似sql的数据库语句帮助方便查询要展示的数据。

3.Grafana
> Grafana是一个开源的指标量监测和可视化工具。常用于展示基础设施的时序数据和应用程序运行分析。在本示例中用于展示InfluxDB中的数据。

### 安装
下面安装过程都是基于docker，关于docker安装过程请大家自行百度</br>

**注意**</br>
docker实例中的时区会差8个小时别忘记改时区，先进入docker实例，在实例中执行。
```
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \ 
    echo 'Asia/Shanghai' >/etc/timezone
```
## InfluxDB（https://www.influxdata.com/）
安装：
```
docker run --name study-grafana-influxdb -p 8083:8083 -p 2003:2003 -p 8086:8086 -v /home/data/dockerwork/study-grafana-influxdb/data:/var/lib/influxdb -d influxdb
```
用docker启动我们的InfluxDB，并且映射出8083 2003 8086端口，同时把influxdb中的数据文件映射到本机存储。

创建空数据库：
```
curl -i -XPOST http://localhost:8086/query --data-urlencode "q=CREATE DATABASE telegraf"
```
当然也可以用客户端操作，文档地址：https://docs.influxdata.com/influxdb/v1.3/introduction/getting_started/

这样我们的数据库是算搭建完成了。

## Grafana （https://github.com/grafana/grafana）
安装：
```
docker run --name study-grafana -v /home/data/dockerwork/study-grafana:/var/lib/grafana -p 3000:3000 -d grafana/grafana:latest
```
映射出3000端口，并且把需要存在本地的数据盘从docker中映射给本地

配置：

1.登录我们的Grafana：http://localhost:3000/，默认帐号密码admin,admin。

2.进入Data Sources 点击 Add data source
{{< gallery "/images/5E697DE2-CECF-44E9-85F3-FCE3528A2F8D.png" "/images/5E697DE2-CECF-44E9-85F3-FCE3528A2F8D.png" "/images/5E697DE2-CECF-44E9-85F3-FCE3528A2F8D.png" >}}

3.设置数据库用于采集展示
{{< gallery "/images/C759A64A-19B8-4608-9014-74F8DF8CB030.png" "/images/C759A64A-19B8-4608-9014-74F8DF8CB030.png" "/images/C759A64A-19B8-4608-9014-74F8DF8CB030.png" >}}
如上图设置点击保存

Grafana 已经配置完一半了，虽然连接了数据库但是我们还没有要展示的数据，所以我们下一步要安装Telefraf采集数据，到InfluxDB中，最后通过Grafana展示采集过来的数据。

## Telefraf（https://github.com/influxdata/telegraf）

1.下载
https://github.com/influxdata/telegraf/releases

2.安装

`yum localinstall telegraf-1.3.4-1.x86_64.rpm`

3.配置

`vi /etc/telegraf/telegraf.conf`

找到  [[outputs.influxdb]] 标记修改urls以及database，如下：

```
[[outputs.influxdb]]
  ## The HTTP or UDP URL for your InfluxDB instance.  Each item should be
  ## of the form:
  ##   scheme "://" host [ ":" port]
  ##
  ## Multiple urls can be specified as part of the same cluster,
  ## this means that only ONE of the urls will be written to each interval.
  # urls = ["udp://localhost:8089"] # UDP endpoint example
  urls = ["http://localhost:8086"] # required
  ## The target database for metrics (telegraf will create it if not exists).
  database = "telegraf" # required
```
因为telegraf默认是开启了一些基础监控的 比如cpu 内存等
所以我们暂不配置其他监控采集信息，配置完数据库配置直接保存:wq。

4.启动
`
systemctl start telegraf
`

至此我们的Telefraf安装以及Telefraf基础配置已经完成


## 通过Grafana展示监控数据
之前我们已经配置了Telefraf并且采集到了InfluxDB中，接下来需要完成我们的最后一步工作，通过Grafana展示数据。

1.增加一个新的dashboard，并选择Graph

2.配置我们的采集数据
{{< gallery "/images/4CAFDD34-76E5-4829-A2B2-7A55ADA52716.png" "/images/4CAFDD34-76E5-4829-A2B2-7A55ADA52716.png" "/images/4CAFDD34-76E5-4829-A2B2-7A55ADA52716.png" >}}
上图配置的采集系统的负载信息，具体的其他配置请大家参考Metrics配置的用法以及Telefraf的github官网参考可以采集的数据。

最终上一张效果图
{{< gallery "/images/D1DCE6D6-A003-48F1-8840-FA34B0149893.png" "/images/D1DCE6D6-A003-48F1-8840-FA34B0149893.png" "/images/D1DCE6D6-A003-48F1-8840-FA34B0149893.png" >}}