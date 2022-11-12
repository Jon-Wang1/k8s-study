首先安装 Docker
```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
yum install docker-ce-20.*  docker-ce-cli-20.* -y
mkdir -p /etc/docker/
cat > /etc/docker/daemon.json <<EOF
{
  "storage-driver": "overlay2",
  "registry-mirrors": ["https://xdww4ceo.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "live-restore": true
}
EOF
systemctl daemon-reload && systemctl enable --now docker
```
#### 启动Jenkins
其中8080端口为Jenkins Web界面的端口，50000是jnlp使用的端口，后期Jenkins Slave需要使用50000端口和Jenkins主节点通信。
```shell
#创建Jenkins的数据目录，防止容器重启后数据丢失： 
mkdir /data/jenkins_data -p
chmod -R 777 /data/jenkins_data
#启动Jenkins，并配置管理员账号密码为admin/admin123：
docker run -d --name=jenkins \
  --restart=always \
  -e JENKINS_PASSWORD=admin123 \
  -e JENKINS_USERNAME=admin \
  -e JENKINS_HTTP_PORT_NUMBER=8080 \
  -p 8080:8080 \
  -p 50000:50000 \
  -v /data/jenkins_data:/bitnami/jenkins \
  harbor.ty.com/public/jenkins:2.361.2-debian-11-r5
```
#### 查看状态
有下面的信息说明启动成功：
```text
[root@jenkins ~]# docker logs -f jenkins
2022-10-21 04:02:10.203+0000 [id=20]    INFO    hudson.WebAppMain$3#run: Jenkins is fully up and running
```
之后通过Jenkins宿主机的IP 8080 即可访问 Jenkins

#### 安装插件
!注意：生产环境安装插件之前要备份插件目录，新插件有可能会引起jenkins起不来。
登录后点击Manage Jenkins -> Manage Plugins安装需要使用的插件：
在安装之前首先配置国内的插件源，点击Advanced ，将插件源更改为国内插件源
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
需要的插件列表
Git
* Git Parameter
* Git Pipeline for Blue Ocean
* GitLab
* Credentials
* Credentials Binding
* Blue Ocean
* Blue Ocean Pipeline Editor
* Blue Ocean Core JS
* Pipeline SCM API for Blue Ocean
* Dashboard for Blue Ocean
* Build With Parameters
* Dynamic Extended Choice Parameter Plug-In
* Dynamic Parameter Plug-in
* Extended Choice Parameter
* List Git Branches Parameter
* Pipeline
* Pipeline: Declarative
* Kubernetes
* Kubernetes CLI
* Kubernetes Credentials
* Image Tag Parameter
* Active Choices















