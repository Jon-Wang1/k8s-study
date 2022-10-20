up{job="node exporter"} 属于绝对匹配，PromQL也支持如下表达式：
➢ !=：不等于
➢ =~：表示等于符合正则表达式的指标
➢ !~：和=~类似，=~表示正则匹配, !~表示正则不匹配。
如果想要查看主机监控的指标有哪些,可以输入node,会提示所有主机监控的指标
假如想要查询Kubernetes集群中每个宿主机的磁盘总量，可以使用
node_filesystem_size_bytes
node_filesystem_size_bytes{mountpoint="/"}
或者是查询分区不是/boot,且磁盘是/dev/开头的分区大小
node_filesystem_size_bytes{device=~"/dev/.*", mountpoint!="/boot"}
查询主机master01在最近 5 分钟可用的磁盘空间变化：
node_filesystem_avail_bytes{instance="master01.ty.com",mountpoint="/",device="/dev/mapper/centos-root"}[5m]
目前支持的范围单位如下：
➢ s 秒
➢ m 分钟
➢ h 小时
➢ d 天
➢ w 周
➢ y 年

查询10分钟之前磁盘可用空间,只需要指定 offset 参数即可
node_filesystem_avail_bytes{instance="k8s master01", mountpoint="/",device="/dev/mapper/centos-root"} offset 10m
查询10分钟之前，5分钟区间的磁盘可用空间的变化： 
node_filesystem_avail_bytes{instance="k8s master01", mountpoint="/",device="/dev/mapper/centos root"}[5m] offset 10m



