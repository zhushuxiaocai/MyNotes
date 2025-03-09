# !!!认真!!!

## 1. DVWA 搭建

##### [下载](https://github.com/digininja/DVWA)

> - 安装 PHPSTUDY

> - 在 `D:\phpstudy_pro\WWW` 下解压缩 DVWA , 访问 `127.0.0.1/DVWA` 查看报错，编辑 `D:\phpstudy_pro\WWW\DVWA\config` 下的 `config.inc.php.dist` 文件为 `config.inc.php` , 修改其中两项为 `root`
> 
> ```php
> $_DVWA[ 'db_user' ]     = getenv('DB_USER') ?: 'root';
> $_DVWA[ 'db_password' ] = getenv('DB_PASSWORD') ?: 'root';
> ```
> 
> - 刷新后可以看到 `PHP function allow_url_include: Disable`![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-08-09-25-35-image.png)
> 
> - 修改 `php7.3.4nts` 下的 `allow_url_include=OFF` 为 `On`
> 
> - 刷新后点击最下方进行创建, 默认账号密码为 `admin` `password`

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-08-09-55-41-image.png) 

## 2. SQLi-LABS 搭建

##### [下载](https://github.com/Audi-1/sqli-labs)

> - 解压缩后进入 `D:\phpstudy_pro\WWW\sqli-labs\sql-connections` 编辑 `db-creds.inc` 

> ```
> $dbpass ='root';
> ```

> - 如果还是不行 修改php版本为`php5.5.9nts`, 安装后在 网站->管理->php版本选择`php5.5.9nts` 后重启![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-08-09-49-02-image.png)
> 
> ![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-08-09-55-09-image.png)

## 3. UPLOAD-LABS

##### [下载](https://github.com/c0ny1/upload-labs)

> - 解压缩后即安装完成![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-08-09-54-38-image.png)

## 4. Docker Docker-compose

##### [docker-compose下载](https://github.com/docker/compose)

##### [VulApps下载](https://github.com/Medicean/VulApps)

#### Kali

> - `apt install docker.io -y`

> - `VulApps`中包含了 `Dockerfile` , 通过 `docker build -t NAME_IMAGE .`运行.

## 

## 5. Burp Suite

[Burp Suite Pro 2022.11.4 本体](https://portswigger.net/burp/releases/professional-community-2022-11-4)

[BurpSuite汉化](https://github.com/Leon406/BurpSuiteCN-Release)  release下选择 burpsuitloader.jar

[注册机](https://github.com/Tcilay-xi/backup/blob/main/BurpLoaderKeygen/BurpLoaderKeygen.jar)

> - 安装后将汉化包和注册包放到安装目录下![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-08-16-10-34-image.png)
> 
> - `jre\bin\java.exe --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/jdk.internal.org.objectweb.asm=ALL-UNNAMED --add-opens=java.base/jdk.inernal.org.objectweb.asm.tree=ALL-UNNAMED --add-opens=java.base/jdk.internal.org.objectweb.asm.Opcodes=ADD-UNNAMED -javaagent:BurploaderKeygen.jar -jar burpsuite_pro.jar` 
>   
>   打开激活界面
>   
>   `jre\bin\java.exe -jar BurpLoaderKeygen.jar` 
>   
>   将左侧的`License`中的内容复制到激活的 `Enter License Key` 中, 单击 `Next`,  将2中的内容复制到左侧的 `Activation Requsts` 后得到 `Activation Response` 复制到3中 

> - 汉化
> 
> - 在`BurpSuitePro.vmoptions`添加, 修改其中版本名称
>   
>   ```bash
>   --add-opens=java.desktop/javax.swing=ALL-UNNAMED
>   --add-opens=java.base/java.lang=ALL-UNNAMED
>   --add-opens=java.base/jdk.internal.org.objectweb.asm=ALL-UNNAMED
>   --add-opens=java.base/jdk.internal.org.objectweb.asm.tree=ALL-UNNAMED
>   --add-opens=java.base/jdk.internal.org.objectweb.asm.Opcodes=ALL-UNNAMED
>   -javaagent:burpsuitloader-3.7.17-all.jar=loader,han
>   -javaagent:BurpLoaderKeygen.jar
>   -Xmx2048m
>   ```
>   
>   添加浏览器拓展情景模式为 HTTP 127.0.0.1 8080 <-loopback>
>   
>   默认burp只抓http 这里要在![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-08-16-47-06-image.png)
>   
>   选择导出der格式证书, 选择保存到 受信任的??? 中 后安装证书, 重启浏览器使用bp代理即可.

  

### 6.1 TEST

```python
调整DVWA难度为low, 选择 Brute Force 
```

- 找到包后右键, 进入到重放中, 发送一次包, 右键

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-09-09-19-22-image.png)    ![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-09-09-25-24-image.png)

- 进入到 Intruder  调整参数, 导入字典, `Start`
