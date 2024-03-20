# Prometheus 日常运维 —— 清理旧数据


> Prometheus怎么清理旧数据 ~</br>

<!--more-->

## 清理意义
Prometheus是一个开源的监控系统，它可以帮助用户收集和存储大量的时间序列数据。当监控系统收集的数据变得过于庞大时，就需要清理旧数据以释放存储空间。

## 两种方式
### 配置方式
Prometheus可以通过设置retention参数来自动清理旧数据。这个参数指定了数据保留的时间范围，超过这个时间范围的数据将被删除。可以通过修改Prometheus的配置文件来设置retention参数。
```
--storage.tsdb.path：Prometheus写入数据库的位置。默认为data/。
--storage.tsdb.retention.time：何时删除旧数据。默认为15d。storage.tsdb.retention如果此标志设置为默认值以外的任何值，则覆盖。
--storage.tsdb.retention.size：[EXPERIMENTAL]要保留的最大存储块字节数。最旧的数据将首先被删除。默认为0或禁用。该标志是试验性的，将来的发行版中可能会更改。支持的单位：B，KB，MB，GB，TB，PB，EB。例如：“ 512MB”
--storage.tsdb.retention：不推荐使用storage.tsdb.retention.time。
--storage.tsdb.wal-compression：启用压缩预写日志（WAL）。根据您的数据，您可以预期WAL大小将减少一半，而额外的CPU负载却很少。该标志在2.11.0中引入，默认情况下在2.20.0中启用。请注意，一旦启用，将Prometheus降级到2.11.0以下的版本将需要删除WAL。
```

### 手动方式
如果需要手动清理旧数据，可以使用Prometheus的TSDB Admin API来删除时间序列数据。可以通过查询接口获取要删除的时间序列数据，然后调用API删除这些数据。

#### Prometheus TSDB Admin API
当前Prometheus TSDB Admin API提供了三个接口，分别是快照(Snapshot), 数据删除(Delete Series), 数据清理(Clean Tombstones)。 下面介绍数据删除和数据清理。

Prometheus的TSDB Admin API默认是关闭的，需要加入启动参数--web.enable-admin-api才会启动。

开启TSDB Admin API后，可以使用下面的API删除metrics数据:
```json
PUT /api/v1/admin/tsdb/delete_series
```
这个接口有以下三个URL Query参数:
- match[]=<series_selector> : Metrics的名称
- start=<rfc3339 | unix_timestamp> : 开始的时间戳
- end=<rfc3339 | unix_timestamp> : 结束的时间戳

例如:
```json
# 删除匹配数据
curl -X POST -g 'http://xxx.com/api/v1/admin/tsdb/delete_series?match[]={wanip="10.244.2.158:9090"}' 

# 删除所有数据
curl -X POST -g 'http://xxx.com/api/v1/admin/tsdb/delete_series?match[]={__name__=~".+"}' 

# 删除指定的Metric
curl -X POST -g 'http://127.0.0.1:9090/api/v1/admin/tsdb/delete_series?match[]=node_cpu_seconds_total'

# 删除指定 Metric 名称和特定 label 名称的全部数据
curl -X POST -g 'http://127.0.0.1:9090/api/v1/admin/tsdb/delete_series?match[]=node_cpu_seconds_total{mode="idle"}'

#删除指定时间范围内的 Metric 数据
curl -X POST -g 'http://127.0.0.1:9090/api/v1/admin/tsdb/delete_series?start=1578301194&end=1578301694&match[]=node_cpu_seconds_total{mode="idle"}'
```
需要注意使用数据删除接口将metric数据删除后，只是将数据标记为删除，实际的数据(tombstones)仍然存在于磁盘上，Prometheus发生Block压缩时或者调用clean_tombstones接口时才会真正删除数据。

clean_tombstones手动数据清理接口十分简单，不需要参数:
```json
curl -X POST http://127.0.0.1:9090/api/v1/admin/tsdb/clean_tombstones
```

## 参考
https://prometheus.io/docs/prometheus/latest/querying/api/#tsdb-admin-apis</br>
https://www.yisu.com/ask/83355172.html</br>
https://blog.frognew.com/2021/08/how-to-delete-prometheus-metrics.html</br>
https://blog.csdn.net/w345731923/article/details/113643866</br>
