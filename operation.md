# 压力测试操作流程


### 一、压测申请表

#### 1. 压测目标
部署项目名称、压测接口 

#### 2. 压测接口和场景说明

<br/>

|  接口依赖   | 是否有经过网关鉴权 | 是否内部接口调用 | 接口有无异步任务逻辑 |是否有缓存 |缓存时间 |
|  ----  | ----  |----  |----  |----  |----  |
| 有 | 有 | 有 | 有 | 有 | 有 |
| --  | 调用内部接口获取token |响应时间变长 |-- |拿数据库/代码缓存数据返回 |--|

#### 3. 硬件环境要求（CPU、内存、磁盘类型等）
<br/>

|CPU配备|内存配备|操作系统|测试网络|
|  ----  | ----  |----  |----  |
|4核|8G|Linux|内网访问|
<br/>

>  1. 注意：环境对压力测试的影响，规避掉网络层不同因素的影响，可以考虑直接在服务器本机上跑压测程序  
>  2. ab压力测试弊端：ab是单线程程序，只能利用单一CPU，在给性能好的服务器端应用做压测时，往往跑ab的测试机负荷满了；而服务器应用的性能还绰绰有余。
比方说，应用程序反复查询、返回同一个账号的资料，跟随机查询、返回十万个用户是不一样的；前者的返回结果很容易就被数据库、应用给“缓存”掉。而对于被严重“缓存”的性能测试结果，并不能很好的反应真实场景下的性能表现。
[参考连接](https://www.zhihu.com/question/19867883)  

#### 4. 在以上硬件环境下的其他指标
<br/>

1. 并发线程数要求：等同在另外一台 4C8G Linux 内网机器上基于内网 IP 使用 wrk 执行 4 threads 100 connections
2. QPS指标：
```
hello world > 25k
redis ping > 10k
mysql ping > 5k
走 redis 缓存 > 5k
走 mysql 查询 > 1.5k
```
3. TPS指标:无
4. CPU消耗指标:99%，如果CPU达不到95%以上说明压测方发出的请求压力还不够
5. 平均响应时间指标:
6. Throughput吞吐量：取最大的那个数值
```buildoutcfg
hello world 平均约25ms 最快15ms 最大150ms
走 redis 缓存平均约60ms 最小25ms 最大250ms
走 mysql 查询平均约200ms 最小100ms 最大500ms
```

<br/>

### 二、压测服务器

IP：XXX.XX.XXX.183 （主机master机）  
IP：XXX.XX.XXX.175 （执行机salve机）

#### 1. window打开远程桌面连接 输入mstsc，服务器IP、账号密码
#### 2. 接口脚本配置-jmx ，每次生成的报告都需要删除 
- 聚合报告（.csv）:  
- 插件PerfMon Metrics Collector（q-cpu10.jmx）


|  Host/IP   | Port |Metric to collect | 
|  ----  | ----  |----  |
|  内网IP  | 默认端口4444  | CPU、Memory、Disks I/O、Network I/O|

<br/>

### 三、master机执行命令
<br/>

> 原因：
使用GUI方式启动jmeter，运行线程较多的测试时，会造成内存和CPU的大量消耗，导致客户机卡死  
> 所以正确的打开方式是在GUI模式下调整测试脚本，再用命令行模式执行    
> 注意：需要等命令执行完了才打开jmeter脚本，否则会导致执行脚本时报错
```buildoutcfg
jmeter -n -t D:\apache-jmeter-3.1\2hao\jiaoben.jmx -R IP:端口（其中IP为apache-jmeter-3.1\bin目录下，jmeter.properties文件中# Remote Hosts指定IP） -l abtest.csv -e -o ./output
```
[参考链接](https://www.cnblogs.com/kongzhongqijing/p/7216693.html)  
-n 非 GUI 模式 -> 在非 GUI 模式下运行 JMeter  
-t 测试文件 -> 要运行的 JMeter 测试脚本文件  
-l 日志文件 -> 记录结果的文件jtl/csv，此文件必须不存在  
-e 设置测试完成后生成测试报表
-o 用于指定报表生成的文件夹路径
-R 分布式中，执行机+端口，多个时用分号隔开

<br/>

### 五、数据整理
1. 将D:\apache-jmeter-3.1\2hao目录下的文件：q-cpu10.jmx、q-error.jmx剪切进命名后的文件夹中
2. 将D:\apache-jmeter-3.1\bin     目前下的文件：abtest.csv剪切进命名后的文件夹中，确保重新压测生成的文件是唯一的 