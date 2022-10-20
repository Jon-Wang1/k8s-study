## 安装calico

### 以下步骤只在master01执行
```shell
cd ~
git clone https://github.com/dotbalo/k8s-ha-install.git
cd /root/k8s-ha-install 
git checkout manual-installation-v1.23.x 
cd calico/

```

	
### 修改calico配置文件Pod网段POD_CIDR字段，写成自己的pod网段。
```shell
cd /root/k8s-ha-install/calico/
sed -i "s#POD_CIDR#172.16.0.0/12#g" calico.yaml
kubectl apply -f calico.yaml

```
