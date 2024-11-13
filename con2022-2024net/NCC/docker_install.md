# docker

* #### 基础准备

> `docker-repo` `Centos-7.repo` 上传
- 关闭防火墙 SElinux
- 清空 `/etc/yum.repos.d/` 下的所有文件,将`Centos-7.repo`移至该目录下,在该目录下编写 `local.repo` 文件,内容为:
```repo
[docker]
name=docker
baseurl=file:///root/docker-repo
gpgcheck=0
enabled=1
```
- 完成后 建议先清除缓存 再进行安装
- `yum install docker-ce -y` 进行安装
- 若出现此错误可通过 `yum update audit audit-libs` 更新 `audit` 和 `audit-libs` 包解决
```
Error: Package: audit-2.7.6-3.el7.x86_64 (@anaconda)
           Requires: audit-libs(x86-64) = 2.7.6-3.el7
           Removing: audit-libs-2.7.6-3.el7.x86_64 (@anaconda)
               audit-libs(x86-64) = 2.7.6-3.el7
           Updated By: audit-libs-2.8.1-3.el7.x86_64 (docker)
               audit-libs(x86-64) = 2.8.1-3.el7
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
```
-最后启动`docker`并设置开机自启即可`systemctl start docker && systemctl enable docker`
