---
title: Ubuntu 16.04 安装有道词典
date: 2016-06-19 00:35:33
tags: Linux
---

作为一个英语奇烂无比的次时代程序员，怎么能没有一款好的翻译工具在手呢，难道我每次需要翻译的时候都去打开浏览器么，这真是太影响效率了(PS: 其实之前有基于百度翻译API开发过一款简单的翻译工具，后来发现有Linux版本的有道就放弃了)

言归正传，最近博主升级了16.04，去[有道词典官网](http://cidian.youdao.com/ "Title")下载了最新的1.1.0版本进行安装：

``` shell
sudo dpkg -i youdao-dict_1.1.0-0-ubuntu_amd64.deb
```

执行结果如下：

``` shell
(正在读取数据库 ... 系统当前共安装有 277684 个文件和目录。)
正准备解包 youdao-dict_1.1.0-0-ubuntu_amd64.deb  ...
正在将 youdao-dict (1.1.0-0~ubuntu) 解包到 (1.0.2~ubuntu) 上 ...
dpkg: 依赖关系问题使得 youdao-dict 的配置工作不能继续：
 youdao-dict 依赖于 gstreamer0.10-plugins-ugly；然而：
  未安装软件包 gstreamer0.10-plugins-ugly。

dpkg: 处理软件包 youdao-dict (--install)时出错：
 依赖关系问题 - 仍未被配置
正在处理用于 hicolor-icon-theme (0.15-0ubuntu1) 的触发器 ...
正在处理用于 bamfdaemon (0.5.3~bzr0+16.04.20160523-0ubuntu1) 的触发器 ...
Rebuilding /usr/share/applications/bamf-2.index...
正在处理用于 gnome-menus (3.13.3-6ubuntu3) 的触发器 ...
正在处理用于 desktop-file-utils (0.22-1ubuntu5) 的触发器 ...
正在处理用于 mime-support (3.59ubuntu1) 的触发器 ...
在处理时有错误发生：
 youdao-dict
```

看输出，发现我们安装失败了，我就看不懂了，你说你一个翻译软件依赖gstreamer0.10-plugins-ugly是几个意思，按照正常思路，此时我们应该执行如下命令强制修复依赖：

``` shell
sudo apt-get install -f
```

输出如下：

``` shell
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
正在修复依赖关系... 完成
下列软件包将被【卸载】：
  youdao-dict
升级了 0 个软件包，新安装了 0 个软件包，要卸载 1 个软件包，有 26 个软件包未被升级。
有 1 个软件包没有被完全安装或卸载。
解压缩后将会空出 13.5 MB 的空间。
您希望继续执行吗？ [Y/n]
```

apt工具告诉我们youdao-dict将被卸载，what the fxxk? 居然找不到这个包，去官网一看，是2015-12-30发布的就释然了，估计依赖在ubuntu16.04上有差异，这时候我开始方了，难道要解包修改依赖再重新打包么？答案当然是不，机智的我找到了有道词典的1.0版本(博主已上传七牛)[下载传送门](http://o8z1hn5vs.bkt.clouddn.com/youdao-dict_1.0.2~ubuntu_amd64.deb "Title")，下载完成进入文件所在目录，执行如下命令：

``` shell
sudo dpkg -i youdao-dict_1.0.2~ubuntu_amd64.deb
sudo apt-get install -f
```

执行完这两条命令，如果安装顺利，现在你已经可以启动有道词典~\\(≧▽≦)/~啦啦啦

``` shell
youdao-dict
```
