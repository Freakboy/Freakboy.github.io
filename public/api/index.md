# HTTP API文档

# HTTP API文档



### 安装镜像

使用etcher烧录镜像


**SD卡格式化工具**

https://www.sdcard.org/downloads/formatter/

### 开启ssh
使用IP scan获取ip

在/boot下新建名为ssh的空文件夹


```
默认账户pi
密码raspberry
```



**解锁root**

```
sudo passwd root
sudo passwd --unlock root
```

sudo vim /etc/ssh/sshd_config


```
PermitRootLogin yes
```


**安装omv5**

```
export http_proxy="http://192.168.1.6:10101"
export https_proxy="https://192.168.1.6:10101"

wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash

# omv-extras releases(as root)
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install | bash


```


**无显示器和路由器连接树莓派**

把WiFi共享给以太网,默认分配的ip段是192.168.137.1

在cmdline文件头中加入,设置静态ip地址


```
ip=192.168.137.11
```


```
ping raspberrypi.local

arp -a

# 网线直连电脑和树莓派
ssh root@raspberrypi
ssh root@fe80::1d07:6bcc:506d:75d5%3

```


**树莓派换源**

备份

```
# 软件更新源
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
# 系统更新源
sudo cp /etc/apt/sources.list.d/raspi.list /etc/apt/sources.list.d/raspi.list.bak

sudo nano /etc/apt/sources.list
sudo nano /etc/apt/sources.list.d/raspi.list

:%s/raspbian.raspberrypi.org/mirrors.ustc.edu.cn\/raspbian/g

:%s/archive.raspberrypi.org/mirrors.ustc.edu.cn\/archive.raspberrypi.org/g

# 中国科学技术大学(推荐使用)
http://mirrors.ustc.edu.cn/raspbian/raspbian/
http://mirrors.ustc.edu.cn/archive.raspberrypi.org/debian/

# 阿里云(Debian报无法验证公钥问题)
http://mirrors.aliyun.com/raspbian/raspbian/
http://mirrors.aliyun.com/debian/

# 清华大学
http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/
```



**树莓派设置静态IP地址**

sudo nano /etc/dhcpcd.conf
sudo systemctl restart dhcpcd.service

```
interface eth0

static ip_address=192.168.1.11/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1

interface wlan0

static ip_address=192.168.1.22/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

sudo nano /etc/network/interfaces
sudo systemctl restart networking.service

```
auto lo

iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0
iface wlan0 inet manual
address 192.168.1.22
netmask 255.255.255.0
gateway 192.168.1.1
wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
```

****


### 配置miniDLNA

安装


```
sudo apt-get install minidlna
```


编辑/etc/minidlna.conf

```
#A表示这个目录是存放音乐的，当minidlna读到配置文件时，它会自动加载这个目录下的音乐文件
media_dir=A,/media/pi/SAMSUNG/Music
media_dir=P,/media/pi/SAMSUNG/Picture
media_dir=V,/media/pi/SAMSUNG/video
#配置minidlna的数库数据的存放目录
db_dir=/var/lib/DLNA/db
#配置日志目录
log_dir=/var/log/DLNA
```

```
# 重启 minidlna
/etc/init.d/minidlna restart
/etc/init.d/minidlna status
# 开机自启minidlna
sudo update-rc.d minidlna defaults
# 启动 minidlna 服务
sudo service minidlna start
# 修改配置文件及媒体资源更新时，需要强制刷新
sudo service minidlna force-reload
# 取消 minidlna 的开机自动启动
sudo update-rc.d -f minidlna remove
# 停止 minidlna 所有进程
sudo killall minidlna
# 卸载 minidlna
sudo apt-get remove --purge minidlna
```

**报错**


```
# 1
minidlna.c:631: error: Media directory "P,/media/pi/SAMSUNG/Picture" not accessible [Permission denied]
# 2
minidlna.c:631: error: Media directory "V,/media/pi/SAMSUNG/video" not accessible [No such file or directory]
```


```
# edited /etc/default/minidlna
USER=root
GROUP=root

# then /etc/minidlna.conf
user=root

chown <user>:<group> /var/cache/minidlna
chown <user>:<group> /run/minidlna

```



**树莓派连接VNC**

默认没有开通


```
# 选 5 network->enable vnc
sudo raspi-config
```

vnc viewer 连接报错**cannot currently show the desktop**


```
# 选 7 adv->A5 设置分辨率
sudo raspi-config
```


**树莓派默认的VNC是基于用户名密码验证,需要使用vncviewer连接**


```
默认账户pi
密码raspberry
```

**使用mobaxterm的vnc连接提示`No configured security type is supported by 3.3 VNC Viewer`**

开启身份认证的VNC连接

**图形化设置**

1. 单击顶部栏中的Vnc图标，打开汉堡包菜单，然后选择“选项...”
在“安全性”选项卡下，为“身份验证”选择“ VNC密码”选项
在“用户和权限”选项卡下，选择“标准用户”，然后单击“密码...”以设置您的VNC密码
重新启动RPi，您现在应该可以使用首选的VNC客户端进行连接了！

**ssh配置**


```
sudo echo 'Authentication=VncAuth' >> /root/.vnc/config.d/vncserver-x11

# 设置vnc连接的密码
sudo vncpasswd -service

sudo reboot
```


### 旁路由访问国内网站很慢

国外网站访问很快,什么设置DNS,分开设置DNS都不行,看right论坛说,设置防火墙规则


```
# 测试可用
iptables -t nat -I POSTROUTING -j MASQUERADE

# 未测试
iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
```

**一键设置网关,IP,DNS**


```
@echo off
echo 正在修改IP地址和DNS服务器地址,请耐心等待…………
echo 正在更改本机IP地址...
netsh interface ipv4 set address name="WLAN" source=static addr=192.168.1.100 mask=255.255.255.0 gateway=192.168.1.2 gwmetric=0 >nul
echo 正在添加本机首选DNS服务器...
netsh interface ipv4 set dns name="WLAN" source=static addr=192.168.1.2 register=PRIMARY
echo 正在添加备用DNS服务器...
netsh interface ipv4 add dns name="WLAN" addr=223.5.5.5
echo 检查当前本机配置...
ipconfig /all
pause
```


dhcp方式


```
@echo off
echo 正在修改IP地址和DNS服务器地址,请耐心等待…………
echo 正在更改本机IP地址...
netsh interface ipv4 set address name="WLAN" source=dhcp
echo 正在添加本机首选DNS服务器...
netsh interface ipv4 set dns name="WLAN" source=dhcp
echo 检查当前本机配置...
ipconfig /all
pause
```


```
1.name：网络连接名称；
2.source：获取ip的途径，动态获取为dhcp，手动设置为static；
3.addr：获取的ip地址；
4.mask：子网掩码；
5.gateway：网关；
6.gwmetric：网关跃点数，可以设置为整型数值，也可以设置为“自动”:auto；
7.register：
   primary: 只在主 DNS 后缀下注册；
      none: 禁用动态 DNS 注册；
      both: 在主 DNS 后缀下注册，也在特定连接后缀下注册；
```



### WiFi配置

https://jingyan.baidu.com/article/91f5db1b9daa3e5c7f05e3e8.html

用户可以在未启动树莓派的状态下单独修改 /boot/wpa_supplicant.conf 文件配置 WiFi 的 SSID 和密码，这样树莓派启动后会自行读取 wpa_supplicant.conf 配置文件连接 WiFi 设备。

操作方法简单：将刷好 Raspbian 系统的 SD 卡用电脑读取。在 boot 分区，也就是树莓派的 /boot 目录下新建 wpa_supplicant.conf 文件，按照下面的参考格式填入内容并保存 wpa_supplicant.conf 文件。



```
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
 
network={
ssid="902"
psk="12345678900_902"
key_mgmt=WPA-PSK
priority=1
}
 
network={
ssid="z"
psk="1234567899"
key_mgmt=WPA-PSK
priority=2
scan_ssid=1
}
```

期间发现连不上我刷的华硕固件的隐藏WiFi


```
# 把这行去掉,可以连接,但是提示no set country
country=CN

# 可以连接,可以显示其他WiFi列表
country=US
```



vim /etc/wpa_supplicant/wpa_supplicant.conf

运行命令wpa_cli -i wlan0 reconfigure

使用ifconfig wlan0确认连接。


说明以及不同安全性的 WiFi 配置示例：
#ssid:网络的ssid
#psk:密码
#priority:连接优先级，数字越大优先级越高（不可以是负数）
#scan_ssid:连接隐藏WiFi时需要指定该值为1


如果你的 WiFi 没有密码


```
network={
ssid="你的无线网络名称（ssid）"
key_mgmt=NONE
}
```

如果你的 WiFi 使用WEP加密


```
network={
ssid="你的无线网络名称（ssid）"
key_mgmt=NONE
wep_key0="你的wifi密码"
}
```

如果你的 WiFi 使用WPA/WPA2加密


```
network={
ssid="你的无线网络名称（ssid）"
key_mgmt=WPA-PSK
psk="你的wifi密码"
}
```

如果你不清楚 WiFi 的加密模式，可以在安卓手机上用 root explorer 打开 /data/misc/wifi/wpa/wpa_supplicant.conf，查看 WiFi 的信息。



### HDMI连接无信号

在Windows下，进入已写入树莓派系统的SD卡，找到config.txt（最好备份一下这个）。建议，不管你是什么显示器，或者高清电视机，最好在没显示的情况，请将config.txt 中的分辨率调低一些，不要老想着，我的显示设备支持1080p，就非得一步到位。。。建议从下面的低分辨率尝试开始：
计算机显示器使用的分辨率 ：

```
hdmi_mode=4    640x480   60Hz 
hdmi_mode=9    800x600   60Hz 
hdmi_mode=16   1024x768  60Hz
```


CEA规定的电视规格分辨率。：

```
hdmi_mode=2    480p  60Hz 
hdmi_mode=4    720p  60Hz
```


另外：若直接使用的是HDMI线接显示设备，请在config.txt中添加一条：


```
hdmi_ignore_edid=0xa5000080
```

 
这个是命令树莓派不检测HDMI设备的任何信息，只按照我们指定的分辨率输出。 这样就不会自动检测显示设备的分辨率，而避免掉很多可能不显示的造成因素，就会按照你自己设置的分辨率显示。分辨率也按照上面列出的尝试，修改config.txt 中hdmi_mode=x 的”x“值。



## 4B OpenWRT


**使用无线模式中继的软路由设置**

![image-20210814212616793](https://gitee.com/gentisz/imgs/raw/master/imgs/image-20210814212616793.png)

![image-20210814212720464](https://gitee.com/gentisz/imgs/raw/master/imgs/image-20210814212720464.png)



### opkg相关

**OpenWrt镜像源**


```
# 官方源
src/gz openwrt_core https://downloads.openwrt.org/releases/21.02.0-rc4/targets/bcm27xx/bcm2711/packages
src/gz openwrt_base https://downloads.openwrt.org/releases/21.02.0-rc4/packages/aarch64_cortex-a72/base
src/gz openwrt_luci https://downloads.openwrt.org/releases/21.02.0-rc4/packages/aarch64_cortex-a72/luci
src/gz openwrt_packages https://downloads.openwrt.org/releases/21.02.0-rc4/packages/aarch64_cortex-a72/packages
src/gz openwrt_routing https://downloads.openwrt.org/releases/21.02.0-rc4/packages/aarch64_cortex-a72/routing
src/gz openwrt_telephony https://downloads.openwrt.org/releases/21.02.0-rc4/packages/aarch64_cortex-a72/telephony

# 中科大
src/gz openwrt_core https://openwrt.proxy.ustclug.org/snapshots/targets/bcm27xx/bcm2711/packages
src/gz openwrt_base https://openwrt.proxy.ustclug.org/snapshots/packages/aarch64_cortex-a72/base
src/gz openwrt_luci https://openwrt.proxy.ustclug.org/snapshots/packages/aarch64_cortex-a72/luci
src/gz openwrt_packages https://openwrt.proxy.ustclug.org/snapshots/packages/aarch64_cortex-a72/packages
src/gz openwrt_routing https://openwrt.proxy.ustclug.org/snapshots/packages/aarch64_cortex-a72/routing

# 清华源
src/gz openwrt_core https://mirrors.tuna.tsinghua.edu.cn/releases/21.02.0-rc4/targets/bcm27xx/bcm2711/packages
src/gz openwrt_base https://mirrors.tuna.tsinghua.edu.cn/releases/21.02.0-rc4/packages/aarch64_cortex-a72/base
src/gz openwrt_luci https://mirrors.tuna.tsinghua.edu.cn/releases/21.02.0-rc4/packages/aarch64_cortex-a72/luci
src/gz openwrt_packages https://mirrors.tuna.tsinghua.edu.cn/releases/21.02.0-rc4/packages/aarch64_cortex-a72/packages
src/gz openwrt_routing https://mirrors.tuna.tsinghua.edu.cn/releases/21.02.0-rc4/packages/aarch64_cortex-a72/routing
src/gz openwrt_telephony https://mirrors.tuna.tsinghua.edu.cn/releases/21.02.0-rc4/packages/aarch64_cortex-a72/telephony
```

常用命令


```
# 查看路由器的架构
opkg print-architecture | awk '{print $2}'

opkg update
opkg install luci-i18n-base-zh-cn


```

shadowsocks源


```
# 添加opkg key
wget http://openwrt-dist.sourceforge.net/openwrt-dist.pub
opkg-key add openwrt-dist.pub
opkg install wget ca-certificates ca-bundle

# vi /etc/opkg/customfeeds.conf
src/gz openwrt_dist http://openwrt-dist.sourceforge.net/packages/base/aarch64_cortex-a72
src/gz openwrt_dist_luci http://openwrt-dist.sourceforge.net/packages/luci
# 安装SHADOWSOCKS及ChinaDNS
opkg update
opkg install ChinaDNS iptables-mod-tproxy
opkg install luci-app-chinadns
opkg install shadowsocks-libev
opkg install luci-app-shadowsocks
# 生成中国IP列表
wget -O /tmp/delegated-apnic-latest 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' && awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' /tmp/delegated-apnic-latest > /etc/chinadns_chnroute.txt
# 每周更新中国IP列表
crontab -e
0 3 * * 1    wget http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest -O /tmp/delegated-apnic-latest && awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' /tmp/delegated-apnic-latest > /etc/chinadns_chnroute.txt
# 自启动
/etc/init.d/cron start
/etc/init.d/cron enable
```

**module 'luci.cbi' not found:**

```
# 删LuCI缓存
rm -rf /tmp/luci-*
opkg update
opkg install luci luci-base luci-compat
```

#### img文件选择和区别

基本上OpenWRT针对每个型号的产品都有4个文件：


```
rpi-3-ext4-factory.img.gz
rpi-3-ext4-sysupgrade.img.gz
rpi-3-squashfs-factory.img.gz
rpi-3-squashfs-sysupgrade.img.gz
```

1. 带ext4的是可以利用Linux命令把你的tf卡空余空加拿回来做其他用途，毕竟img才几十M大小而已，一般现在的tf卡都要16G起了吧；这个下次再开一篇讲解；

2. 带squashfs的相当于品牌路由器的rom，当你对自己安装的应用或配置不满意的时候，可以直接重置系统，就像回到初始状态一样；

3. 带factory的是给之前不是用OpenWRT系统的用户初始刷tf卡用的；

4. 带sysupgrade的是针对原先使用OpenWRT的用户，可以用cmd命令或者GUI界面直接升级用。

### 4B 自编译openwrt镜像




```
Image for your Device   sha256sum   File Size   Date
rpi-4-ext4-factory.img.gz   31bc61700978fa8f30eeb5e888f34b665582bb8170ac6b7be7ff0d1caa3a2ca6    14328.4 KB  Sun Aug 1 21:27:40 2021
rpi-4-ext4-sysupgrade.img.gz    68f87c294335efeb2404f11abebeb3723be699660bdb5e44369c5a9cc31cdd2d    14328.8 KB  Sun Aug 1 21:27:41 2021
rpi-4-squashfs-factory.img.gz   93df781cc17b8eb2b64ef205fd4432b1538240300d571b1662633b1b2fd632b6    12882.0 KB  Sun Aug 1 21:27:40 2021
rpi-4-squashfs-sysupgrade.img.gz    3ec10b4d8a9659cbcd157166d70c230767f86f9550fb333ef85ba205780a17ac    12882.3 KB  Sun Aug 1 21:27:40 2021
```

bin文件名称中有两种不同的格式，jffs2与 squashfs。这两种格式的固件区别在于，squashfs格式的bin文件安装后，会占用一定的空间来存放系统的一些必要文件，这些文件都只是可读的，其作用是帮助恢复系统。当OpenWrt崩溃时，可以基于这些文件，使用firstboot脚本重建初始系统，而jffs2则不会存储这样的文件，好处是节省了空间。一般使用squashfs格式的固件，方便恢复系统到初始状态。

factory与sysupgrade，这两者的区别是，factory多了一些验证的东西，用于在原厂固件的基础上进行升级，如果已经是OpenWrt，直接使用sysupgrade文件即可。并且，在原厂固件的基础上进行升级时，首先使用factory文件，然后需要再次使用 sysupgrade文件，选择不保留原来配置进行升级。



https://leux.cn/doc/OPENWRT%E7%BC%96%E8%AF%91%E4%B9%8B%E6%A0%91%E8%8E%93%E6%B4%BE4B.html

```
# 安装编译工具
sudo apt-get update
sudo apt-get install build-essential asciidoc binutils bzip2 \
gawk gettext git libncurses5-dev libz-dev patch unzip zlib1g-dev \
lib32gcc1 libc6-dev-i386 subversion flex uglifyjs libssl-dev upx \
gcc-multilib p7zip p7zip-full msmtp texinfo libglib2.0-dev xmlto \
git-core qemu-utils libelf-dev autoconf automake libtool autopoint \
curl wget device-tree-compiler
```

> 注意：不要使用root用户编译，最好准备好梯子
> 
> 编译后在openwrt/bin/targets/brcm2708/bcm2711/下找到openwrt-brcm2708-bcm2711-rpi-4-ext4-factory.img.gz，把其中的img刷入SD卡中即可



```
# 下载源码，二选一即可
mkdir openwrt
cd openwrt/
git clone https://git.openwrt.org/openwrt/openwrt.git ./  # openwrt官方源码
git clone https://github.com/coolsnowwolf/lede ./     # lean版魔改源码

# 以后每次编译前建议执行以下三行命令更新源码
git pull
./scripts/feeds update -a
./scripts/feeds install -a

make defconfig      # 测试编译环境
make menuconfig     # 配置编译参数
make download -j8 V=s   # 下载所需源码，请尽量使用梯子
make -j1 V=s        # 首次编译推荐用单线程

# 再次编译前建议使用make clean清理
make clean  # 清除bin目录
make dirclean   # 清除bin目录和交叉编译工具及工具链目录
make distclean  # 清除所有相关的东西，包括下载的软件包，配置文件，feed内容等
```

**固件编译配置**

1. 简单的make menuconfig参数配置，除必选配置外的其他项可自行选择

2. 基础配置


```
# 必选配置
Target System -> Broadcom BCM27xx
Subtarget -> BCM2711 boards (64 bit)
Target Profile -> Raspberry Pi 4B

# 镜像参数
Target Images -> ext4       # ext4格式的固件可方便地调整分区大小
Target Images -> squashfs   # squashfs格式的固件可恢复出厂设置
Target Images -> Kernel partition size = 20     # boot分区大小为20M
Target Images -> Root filesystem partition size = 500   # root分区大小为500M

# 可选工具
Base system -> block-mount  # 在LuCI界面添加<挂载点>菜单
Base system -> blockd       # 自动挂载设备
Administration -> htop      # 添加htop命令
Firmware -> xxx         # 选择你需要的网卡固件，默认即可
```

3. 内核模块

```
# 文件系统
Kernel modules -> Filesystems -> kmod-fs-ext4
Kernel modules -> Filesystems -> kmod-fs-ntfs
Kernel modules -> Filesystems -> kmod-fs-squashfs
Kernel modules -> Filesystems -> kmod-fs-vfat
Kernel modules -> Filesystems -> kmod-fuse

# 网卡支持
Kernel modules -> Network Devices -> kmod-xxx   # 有线网卡支持，默认即可
Kernel modules -> USB Support -> kmod-usb-net -> kmod-usb-net-xxx  # USB有线网卡支持，默认即可
Kernel modules -> Wireless Drivers -> kmod-xxx  # 无线网卡支持，默认即可

# USB支持
Kernel modules -> USB Support -> kmod-usb-core      # 启用USB支持
Kernel modules -> USB Support -> kmod-usb-hid       # USB键鼠支持
Kernel modules -> USB Support -> kmod-usb-storage   # 启用USB存储
Kernel modules -> USB Support -> kmod-usb-storage-extras
Kernel modules -> USB Support -> kmod-usb-usb2      # 开启USB2支持
Kernel modules -> USB Support -> kmod-usb-usb3      # 开启USB3支持
```

4. LuCI设置

```
# LuCI设置
LuCI -> Collections -> luci             # 开启luci
LuCI -> Modules -> Translations -> Chinese(zh-cn)   # 中文支持
LuCI -> Themes -> luci-theme-material           # 添加皮肤

# LuCI应用
LuCI -> Applications -> luci-app-aria2          # 下载工具
LuCI -> Applications -> luci-app-firewall       # 防 火 墙
LuCI -> Applications -> luci-app-hd-idle        # 硬盘休眠
LuCI -> Applications -> luci-app-opkg           # 软 件 包
LuCI -> Applications -> luci-app-qos            # 服务质量
LuCI -> Applications -> luci-app-samba          # 网络共享
LuCI -> Applications -> luci-app-shadowsocks-libev  # 翻墙软件
LuCI -> Applications -> luci-app-upnp           # UPnP服务
LuCI -> Applications -> luci-app-wol            # 网络唤醒
```

5. 其他设置

```
Network -> Download Manager -> ariang   # Aria2管理页面
Network -> File Transfer -> Aria2 Configuration -> ***  # 选择Aria2支持的功能
Network -> File Transfer -> curl    # 添加curl命令
Network -> File Transfer -> wget    # 添加wget命令
Utilities -> Compression -> bsdtar  # tar打包工具
Utilities -> Compression -> gzip    # GZ 压缩套件
Utilities -> Compression -> xz-utils    # XZ 压缩套件
Utilities -> Compression -> unzip   # zip解压工具
Utilities -> Compression -> zip     # zip压缩工具
Utilities -> Disc -> fdisk      # 磁盘分区工具
Utilities -> Disc -> lsblk      # 磁盘查看工具
Utilities -> Disc -> hd-idle        # 磁盘休眠
Utilities -> Disc -> hdparm     # 硬盘控制软件
Utilities -> Editors -> vim     # vim编辑器
Utilities -> Filesystem -> ntfs-3g  # NTFS读写支持
Utilities - Terminal -> screen      # 添加screen
```

6. IPv6支持

```
Global build settings -> Enable IPv6 support in packages    # 启用IPv6项
Network -> odhcp6c                      # IPv6客户端
Network -> odhcpd-ipv6only                  # IPv6服务端
Network -> Firewall -> ip6tables                # IPv6防火墙
LuCI -> Protocols -> luci-proto-ipv6                #  WebUI支持
```





### 树莓派安装centos7


```
初始账号：root
密码：centos
```

将可用分区扩展到整个SD卡
/usr/bin/rootfs-expand
df -Th
这里就可以看到扩展以后的分区了














