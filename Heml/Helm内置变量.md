## Helm 内置变量
 
* Release.Name: 实例的名称， helm install 指定的名字
* Release.Namespace: 应用实例的命名空间
* Release.IsUpgrade: 如果当前对实例的操作是更新或者回滚，这个变量的值就会被置为 true
* Release.IsInstall: 如果当前对实例的操作是安装，则这边变量被置为 true
* Release.Revision: 此次修订的版本号，从 1 开始，每次升级回滚都会增加 1
* Chart:Chart.yaml 文件中的内容，可以使用 Chart.Version 表示应用版本， Chart.Name 表示 Chart 的名称