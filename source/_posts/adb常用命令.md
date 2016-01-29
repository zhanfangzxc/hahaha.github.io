title: adb常用命令
date: 2016-01-07 17:49:24
tags:
- Android
- adb

---
原文地址:[http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0105/3833.html](http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0105/3833.html)

- adb kill-server 关闭服务
- adb devices 查看所有链接的设备以及对应的序列号
- adb install -r xxx.apk 安装APK,如果同时连接多台设备的话，需要添加**-s <设备号>**
- adb uninstall packageName 卸载APK
- adb shell pm clear packageName 清除应用的数据
- adb connect <设备的ip地址> 链接到指定的ip，通常用来wifi调试
- adb shell 进入shell环境
- adb shell dumpsys activity top 查看栈顶Activity,用来获取包名
- adb shell pm list packages -f 查看所有已安装的应用的包名
- adb shell dumpsys activity Activity Manager State
- adb shell dumpsys package 包信息
- adb shell dumpsys meminfo 内存使用情况
- adb shell dumpsys procstats 随着时间的推移，内存的使用情况
- adb shell dumpsys gfxinfo 图形状态
- adb pull <remote> <local> 从手机复制文件出来
- adb push <local> <remote> 向手机发送文件
- adb shell cat /proc/cpuinfo 查看手机cpu，可以看到手机架构和处理器
- adb version 查看adb版本
- adb help 进入adb帮助界面