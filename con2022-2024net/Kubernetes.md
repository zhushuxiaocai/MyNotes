# Kubernetes

### 本篇来源于

Centos7.5最小化安装后基础环境
```
建议三台centos , 2核2G起 , master worker1 worker2 ,
配置 IP GTEWAY DNS ,配置 yum 源 curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

* ****

> 1. 更新并安装依赖(所有)
> > - `yum update -y`
> > - `yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp`

> 2. 安装 docker (所有)
> > 2.1 安装依赖
> > > `yum install yum-utils  device-mapper-persistent-data lvm2 -y`
> > > 
> > 2.2 设置 docker 仓库
> > > - `yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`
> > > - `mkdir -p /etc/docker`
> > > - `vi /etc/docker/daemon.json `
```
{
  "registry-mirrors": ["IP"]
}
```
> > > - `systemctl daemon-reload`
> > > 
> > 2.3 安装 docker
> > > -  yum install docker-ce-18.09.0 docker-ce-cli-18.09.0 containerd.io -y
> > >   
> > 2.4 启动 docker
> > > - systemctl start docker && systemctl enable docker
> 3. 修改 hosts 文件
> > 3.1 master
> > > - `hostnamectl set-hostname master`
> > > - `vi /etc/hosts`
```yaml
192.168.100.10 master
192.168.100.11 worker1
192.168.100.12 worker2
```
> > 3.2 worker
> > > - `hostnamectl set-hostname worker1`
> > > - `hostnamectl set-hostname worker2`
> > > - `vi /etc/hosts`
```yaml
192.168.100.10 master
192.168.100.11 worker1
192.168.100.12 worker2
```
> > 3.3 三台机子 ping 通

> 4. 系统基础配置
> > 4.1 关闭防火墙 SELinux swap
> > > - `systemctl stop firewalld && systemctl disable firewalld`
> > > - `setenforce 0`
> > > - `sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config`
> > > - `swapoff -a`
> > > - `sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab`
> > 4.2 配置iptables的ACCEPT规则
> > >  - `iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat && iptables -P FORWARD ACCEPT`
> > 4.3 设置系统参数
> > > - `vi /etc/sysctl.d/k8s.conf`
```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```
> > > - `sysctl --system`






