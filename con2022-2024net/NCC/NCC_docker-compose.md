# Docker-Compose

> 安装软件包， 加载`CentOS_7.9.2009.tar` 镜像, `init-cluster` 初始化集群 kubernetes ，通过 `kubectl cluster-info` 查看集群状态

* ### 容器化部署 MariaDB
> - cd /root/Pig

编写 `mysql_init.sh`

```shell
#!/bin/bash
mysql_install_db --user=root
mysqld_safe --user=root &
sleep 8
mysqladmin -u root password 'root'
mysql -uroot -proot -e "grant all on *.* to 'root'@'%' identified by 'root';flush privileges;"
mysql -uroot -proot -e "source /opt/pig.sql;source /opt/pig_codegen.sql;source /opt/pig_config.sql;source /opt/pig_job.sql;"
```

_编写yum源_
> - `vi local.repo`
```yaml
[pig]
name=pig
baseurl=file:///root/yum
gpgcheck=0
enabled=1
```

编写 `Dockerfile-mariadb`

```dockerfile
FROM centos:centos7.9.2009
MAINTAINER Chinaskills
RUN rm -rf /etc/yum.repos.d/*
COPY local.repo /etc/yum.repos.d/
COPY yum /root/yum
ENV LC_ALL en_US.UTF-8
RUN yum -y install mariadb-server
COPY mysql /opt/
COPY mysql_init.sh /opt/
RUN bash /opt/mysql_init.sh
EXPOSE 3306
CMD ["mysqld_safe","--user=root"]
```

> 创建镜像 `docker build -t pig-mysql:v1.0 -f Dockerfile-mariadb .`

* ### 容器化部署 Redis

编写 `Dockerfile-redis`

```dockerfile
FROM centos:centos7.9.2009
MAINTAINER Chinaskills
RUN rm -rf /etc/yum.repos.d/*
COPY local.repo /etc/yum.repos.d/
COPY yum /root/yum
RUN yum -y install redis
RUN sed -i 's/127.0.0.1/0.0.0.0/g' /etc/redis.conf && \
     sed -i 's/protected-mode yes/protected-mode no/g' /etc/redis.conf
EXPOSE 6379
CMD ["/usr/bin/redis-server","/etc/redis.conf"]
```

> 创建镜像 `docker build -t pig-redis:v1.0 -f Dockerfile-redis .`

* ### 容器化部署 Pig

编写启动脚本 `pig_init.sh`

```bash
#!/bin/bash
sleep 20
nohup java -jar /root/pig-register.jar  $JAVA_OPTS  >/dev/null 2>&1 &
sleep 20
nohup java -jar /root/pig-gateway.jar  $JAVA_OPTS >/dev/null 2>&1 &
sleep 20
nohup java -jar /root/pig-auth.jar  $JAVA_OPTS >/dev/null 2>&1 &
sleep 20
nohup java -jar /root/pig-upms-biz.jar  $JAVA_OPTS >/dev/null 2>&1
```

编写 `Dockerfile-pig`

```dockerfile
FROM centos:centos7.9.2009
MAINTAINER Chinaskills
COPY service /root
ADD yum /root/yum
RUN rm -rfv /etc/yum.repos.d/*
COPY local.repo /etc/yum.repos.d/local.repo
RUN yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel
COPY pig_init.sh /root
RUN chmod +x /root/pig_init.sh
EXPOSE 8848 9999 3000 4000
CMD ["/bin/bash","/root/pig_init.sh"]
```

> 创建镜像 `docker build -t pig-server:v1.0 -f Dockerfile-pig .`

* ### 容器化部署 前端

编写 `Dockerfile-nginx`

```dockerfile
FROM centos:centos7.9.2009
MAINTAINER Chinaskills
RUN rm -rf /etc/yum.repos.d/*
COPY local.repo /etc/yum.repos.d/
COPY yum /root/yum
RUN yum -y install nginx
COPY nginx/dist /data
ADD nginx/pig-ui.conf /etc/nginx/conf.d/
RUN /bin/bash -c 'echo init ok'
EXPOSE 80
CMD ["nginx","-g","daemon off;"]
```

> 创建镜像 `docker build -t pig-ui:v1.0 -f Dockerfile-nginx .`

* ### 编排部署 Pig

编写 `docker-compose.yaml`

```yaml
version: '2'
services:
  pig-mysql:
    environment:
      MYSQL_ROOT_PASSWORD: root
    restart: always
    container_name: pig-mysql
    image: pig-mysql:v1.0
    ports:
      - 3306:3306
    links:
      - pig-service:pig-register
  pig-redis:
    image: pig-redis:v1.0
    ports:
      - 6379:6379
    restart: always
    container_name: pig-redis
    hostname: pig-redis
    links:
      - pig-service:pig-register
  pig-service:
    ports:
      - 8848:8848
      - 9999:9999
    restart: always
    container_name: pig-service
    hostname: pig-service
    image: pig-service:v1.0
    extra_hosts:
      - pig-register:127.0.0.1
      - pig-upms:127.0.0.1
      - pig-gateway:127.0.0.1
      - pig-auth:127.0.0.1
      - pig-hou:127.0.0.1
    stdin_open: true
    tty: true
    privileged: true
  pig-ui:
    restart: always
    container_name: pig-ui
    image: pig-ui:v1.0
    ports:
      - 8888:80
    links:
      - pig-service:pig-gateway
```

> 部署 `docker-compose up -d`

> 通过 `docker-compose ps` 查看服务状态，`master_ip:8888` 访问Pig
