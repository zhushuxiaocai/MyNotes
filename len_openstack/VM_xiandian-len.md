# VM-Openstack-iaas-xiandian-len

## 本篇来源于cnblog [点此跳转](https://www.cnblogs.com/yuwen01/p/16479672.html)

```
主机要求双网卡  2核4GB 磁盘建议30+
上传 CentOS-7-x86_64-DVD-1810.iso XianDian-IaaS-v2.2.iso

主机/节点           主机名          ip                        内部通信
控制节点            controller      192.168.100.9            192.168.120.9
实例节点            compute         192.168.100.82           192.168.120.82

```

* ### 基础环境
  - 修改节点主机名
  - 配置域名解析
    `vi /etc/hosts` 添加 
    ```
    192.168.100.9 controller
    192.168.100.82 compute
    ```
  - 关闭防火墙
    ```bash
    systemctl stop firewalld && systemctl disable firewalld #关闭防火墙开机自启动

    # 清除所有 chains 链（INPUT/OUTPUT/FORWARD）中所有的rule规则。
    iptables -F

    # 清空所有chains链（INPUT/OUTPUT/FORWARD）中包及字节计数器。
    iptables -X

    # 清除用户自定义chains链（INPUT/OUTPUT/FORWARD）中的rule规则。
    iptables -Z

    # 执行清除命令没有返回结果，通过/usr/sbin/iptables-save保存。
    /usr/sbin/iptables-save
    ```

  - 关闭selinux `vi /etc/selinux/config`, 修改后需要重启才生效 可用 `setenforce 0` 临时生效
    ``` bash
    改：SELINUX=enforcing 
    为：SELINUX=disabled
    ```
    
  - 配置yum源
    
    A.  
    ```bash
    mkdir /opt/{centos,iaas}
    mount -o loop /opt/CentOS-7-x86_64-DVD-1810.iso /opt/centos/
    mount -o loop XianDian-IaaS-v2.2.iso /opt/iaas/
    ```
    B.
     `vi /etc/yum.repos.d/local.repo`
    ```bash
    
    ###controller节点
    [centos] 
    name=centos
    baseurl=file:///opt/centos 
    gpgcheck=0
    enabled=1
    [iaas] 
    name=iaas 
    baseurl=file:///opt/iaas/iass-repo 
    gpgcheck=0 
    enabled=1

    yum install vsftpd -y
    vi /etc/vsftpd/vsftpd.conf
    尾部添加一行:
    anon_root=/opt/
    保存后设置自启并启动
    systemctl enable vsftpd --now


    ###compute节点
    [centos]
    name=centos
    baseurl=ftp://192.168.200.40/centos
    gpgcheck=0
    enabled=1
    [iaas]
    name=iaas
    baseurl=ftp://192.168.200.40/mnt/iaas-repo
    gpgcheck=0
    enabled=1
   
    ```

    C.
    yum clean all && yum makecache

    D.通过 nmtui 进行基础设置 并进入配置文件进行修改
    ```yaml
    controller 节点
    vi /etc/sysconfig/network-scripts/ifcfg-eth2
    TYPE=Ethernet
    BOOTPROTO=static
    NM_CONTROLLED=yes
    DEVICE=eth0
    ONBOOT=yes      
    IPADDR=192.168.120.9
    PREFIX=24

    vi /etc/sysconfig/network-scripts/ifcfg-eth0
    TYPE=Ethernet
    BOOTPROTO=static
    NM_CONTROLLED=yes
    DEVICE=eth0
    ONBOOT=yes      
    IPADDR=192.168.100.9
    PREFIX=24


    compute节点
    vi /etc/sysconfig/network-scripts/ifcfg-eth2
    TYPE=Ethernet
    BOOTPROTO=static
    NM_CONTROLLED=yes
    DEVICE=eth0
    ONBOOT=yes      
    IPADDR=192.168.120.82
    PREFIX=24

    vi /etc/sysconfig/network-scripts/ifcfg-eth0
    TYPE=Ethernet
    BOOTPROTO=static
    NM_CONTROLLED=yes
    DEVICE=eth0
    ONBOOT=yes      
    IPADDR=192.168.100.82
    PREFIX=24
    ```

  - 安装软件包
    
    A. `yum install iaas-xiandian -y` 进行分区
    ```
    
    ```
    






















