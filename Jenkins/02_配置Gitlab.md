vim /etc/gitlab/gitlab.rb
修改external_url地址  
如果是docker拉起的gitlab，还需要修改gitlab的gitlab_rails[‘gitlab_shell_ssh_port’]=2222并使用命令gitlab-ctl reconfigure命令重新配置
```text
##! https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
external_url 'http://gitlab.ty.com'  
```
https://blog.csdn.net/ssehs/article/details/105670046