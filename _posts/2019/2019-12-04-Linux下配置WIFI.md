## 准备工作
Debian系统
内核：4.5+
硬件：
* 无线网卡

软件：
* 网卡驱动
* wireless-tools
* wpasupplicant

<!--more-->

## 安装
### 驱动
这边以Linux（Debian）为例，无线网卡为`Intel Corporation Wireless-AC 9260`
查看网卡
```bash
sudo lspci -vvv
```
![](https://raw.githubusercontent.com/is0late/is0late.github.io/master/_posts/2019/media/2019-12-04-13-16-01.png)

网卡驱动一般会默认安装，如果没有按照驱动名字去官网下载

intel的网卡驱动网址
[https://www.intel.com/content/www/us/en/support/articles/000005511/network-and-io/wireless-networking.html](https://www.intel.com/content/www/us/en/support/articles/000005511/network-and-io/wireless-networking.html)

安装过程:
1. 将文件复制到特定于发行版的固件目录/ lib / firmware中。
2. 如果该目录不起作用，请参考您的发行文档。
3. 如果您自己配置内核，请确保启用了固件加载。



### 软件

软件需要安装`wireless-tools` 和 `wpasupplicant` ，采用apt源进行安装，首先要进行更新
```sh
apt-get update
```
安装
```sh
apt-get install wireless-tools
apt-get install wpasupplicant
```

如果找不到安装文件，可以尝试编译安装，参考:

[wireless-tools](https://www.linuxidc.com/Linux/2013-02/79935p2.htm)
[wpasupplicant](https://blog.csdn.net/u012503786/article/details/79541811)

具体使用方法请使用 `-h` 查看

## 配置

### 查看无线网卡

```bash
iwconfig
```
![](https://raw.githubusercontent.com/is0late/is0late.github.io/master/_posts/2019/media/2019-12-04-13-16-02.png)

### 激活无线网卡

```bash
ifconfig wlan0 up # 此处wlan0为你的网卡名称
ifconfig # 查看无线网卡
```
### 扫描附近无线热点

```bash
iwconfig wlan0 scanning
```

各类wifi信息
![](https://raw.githubusercontent.com/is0late/is0late.github.io/master/_posts/2019/media/2019-12-04-13-16-03.png)



### 连接wifi

选择合适的wifi ， 然后配置文件

```
vim /etc/wpa_supplicant/wpa_supplicant.conf # 配置wpa的连接配置文件（默认不存在）

```
填写内容如下：

```bash
network={
ssid="wifi name"
psk="wifi password"
key_mgmt=WPA-PSK  # 加密方式根据具体的wifi而定
}
```
保存退出 `:wq`

连接

```bash
wpa_supplicant -B -iwlan0 -Dnl80211 -c/etc/wpa_supplicant/wpa_supplicant.conf
```

`ifconfig` 中没有IP信息

需要DHCP自动分配一下IP

```bash
hdclient wlan0
```

配置完成之后就可以使用wifi上网

## 后记

作为Linux菜鸟，查了较多资料，页面过多就不贴参考链接，感谢各位巨人的肩膀



