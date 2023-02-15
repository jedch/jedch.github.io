---
layout: post
title: 系统小技巧
category: python
---

1. windows有一个分区工具好用，比如闪存有多个分区，无法普通方法格式化成一个。这个工具可以开工。diskpart运行，打开新的终端窗口。list disk，显示磁盘。如果闪存是磁盘2，选择它select disk 2，注意disk后的空格。输入clean清除闪存上所有内容。创建分区create partition primary。退出exit。好了，可以在windows磁盘管理里面操作了。
2. mac下制作macos安装u盘。在商店可以下载high sierra镜像，拷贝到下载，命名为highsierra.app，抹掉8g以上u盘，格式Mac OS扩展日志，名称hs。在命令行下进入下载，运行sudo highsierra.app/Contents/Resources/createinstallmedia --volume /Volumes/hs，需要输入Y确认，耐心等待约十多分钟，搞定。顺便多说一句，插上u盘开机听见滴声按住Option键不放，进入安装界面。
3. 两个路由器桥接。入户联通光猫，带路由，后面简称主路由。另有一个网件路由器，用有线连接在光猫上，简称副路由。两个路由设置同一网段，不同地址。两个路由ssid，密码设置一致。使用不同信道。副路由上关闭DHCP。可以了。
4. MacOS安装的软件：百度云，qq音乐，chrome，cakebrew，迅雷，free download，keka，xnviewmp，macdown，anaconda，docker，sublime，latex，texshop，暴风影音，qq，adobereader，xcode，github，cmake，pycharm，ssng，freemind，微信，synology drive client，