centos7.6配置V2ray作为客户端

# linux下配置V2ray作为客户端来访问GitHub等服务

### 1、下载安装包

+   从 [https://github.com/v2ray/v2ray-core/releases/tag/v4.28.2](https://github.com/v2ray/v2ray-core/releases/tag) 下载 [v2ray](https://www.witersen.com/?tag=v2ray "v2ray")\-linux-64.zip 文件并解压
+   (选择适合自己设备的即可)



### 2、配置config.json文件

+   因为我们的v2ray此时是用来作为客户端工作 所以需要配置节点
+   节点的配置文件为 config.json
+   节点文件可以从v2ray的Windows客户端导出
+   检查配置文件 成功效果如下

```
[root@controller v2ray]# v2ray -test -config config.json
V2Ray 4.28.2 (V2Fly, a community-driven edition of V2Ray.) Custom (go1.15.2 linux/amd64)
A unified platform for anti-censorship.
2025/02/01 12:00:56 [Info] v2ray.com/core/common/platform/ctlcmd: <v2ctl message> 
v2ctl> Read config:  config.json
Configuration OK.
```

### 3、运行


+   在 Linux 中，配置文件通常位于 /etc/v2ray/config.json 文件(非压缩包安装)。运行 v2ray –config=/etc/v2ray/config.json，或使用 systemd 等工具把 V2Ray 作为服务在后台运行
+   这里偷懒我用screen再开个终端,

### 4、测试

+   curl -x socks5://127.0.0.1:10808 [https://www.google.com](https://www.google.com)

### 5、配置curl、wget等命令使用代理

#### 方法一（推荐）

+   修改文件“/etc/profile”，在文件结束位置增加如下内容：

```
# 设置http代理
export http_proxy=socks5://127.0.0.1:10808
​
# 设置https代理
export https_proxy=socks5://127.0.0.1:10808
​
# 设置ftp代理
export ftp_proxy=socks5://127.0.0.1:10808
​
# 192.168.x.x为我们自己的云服务器的内网IP 配置为no_proxy代表内网传输不走代理
export no_proxy="192.168.x.x" 
```

+   修改后重启服务器 上面的设置才会生效
+   或者不重启服务器 执行以下命令 使配置立即生效

```
source /etc/profile
```

#### 方法二（在当前终端临时生效）

+   使代理生效

```
export http_proxy=socks5://127.0.0.1:10808
export https_proxy=socks5://127.0.0.1:10808
export ftp_proxy=socks5://127.0.0.1:10808
export no_proxy="192.168.x.x" 
```

+   使代理失效

```
unset http_proxy
unset https_proxy
unset ftp_proxy
unset no_proxy
```

#### 方法三（针对单用户生效）

+   编辑文件 vim ~/.bashrc 添加以下内容

```
# set proxy
function setproxy() {
    export http_proxy=socks5://127.0.0.1:10808
    export https_proxy=socks5://127.0.0.1:10808
    export ftp_proxy=socks5://127.0.0.1:10808
    export no_proxy="192.168.x.x" 
}
​
# unset proxy
function unsetproxy() {
    unset http_proxy https_proxy ftp_proxy no_proxy
}
```

+   保存退出
+   执行 source ~/.bashrc ,使得配置立即生效;
+   或是关闭当前终端，重新打开，使得配置立即生效;
+   在终端执行 setproxy 使代理生效
+   在终端执行 unsetproxy 使代理生效
