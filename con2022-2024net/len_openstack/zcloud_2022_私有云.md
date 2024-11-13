# 基于 Kubernetes 构建 CI/CD 持续集成

## 本篇来源于 哔哩哔哩 zcloudedu [点此跳转](https://www.bilibili.com/video/BV1g24y197cd?spm_id_from=333.788.player.switch&vd_source=c394d6d826c1393eb2a903f818ace5ae&p=5)

```
  名称              主机名               接口                IP              说明
                                        eth0          192.168.x.0/24        vlanx
控制节点          controller             eth1           根据题目自定义       自行创建

                                        eth0          192.168.x.0/24        vlanx
计算节点          compute                eth1           根据题目自定义       自行创建
......

```

### 基础环境配置
 * 使用提供的平台, 创建两台云实例, 使用 centos7.9 镜像 类型 4vCPU/12G/100G_50G 默认存在一张网卡 自行创建第二张网卡并连接至 controller 和 compute 节点(网段为 10.10.x.0/24 不需要创建路由) 自行检查安全组策略

### yum 源配置与 ssh 免秘钥 
 * `vi /etc/hosts` 添加
   ```shell
   controller 192.168.1.89
   compute 192.168.1.90
   ```
 * `ssh-keygen` 选项默认(此处 controller 节点配置)
   - `ssh-copy-id compute`
 * `rm -rf /etc/yum.repos.d/*`
   - `vi /etc/yum.repos.d/http.repo` (此处根据提供的进行更改)
     ```repo
     [centos7.9]
     name=centos7.9
     baseurl=http://192.168.1.250/centos7.9/
     gpgcheck=0
     enabled=1

     [iaas]
     name=iaas
     baseurl=http://192.168.1.250/iaas/iaas-repo/
     gpgcheck=0
     enabled=1
     ```
   - `yum repolist`
### 基础安装
 * `yum install openstack-iaas -y`
   - `ls /usr/local/bin/` (此处查看 sh 安装脚本)
   - `vi /etc/openstack/openrc.sh` 删去每行第一个字符# 根据题目所给配置信息填写相关内容
     ```shell
     #--------------------system Config--------------------##
     #Controller Server Manager IP. example:x.x.x.x
     HOST_IP=192.168.100.91
     #Controller HOST Password. example:000000 
     HOST_PASS=Abc@1234
     #Controller Server hostname. example:controller
     HOST_NAME=controller
     #Compute Node Manager IP. example:x.x.x.x
     HOST_IP_NODE=192.168.100.92
     #Compute HOST Password. example:000000 
     HOST_PASS_NODE=Abc@1234
     #Compute Node hostname. example:compute
     HOST_NAME_NODE=compute
     #--------------------Chrony Config-------------------##
     #Controller network segment IP.  example:x.x.0.0/16(x.x.x.0/24)
     network_segment_IP=all
     #--------------------Rabbit Config ------------------##
     #user for rabbit. example:openstack
     RABBIT_USER=openstack
     #Password for rabbit user .example:000000
     RABBIT_PASS=000000
     #--------------------MySQL Config---------------------##
     #Password for MySQL root user . exmaple:000000
     DB_PASS=000000
     #--------------------Keystone Config------------------##
     #Password for Keystore admin user. exmaple:000000
     DOMAIN_NAME=demo
     ADMIN_PASS=000000
     DEMO_PASS=000000
     #Password for Mysql keystore user. exmaple:000000
     KEYSTONE_DBPASS=000000
     #--------------------Glance Config--------------------##
     #Password for Mysql glance user. exmaple:000000
     GLANCE_DBPASS=000000
     #Password for Keystore glance user. exmaple:000000
     GLANCE_PASS=000000
     #--------------------Placement Config----------------------##
     #Password for Mysql placement user. exmaple:000000
     PLACEMENT_DBPASS=000000
     #Password for Keystore placement user. exmaple:000000
     PLACEMENT_PASS=000000
     #--------------------Nova Config----------------------##
     #Password for Mysql nova user. exmaple:000000
     NOVA_DBPASS=000000
     #Password for Keystore nova user. exmaple:000000
     NOVA_PASS=000000
     #--------------------Neutron Config-------------------##
     #Password for Mysql neutron user. exmaple:000000
     NEUTRON_DBPASS=000000
     #Password for Keystore neutron user. exmaple:000000
     NEUTRON_PASS=000000
     #metadata secret for neutron. exmaple:000000
     METADATA_SECRET=000000
     #External Network Interface. example:eth1
     INTERFACE_NAME=eth1
     #External Network The Physical Adapter. example:provider
     Physical_NAME=provider
     #First Vlan ID in VLAN RANGE for VLAN Network. exmaple:101
     minvlan=101
     #Last Vlan ID in VLAN RANGE for VLAN Network. example:200
     maxvlan=200
     #--------------------Cinder Config--------------------##
     #Password for Mysql cinder user. exmaple:000000
     CINDER_DBPASS=000000
     #Password for Keystore cinder user. exmaple:000000
     CINDER_PASS=000000
     #Cinder Block Disk. example:md126p3
     BLOCK_DISK=vda3
     #--------------------Swift Config---------------------##
     #Password for Keystore swift user. exmaple:000000
     SWIFT_PASS=000000
     #The NODE Object Disk for Swift. example:md126p4.
     OBJECT_DISK=vda4
     #The NODE IP for Swift Storage Network. example:x.x.x.x.
     STORAGE_LOCAL_NET_IP=192.168.100.91
     #--------------------Trove Config----------------------##
     #Password for Mysql trove user. exmaple:000000
     TROVE_DBPASS=000000
     #Password for Keystore trove user. exmaple:000000
     TROVE_PASS=000000
     #--------------------Heat Config----------------------##
     #Password for Mysql heat user. exmaple:000000
     HEAT_DBPASS=000000
     #Password for Keystore heat user. exmaple:000000
     HEAT_PASS=000000
     #--------------------Ceilometer Config----------------##
     #Password for Gnocchi ceilometer user. exmaple:000000
     CEILOMETER_DBPASS=000000
     #Password for Keystore ceilometer user. exmaple:000000
     CEILOMETER_PASS=000000
     #--------------------AODH Config----------------##
     #Password for Mysql AODH user. exmaple:000000
     AODH_DBPASS=000000
     #Password for Keystore AODH user. exmaple:000000
     AODH_PASS=000000
     #--------------------ZUN Config----------------##
     #Password for Mysql ZUN user. exmaple:000000
     ZUN_DBPASS=000000
     #Password for Keystore ZUN user. exmaple:000000
     ZUN_PASS=000000
     #Password for Keystore KURYR user. exmaple:000000
     KURYR_PASS=000000
     #--------------------OCTAVIA Config----------------##
     #Password for Mysql OCTAVIA user. exmaple:000000
     OCTAVIA_DBPASS=000000
     #Password for Keystore OCTAVIA user. exmaple:000000
     OCTAVIA_PASS=000000
     #--------------------Manila Config----------------##
     #Password for Mysql Manila user. exmaple:000000
     MANILA_DBPASS=000000
     #Password for Keystore Manila user. exmaple:000000
     MANILA_PASS=000000
     #The NODE Object Disk for Manila. example:md126p5.
     SHARE_DISK=vda5
     #--------------------Cloudkitty Config----------------##
     #Password for Mysql Cloudkitty user. exmaple:000000
     CLOUDKITTY_DBPASS=000000
     #Password for Keystore Cloudkitty user. exmaple:000000
     CLOUDKITTY_PASS=000000
     #--------------------Barbican Config----------------##
     #Password for Mysql Barbican user. exmaple:000000
     BARBICAN_DBPASS=000000
     #Password for Keystore Barbican user. exmaple:000000
     BARBICAN_PASS=000000
     ###############################################################
     #####在vi编辑器中执行:%s/^.\{1\}//  删除每行前1个字符(#号)#####
     ###############################################################
     ```
   - `iaas-pre-host.sh` (初始化)
     - `vi iaas-pre-host.sh` (找到 yum upgrade -y 注释)
   - `lsblk` (计算节点操作)查看compute分区, 可以是计算节点的任意分区
     - `fdisk /dev/vda`
       ```shell
       n
       Enter
       +5G
       n
       Enter
       +5G
       n
       Enter
       +5G
       
       ```
     - `partprobe` (计算节点)

### 搭建数据库 (控制节点)
  * `iaas-install-mysql.sh` (控制节点)
    - `mysql -uroot -p000000`
      - `show databases;`
      - `use mysql;`
      - `show tables;`
      - `select * from user;`
      - `select * from user\G;`
    - `vi /etc/my.cnf`
      ```shell
      #数据库支持大小写
      lower_case_table_names = 1
      #数据库缓存
      innodb_buffer_pool_size = 4G
      #数据库的log buffer即redo日志缓冲
      innodb_log_buffer_size = 64MB
      #设置数据库的redo log即redo日志大小
      innodb_log_file_size = 256MB
      #数据库的redo log文件组即redo日志的个数配置
      innodb_log_files_in_group = 2
      ```

### Keystone (控制节点)
 * `iaas-install-keystone.sh`
   - `source /etc/keystone/admin-openrc.sh`
   - `openstack user create --password 000000 chinaskill`
### Glance (控制节点)
 * `iaas-install-glance.sh`
   - `curl -O http://10.18.4.100/cirros-0.3.4-x86_64-disk.img` (下载镜像)
   - `openstack image create --min-disk 10 --min-ram 1024 --file cirros-0.3.3-x86_64-disk.img  cirros`
   - `openstack image show cirrors`
### Nova (all)
 * `iaas-install-placement.sh`  `iaas-install-nova-controller.sh` `iaas-install-nova-compute.sh`
   - `vi /etc/nova/nova.conf` (找到 vif_plugging_is_fatal)
     ```shell
     vif_plugging_is_fatal=false
     ```
   - `systemctl restart openstack-nova*`
### Neutron (all)
 * `iaas-install-neutron-controller.sh` ` iaas-install-neutron-compute.sh`
### Doshboard (控制节点)
 * `iaas-install-dashboad.sh`
   - 
