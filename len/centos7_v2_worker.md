# v2ray-worker

1. - 上传 `v2ray-linux-64.zip`
   - 安装 unzip wget screen, 将 v2ray v2ctl geoip.dat geosite.dat 复制到 /usr/bin/ 下
   - 编辑 config.json 文件, 可在win上将配置文件导出后上传

2. - `v2ray -test -config config.json` 检查配置文件是否正确
   - `v2ray –config=/etc/v2ray/config.json` 启动服务

3. - `vi /etc/profile` 后 `source /etc/profile`
   ```json
    export http_proxy=socks5://127.0.0.1:10808
​
    export https_proxy=socks5://127.0.0.1:10808
​
    export ftp_proxy=socks5://127.0.0.1:10808
​
    export no_proxy="192.168.x.x"
   ```

