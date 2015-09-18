title: 'Monkey测试工具介绍'
date: 2015-09-18 14:10:43
categories: Android
tags:
- Android
- Monkey

---

**翻译原文**:[http://antoine-merle.com/introduction-to-the-monkey-runner-tool-2/](http://antoine-merle.com/introduction-to-the-monkey-runner-tool-2/)

欢迎大家来到我的博客来观看这篇旨在介绍Monkey Runner[(谷歌官方介绍)](http://developer.android.com/tools/help/monkeyrunner_concepts.html)的文章，这篇文章可能是一篇延伸阅读。

这篇文章不是一个课程，只是一个介绍，因为目的是让你明白Monkey有多强大，Monkey Runner是一个可以让开发者创建一个程序来控制已经连接的设备，这些程序使用Python创建的，可以安装，运行，操作app，还可以截屏，以下是谷歌API提供的相关的三个模块:

- MonkeyRunner:提供连接设备/模拟器的方法
- MonkeyDevice:提供安装和卸载应用程序的方法
- MonkeyImage:提供截取图片的方法

让我们开始创建一个Monkey测试文件，命名为mymonkey.py:

	from com.android.monkeyrunner import MonkeyRunner, MonkeyDevice
	import commands
	import sys

	# starting script
	print "start"

	# connection to the current device, and return a MonkeyDevice object
	device = MonkeyRunner.waitForConnection()

	apk_path = device.shell('pm path com.myapp')
	if apk_path.startswith('package:'):
	    print "myapp already installed."
	else:
	    print "myapp not installed, installing APKs..."
	    device.installPackage('myapp.apk')

	print "launching myapp..."
	device.startActivity(component='com.myapp/com.myapp.MainActivity')

	#screenshot
	MonkeyRunner.sleep(1)
	result = device.takeSnapshot()
	result.writeToFile('./screenshots/splash.png','png')
	print "screen 1 taken"

	#sending an event which simulate a click on the menu button
	device.press('KEYCODE_MENU', MonkeyDevice.DOWN_AND_UP)

	print "end of script"

然后在命令行中执行如下命令:

	monkeyrunner mymonkey.py
	
- monkeyRunner命令直接包含在[android-sdk-path]/tools/
- 如果在windows上创建python项目，执行的时候必须写绝对路径(c:/etc)
