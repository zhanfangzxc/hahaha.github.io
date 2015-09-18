title: Android M 中的运行时权限
date: 2015-09-16 09:33:09
tags:
- Android
- M 权限
---

M 预览版中包含了一个运行时权限的方法:

```
Activity.shouldShowRequestPermissionRationale()

app在实际显示权限对话框之前是否显示一个对正在请求的权限的解释
```
在第一次安装的时候，这个方法会反悔false，因此你可以直接请求权限，如果用户以前拒绝过一个权限的请求，那么再次请求该权限的时候可以显示一个解释该请求用途的的信息。

当app完全没有机会被授权的时候，调用shouldShowRequestPermissionRationale()则返回false。

M预览版中还提供了下面这个方法：

```
Fragment.shouldShowRequestPermissionRationale()

这个方法总是返回false，所以我们可以调用

getActivity().shouldShowRequestPermissionRationale();
```
###参考文章：
[Android M中权限被拒绝时该如何处理](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0916/3464.html)

[运行时权限Sample](https://github.com/googlesamples/android-RuntimePermissions )
