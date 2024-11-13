# 单节点gpmall

## 本篇来源于 https://www.cnblogs.com/sh1ny2/p/14042842.html

```
上传 CentOS-7-x86_64-DVD-1511.iso , Centos-7.repo , gpmall-cloud.tar.gz , mysql-community-release-el6-5.noarch.rpm 成功后默认账号密码为test/test
```

* ### 基础环境
- > 网络配置,`ip dns gateway netmask` 不修改,记得重启
> > 1. `vi /etc/sysconfig/network-scripts/ifcfg-eth1`
> > 2. 更改`BOOTPROTO=static`

- > 修改主机名为`mall`
> > 1. `hostnamectl set-hostname mall`
> > 2. `bash`

- > 配置主机名与ip映射,关闭防火墙和selinux,选择暂时关闭
> > 1. `vi /etc/hosts` 添加 `ip_you mall`

- > 挂载镜像,配置本地yum源,备份原文件
> > 1. `mount /root/CentOS-7-x86_64-DVD-1511.iso /opt/centos/`
> > 2. `cp Centos-7.repo /etc/yum.repos.d/`
> > 3. `vi /etc/yum.repos.d/local.repo`
```yaml
[centos]
name=centos
baseurl=file:///opt/centos
gpgcheck=0
enabled=1
[gpmall-mall]
name=gpmall-mall
baseurl=file:///root/gpmall-repo
gpgcheck=0
enabled=1
```
> > 4. 清除缓存 查看yum列表

- > gpmall基础服务安装
> > 1. 安装java环境 `yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel`
> > 2. 安装redis缓存 `yum install redis -y` 若出现显示无包的情况可先安装 `EPEL` 仓库 `yum install epel-release -y`
> > 3. 安装Elasticsearch服务 `yum install elasticsearch -y` 也可不安装 若出现无包的情况可通过上传`elasticsearch-8.5.2-x86_64.rpm`进行安装
> > 4. 安装nginx服务 `yum install nginx -y`
> > 5. ~~安装MariaDB数据库 `yum install mariadb mariadb-server -y`~~
> > > - 这里安装5.6版本
> > > - `yum install mysql-community-release-el6-5.noarch.rpm -y`或者 `rpm -ivh mysql-community-release-el6-5.noarch.rpm`
> > > - `yum install mysql-community-server`

> > 6. 安装ZooKeeper服务
> > > -  `tar -xzvf gpmall-cloud/zookeeper-3.4.14.tar.gz -C /opt/`
> > > - `cd /opt/zookeeper-3.4.14/conf`
> > > - `mv zoo_sample.cfg zoo.cfg`
> > > - `cd ../bin/`
> > > - `sh zkServer.sh start`
> > 7. 安装Kafka服务
> > > - `cd /root/gpmall-cloud/`
> > > - `tar -xzvf kafka_2.11-1.1.1.tgz -C /opt/`
> > > - `cd /opt/kafka_2.11-1.1.1/bin`
> > > - `sh kafka-server-start.sh -daemon ../config/server.properties`
> > > - 通过 `jps` 或 `netstat -ntpl` 查看kafka是否启动
> > > > 安装net-tools工具包 `yum install net-tools -y`   `netstat -ntpl` 查看是否有9092端口 有则kafka成功启动

- > 配置以上服务
> > 1. 配置mariadb服务,修改数据库配置文件并启动MariaDB数据库，设置root用户密码为123456，并创建gpmall数据库，将提供的gpmall.sql导入 
> > > - `cd ~ && vi /etc/my.cnf` 在文件尾添加以下内容:
```yaml
[mysqld]

init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
```
> > > - `systemctl start mysqld`
> > > - `mysqladmin -uroot password Root123456`
> > > - `mysql -uroot -pRoot123456`
> > > - `GRANT ALL PRIVILEGES ON *.* TO 'root'@'ip-you' IDENTIFIED BY 'Root123456' WITH GRANT OPTION;`
> > > - `GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Root123456' WITH GRANT OPTION;`
> > > - `create database gpmall;`
> > > - `use gpmall;`
> > > - `source /root/gpmall.sql`
> > > - `systemctl enable mariadb` 设置mariadb开机自启
> > 2.  修改redis文件,启动redis
> > > - ` vi /etc/redis.conf` 将61行的`bind 127.0.0.1`这一行注释掉将80行的`protected-mode yes ` 改为 `protected-mode no` 找到 `requirepass ` 去掉注释,更改为`requirepass Root123456 `
> > > - `systemctl start redis && systemctl enable redis`
> > 3. 配置Elasticsearch服务,也可不配置
> > > - `vi /etc/elasticsearch/elasticsearch.yml`在文件头部加入以下内容:
```yaml
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-credentials: true
```
> > > - 将如下4条语句前的注释符'#'去掉,并修改`network.host`的IP为本机IP
```yaml
cluster.name: my-application
node.name: node-1
network.host: 192.168.100.101
http.port: 9200
```
> > > - `systemctl start elasticsearch && systemctl enable elasticsearch`
> > 4. 配置nginx服务 `systemctl start nginx && systemctl enable nginx`

- > gpmall部署
> > 1. `vi /etc/hosts`
```yaml
192.168.100.9 mall
192.168.100.9 kafka.mall
127.0.0.1 mysql.mall
192.168.100.9 redis.mall
192.168.100.9 zookeeper.mall
```
> > 2. 清除原nginx的网页文件
> > > - `rm -rf /usr/share/nginx/html/*`
> > > - `cp -rvf gpmall-cloud/dist/* /usr/share/nginx/html/`
> > > - `vi /etc/nginx/nginx.conf.default`
```
location /user {
    proxy_pass http://127.0.0.1:8082;
}
location /shopping {
    proxy_pass http://127.0.0.1:8081;
}
location /cashier {
    proxy_pass http://127.0.0.1:8083;
}
```
> > > - `systemctl restart nginx`
- > 后端部署,此处使用 screen 
> > 1. `java -jar shopping-provider-0.0.1-SNAPSHOT.jar `
> > 2. `java -jar user-provider-0.0.1-SNAPSHOT.jar `
> > 3. `java -jar gpmall-shopping-0.0.1-SNAPSHOT.jar`
> > 4. `java -jar gpmall-user-0.0.1-SNAPSHOT.jar `
