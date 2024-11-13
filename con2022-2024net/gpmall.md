# gpmall(有问题)

### 本篇来源于 稀土掘金 作者：斐秋 原文链接: `https://juejin.cn/post/7098247684221304868`

## 公有云应用迁移上云 gpmall 商城部署

```
使用公有云服务器，要求 centos7.5 2核8GB内存40G硬盘 192.168.1.83 ，云数据库 MySQL5.6 2核4GB/40GB 192.168.1.99，Redis 为 4.0 内存2GB 192.168.1.82, centos , mysql密码为 Root123456 , Redis 免密访问
上传软件包 `elasticsearch-8.5.2-x86_64.rpm` `gpmall-cloud.tar.gz`
```

* **云主机部署**

> 1. 登录云主机 , 修改主机名为 `gpmall`
>
> > `hostnamectl set-hostname gpmall`
>
> 2. 修改 /etc/hosts
>
> > \[root@gpmall \~]# `cat /etc/hosts`
> >
> > ```
> > ::1	localhost	localhost.localdomain	localhost6	localhost6.localdomain6
> > 127.0.0.1	localhost	localhost.localdomain	localhost4	localhost4.localdomain4
> > 127.0.0.1	gpmall	gpmall
> > ```
>
> 3. 配置 yum 源
>
> > \[root@gpmall \~]# `cat /etc/yum.repos.d/local.repo`
> >
> > ```
> > [mall]
> > name=mall
> > baseurl=file:///root/gpmall-repo
> > gpgcheck=0
> > enabled=1
> > ```
>
> 4. 安装基础服务 , java elasticsearch nginx zookeeper kafka , 将 `zookeeper-3.4.14.tar.gz` 移动到 /opt 目录下 , 将 `kafka_2.11-1.1.1.tar.gz` 移动到opt目录下
>
> > * \[root@gpmall \~]# `yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel`
> > * \[root@gpmall \~]# `java -version`
> >
> > ```
> > openjdk version "1.8.0_312"
> > OpenJDK Runtime Environment (build 1.8.0_312-b07)
> > OpenJDK 64-Bit Server VM (build 25.312-b07, mixed mode)
> > ```
> >
> > * \[root@gpmall \~]# `yum -y install elasticsearch`
> > * \[root@gpmall \~]# `yum -y install nginx`
> > * \[root@gpmall \~]# `mv zookeeper-3.4.14.tar.gz /opt/`
> > * \[root@gpmall \~]# `cd /opt/`
> > * \[root@gpmall opt]# `tar -zxvf zookeeper-3.4.14.tar.gz`
> > * \[root@gpmall conf]# `mv zoo_sample.cfg zoo.cfg`
> > * \[root@gpmall bin]# `./zkServer.sh start`
> >
> > ```
> > ZooKeeper JMX enabled by default
> > Using config: /root/zookeeper-3.4.14/bin/../conf/zoo.cfg
> > Starting zookeeper ... STARTED
> > ```
> >
> > \[root@gpmall bin]# `./zkServer.sh status`
> >
> > ```
> > ZooKeeper JMX enabled by default
> > Using config: /root/zookeeper-3.4.14/bin/../conf/zoo.cfg
> > Mode: standalone
> > ```
> >
> > \[root@gpmall \~]# `tar -zxvf kafka_2.11-1.1.1.tgz`
> >
> > \[root@gpmall bin]# `./kafka-server-start.sh -daemon ../config/server.properties`
> >
> > 使用jps或者netstat -ntpl 命令查看Kafka是否成功启动
> >
> > > \[root@gpmall bin]# jps
> >
> > ```
> > 658 WrapperSimpleApp
> > 8388 QuorumPeerMain
> > 8775 Jps
> > 8711 Kafka
> > ```
> >
> > > \[root@gpmall bin]# netstat -lntp
> >
> > ```
> > Active Internet connections (only servers)
> > Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
> > tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      895/master          
> > tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1732/sshd           
> > tcp6       0      0 ::1:25                  :::*                    LISTEN      895/master          
> > tcp6       0      0 :::43136                :::*                    LISTEN      8388/java           
> > tcp6       0      0 :::9092                 :::*                    LISTEN      8711/java           
> > tcp6       0      0 :::2181                 :::*                    LISTEN      8388/java           
> > tcp6       0      0 :::33397                :::*                    LISTEN      8711/java           
> > tcp6       0      0 :::22                   :::*                    LISTEN      1732/sshd  
> > ```
>
> 5. 登录云数据库导入 `gpmall.sql` , 密码更改为 `Root123456` , 配置 `Elasticsearch` 服务 ,
>
> > \[root@gpmall \~]# `vim /etc/elasticsearch/elasticsearch.yml`
>
> 在文件最上面加入3条语句如下:

```
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-credentials: true
```

> 将下面4条语句的注释去掉，并修改network.host的IP为本机IP

```
cluster.name: my-application
node.name: node-1
network.host: 192.168.1.183
http.port: 9200
```

> 保存退出，启动 `Elasticsearch` 服务 , 设置开机自启
>
> > \[root@gpmall \~]# `systemctl start elasticsearch`
> >
> > \[root@gpmall \~]# `systemctl enable elasticsearch`
> >
> > ```
> > Created symlink from /etc/systemd/system/multi-user.target.wants/elasticsearch.service to /usr/lib/systemd/system/elasticsearch.service.
> > ```
> >
> > 启动 nginx , 设置开机自启 \[root@gpmall \~]# `systemctl start nginx`
> >
> > \[root@gpmall \~]# `systemctl enable nginx`
> >
> > ```
> > Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service.
> > ```
>
> 6. 全局变量配置 修改 `/etc/hosts` 文件 ,

```
192.168.1.83 gpmall
192.168.1.83 kafka.mall
192.168.1.99 mysql.mall
192.168.1.82 redis.mall
192.168.1.83 zookeeper.mall
```

> > 部署前端 将 `dist` 目录下文件复制到 `Nginx` 默认路径 , 首先清空路径下的文件
> >
> > * \[root@gpmall \~]# `rm -rf /usr/share/nginx/html/*`
> > * \[root@gpmall \~]# `cp -rvf dist/* /usr/share/nginx/html/` 修改`Nginx`配置文件`/etc/nginx/nginx.conf`,添加映射 重启服务

```yaml
location / {
        root /usr/share/nginx/html;
        index index.html index.html;

        }
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

> > * \[root@gpmall \~]# `systemctl restart nginx`
>
> 部署后端 将提供的4个`JAR`包上传到`/root`目录下 并启动
>
> > * \[root@gpmall \~]# `nohup java -jar shopping-provider-0.0.1-SNAPSHOT.jar &`
> > * \[1] 19268
> > * \[root@gpmall \~]# `nohup java -jar user-provider-0.0.1-SNAPSHOT.jar &`
> > * \[2] 19312
> > * \[root@gpmall \~]# `nohup java -jar gpmall-shopping-0.0.1-SNAPSHOT.jar &`
> > * \[3] 19365
> > * \[root@gpmall \~]# `nohup java -jar gpmall-user-0.0.1-SNAPSHOT.jar &`
> > * \[4] 19423

### 可以通过 screen 创建多个窗口 , 等待所有窗口显示正常运行

* screen -S shopping //创建名为shopping的
* screen -S user
* screen -S gpmall-shopping
* screen -S gpmall-user
* screen -ls // 显示目前所有的screen窗口
* Ctrl + a d // 离开


