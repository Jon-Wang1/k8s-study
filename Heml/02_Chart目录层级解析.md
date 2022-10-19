创建一个Chart：
```shell
helm create helm-test

```
.
├── charts # 依赖文件
├── Chart.yaml #当前chart的基本信息
│       apiVersion: Chart的apiVerion，目前默认都是V2
│       name: Chart的名称
│       type: 图标的类型[可选]
│       version: # Chart自己的版本号
│       appVersion: # Chart内应用的版本号[可选]
│       description: # Chart描述信息[可选]
├── templates # 模板位置
│      ├── deployment.yaml
│      ├── _helpers.tpl # 自定义的模板或函数
│      ├── hpa.yaml # 水平自动扩展[可选]
│      ├── ingress.yaml # ingress配置
│      ├── NOTES.txt  # Chart安装完成后的提示信息
│      ├── serviceaccount.yaml 
│      ├── service.yaml
│      └── tests # 测试文件
│          └── test-connection.yaml
└── values.yaml # 配置全局变量或一些参数