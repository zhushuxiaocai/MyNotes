# !!!认真!!!



## DVWA 搭建

```
[DVWA下载](https://github.com/digininja/DVWA)
```

> - 安装 PHPSTUDY

> - 在 `D:\phpstudy_pro\WWW` 下解压缩 DVWA , 访问 `127.0.0.1/DVWA` 查看报错，编辑 `D:\phpstudy_pro\WWW\DVWA\config` 下的 `config.inc.php.dist` 文件为 `config.inc.php` , 修改其中两项为 `root`
> 
> ```php
> $_DVWA[ 'db_user' ]     = getenv('DB_USER') ?: 'root';
> $_DVWA[ 'db_password' ] = getenv('DB_PASSWORD') ?: 'root';
> ```
> 
> - 刷新后可以看到 `PHP function allow_url_include: Disable` ，这里在![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-08-09-25-35-image.png)
> 
> - 修改 `php7.3.4nts` 下的 `allow_url_include:OFF` 为 `On`
> 
> - 刷新后点击最下方进行创建, 默认账号密码为 `admin` `password`
> 
> 

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-08-09-55-41-image.png) 



## SQLi-LABS 搭建

```
[SQLI-LABS下载](https://github.com/Audi-1/sqli-labs)
```

> - 解压缩后进入 `D:\phpstudy_pro\WWW\sqli-labs\sql-connections` 编辑 `db-creds.inc` 

> ```
> $dbpass ='root';
> ```

> - 如果还是不行 修改php版本为, 安装后在 网站->管理->php版本选择`php5.5.9nts` 后重启![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-08-09-49-02-image.png)
> 
> ![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-08-09-55-09-image.png)



## UPLOAD-LABS

```
[UPLOAD-LABS下载](https://github.com/c0ny1/upload-labs)
```

> - 解压缩后即安装完成![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-03-08-09-54-38-image.png)


