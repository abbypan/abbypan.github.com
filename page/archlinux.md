---
layout: page
title: "Archlinux"
tagline: 安装笔记
---
{% include JB/setup %}

## 基础系统
- 下载archlinux最新iso文件：http://www.archlinux.org/download/
- 可以制作成光盘，或U盘启动


### U盘启动
- 下载LinuxLive USB Creator：http://www.linuxliveusb.com/
- 用LinuxLive USB Creator制作可启动的USB盘
- 修改BIOS，从USB启动，进入archlinux


### 设置arch源
- 编辑/etc/pacman.conf

       [archlinuxfr]
       SigLevel = Optional TrustAll
       Server = http://repo.archlinux.fr/$arch

- 编辑/etc/pacman.d/mirrorlist

        Server = http://mirrors.163.com/archlinux/$repo/os/$arch

### 硬盘分区
- 假设系统硬盘为/dev/sda，这边是SSD
- 执行fdisk /dev/sda进行分区，假设新建的系统分区为/dev/sda1
- ``mkfs -t ext4 -b 4096 -E stride=128,stripe-width=128 /dev/sda1``


### 安装系统
- 执行``wifi-menu``，连接合适的无线网络
- 编辑/etc/pacman.d/mirrorlist，选择合适的server，比如163.com的源就比较快

{% highlight bash %}
mount /dev/sda1 /mnt
pacstrap /mnt base base-devel
pacstrap /mnt grub2-bios
genfstab -p /mnt >> /mnt/etc/fstab
arch-chroot /mnt
{% endhighlight %}

- 设置arch源，见上节
{% highlight bash %}
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
mkinitcpio -p linux
grub-install --no-floppy /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
pacman -S net-tools wpa_actiond wireless_tools wpa_supplicant ifplugd dialog
exit
reboot
{% endhighlight %}

- 重启之后，执行``wifi-menu``连接到无线网络


### 系统更新
{% highlight bash %}
      pacman -Syu
      pacman -S yaourt
{% endhighlight %}

### pacman/yaourt调用axel多线程下载文件

假设同时开8个连接

在/etc/pacman.conf中指定

``XferCommand = /usr/bin/axel -n 8 -o %o %u``

在/etc/makepkg.conf中指定DLAGENTS

          'http::/usr/bin/axel -n 8 -o %o %u'
          'https::/usr/bin/axel -n 8 -o %o %u'
          'ftp::/usr/bin/axel -n 8 -o %o %u'

## 系统配置


### 图形界面

安装X 

``yaourt -S xorg xorg-xinit consolekit``

安装XFCE

``yaourt -S xfce4 xfce4-notifyd``

- 进入X的配置，不然关机键老是灰的：编辑~/.xinitrc

        exec ck-launch-session dbus-launch startxfce4


### 声卡
- 安装：``yaourt -S gstreamer0.10 gstreamer0.10-base-plugins``
- 配置：[ArchWiki:设置ALSA](https://wiki.archlinux.org/index.php/ALSA_%E5%AE%89%E8%A3%85%E8%AE%BE%E7%BD%AE_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29)
- [archlinux音量太小的问题解决](https://bbs.archlinux.org/viewtopic.php?pid=1090109)

{% highlight bash %}
# pacman -Sy alsa-lib alsa-utils alsa-oss
# gpasswd -a USERNAME audio
# alsaconf
# alsamixer
# alsactl store
{% endhighlight %}

编辑 ``/etc/rc.conf`` 文件，添加 ``alsa`` 到DAEMONS行。

编辑``/etc/modprobe.d/alsa-base``文件，添加以下两行：

        options snd-usb-audio index=0
        options snd-hda-intel index=1

### 输入法(scim)
- 安装：``yaourt -S ibus ibus-table-zhengma ibus-pinyin``
- 配置：[IBus](https://wiki.archlinux.org/index.php/IBus_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29)


### 输入法(fcitx)
- 安装：``yaourt -S fcitx fcitx-sunpinyin fcitx-table-extra``


### 中文环境
- ``vim /etc/locale.gen``，指定zh_CN.UTF-8
- ``vim /etc/local.conf``

      LANG=zh_CN.UTF-8
      LC_MESSAGES=zh_CN.UTF-8

- 执行locale-gen
- vim /etc/rc.conf

      LOCALE=zh_CN.UTF-8


### 时间
{% highlight bash %}
      yaourt -S ntp
      ntpdate asia.pool.ntp.org
{% endhighlight %}

### 热插拔(xfce4)
- ``yaourt -S ntfs-3g thunar-volman udisks``
- ``yaourt -S gvfs gvfs-afc gvfs-gphoto2 gvfs-mtp``
- 配置 /etc/fstab，手动挂载磁盘

      /dev/sdb1 /mnt/usb ntfs-3g noauto,users,permissions 0 0


### 常用软件

| 类型 | 软件 |
| ---- | ---- |
| 下载 | rsync curl lftp wget axel
| 字体 | wqy-bitmapfont wqy-zenhei ttf-monaco
| 视频 | smplayer flashplugin ffmpeg flashplayer
| 办公 | libreoffice-zh-CN libreoffice-impress libreoffice-writer libreoffice-calc 
| 解压 | unzip
| 浏览器 | firefox firefox-i18n-zh-cn
| 网络 | dnsutils traceroute wireshark-gtk


## 网络

### 根据MAC地址固定网卡名称

编辑文件/etc/udev/rules.d/10-network.rules

      SUBSYSTEM=="net", ATTR{address}=="00:26:2d:f6:ad:43", NAME="eth0"
      SUBSYSTEM=="net", ATTR{address}=="70:f1:a1:28:5a:ad", NAME="wlan0"

### 无线(netctl)
- 安装: ``yaourt -S net-tools wireless_tools wpa_supplicant netctl``
- 配置: 参考/etc/netctl/examples/

新建一个/etc/netctl/athome配置(wpa)

      CONNECTION='wireless'
      DESCRIPTION='athome'
      INTERFACE='wlan0'
      SECURITY='wpa'
      ESSID=athome
      IP='dhcp'
      KEY=athomepasswd


新建一个/etc/netctl/atwork配置(wep)

      CONNECTION='wireless'
      DESCRIPTION='atwork'
      INTERFACE='wlan0'
      SECURITY='wep'
      ESSID=atwork
      IP='dhcp'
      KEY="s:myatworkpasswd"

- 开机启动

      ``netctl enable athome``

- 手工启动

      ``netctl start athome``

### 无线(wpa_supplicant)
- 安装: ``yaourt -S net-tools wireless_tools wpa_supplicant``
- 配置:
假设配置ESSID为mywireless，密码为mypasswd的无线

      wpa_passphrase mywireless mypasswd >> /etc/wpa_supplicant.conf

手动修改/etc/wpa_supplicant.conf

      update_config=1
      ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=wheel
      ap_scan=1
      fast_reauth=1
      
      network={
      ssid="mywireless"
      
      #proto=WPA
      proto=RSN
      key_mgmt=WPA-PSK
      pairwise=CCMP TKIP
      group=CCMP TKIP
      
      #psk="mypasswd"
      psk=09896d6dc939e1d6b279c10ee3d4d1c8c75970ce345c6552b7ee47d892f0740e
      }

- 手动连接：假设无线网卡为wlan0

{% highlight bash %}
WLAN=wlan0
ESSID=mywireless
PASSWD=mypasswd

rm /run/dhcpcd-$WLAN.pid
rm /var/run/wpa_supplicant/$WLAN

ifconfig $WLAN up

wpa_supplicant -dd -B -Dwext -i $WLAN -c /etc/wpa_supplicant.conf

ifconfig $WLAN up
iwconfig $WLAN essid $ESSID key "s:$PASSWD"
dhcpcd $WLAN
{% endhighlight %}


### vpn
- 参考：[archlinux pptp vpn拨号连接](http://blog.vkill.net/read.php?97)
- 没看到arch下有/etc/ppp/ip-up.d目录，用以下脚本来启动vpn，我没有把它加为开机启动项，嗯。

{% highlight bash %}
#!/bin/zsh

#取网关地址
gateway=`route|grep default|grep eth0|awk '{print $2;}'`
vpn_gateway="192.168.6.253"

echo "拨号..."
sudo poff -a
sleep 2
sudo pon lab
sleep 3

echo "修改路由..."
#科大的地址不从vpn走
sudo route add -net 202.38.0.0/16 gw $gateway eth0
sudo route add -net 210.45.0.0/16 gw $gateway eth0
sudo route add -net 211.86.0.0/16 gw $gateway eth0

#默认从vpn走
sudo route del default
sudo route add default gw $vpn_gateway dev ppp0

#看路由
sudo route -n
{% endhighlight %}

###  netctl提示wpa无线连接失败，要看 journal -xn等等

``netctl start somewireless``

netctl 提示wpa无线连接失败，要看journal -xn等等

可以先禁用对应的网卡，再重新start，例如：

``ip link set wlan0 down``

``netctl start somewireless``

## 其它

### NGINX+PHP
- ``yaourt -S nginx spawn-fcgi php-cgi``
- 以http(用户):http(组)启动fastcgi : 
``spawn-fcgi -a 127.0.0.1 -p 9000 -C 5 -u http -g http -f /usr/bin/php-cgi``
- 配置/etc/nginx/conf/nginx.conf

      location / {
      root   /var/www;
      index  index.php index.html index.htm;
      }
      
      location ~ \.php$ {
      root           /var/www;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
      include        fastcgi_params;
      }

- 启动nginx: /etc/rc.d/nginx start


### KERNEL PANIC 恢复

- 系统升级失败，重启提示kernel panic，switch_root : fail to ...
- 从live cd启动，将原来系统的根分区挂载到/mnt下，再用旧版glibc恢复之

{% highlight bash %}
mount /dev/sda1 /mnt
yaourt -U glibc-2.16.0-1-x86_64.pkg.tar.xz -r /mnt
{% endhighlight %}

## 硬件

### 禁用 wifi 键盘灯 LED 闪烁

见：[wireless led blink](https://wiki.archlinux.org/index.php/Wireless_Setup_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29#.E7.A6.81.E7.94.A8_LED_.E9.97.AA.E7.83.81)

{% highlight bash %}
# echo 'options iwlwifi led_mode=1' >> /etc/modprobe.d/wlan.conf
# modprobe -r iwlwifi && modprobe iwlwifi
{% endhighlight %}
或者
{% highlight bash %}
# echo 'w /sys/class/leds/phy0-led/trigger - - - - phy0radio' > /etc/tmpfiles.d/phy0-led.conf
# systemd-tmpfiles --create phy0-led.conf
{% endhighlight %}

## 报错

### /bin/plymouth: No such file or directory
archlinux , thinkpad x61t，开机出错

journalctl -xn显示 /bin/plymouth: No such file or directory 

删掉 /etc/fstab 中不存在的介质就行了

### locale-gen时找不到character map file

``$sudo pacman -Syu``升级的时候出的问题

执行locale-gen提示：``character map file "en_US" not found``

结果locale就变成"C"

``$sudo pacman -S glibc``重装一遍，还是出错，不过提示空间不足

``$sudo pacman -Scc`` 清理空间

``$sudo pacman -S glibc -f``
