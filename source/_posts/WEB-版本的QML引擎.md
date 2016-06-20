---
title: WEB 版本的QML引擎
date: 2016-06-20 23:26:52
tags:
- qml
- web
---

### 前言

> [qmlweb](https://github.com/qmlweb/qmlweb) 是一个 Web 版本的 QML 引擎，是个 JavaScript 驱动的 QML 引擎。CSS 和 HTML 都不擅长于界面设计和交互接口，Qt 提供了一个很好的解决方案：QML。QML 是个声明性语言，非常适用于 UI 设计。qmlweb 的目标是把 QML 的特性应用到 Web 浏览器。

突然发现qml居然可以跑在浏览器，真是好兴奋ヾ(o◕∀◕)ﾉヾ，那时我的心情是酱紫的：喔肏，终于不用写html标签了，终于不用写CSS了，终于不用。。。（由于博主太过激动，已经语无伦次，以下省略一万字），刚开始接触qmlweb，其实我是拒绝的，毕竟我是一名伟大的JavaScript程序员（嘿，谁在拿鸡蛋丢我，给少爷站粗来💢），不过qml实在太好用了，就像加了特效，Duang！Duang！Duang！（嘘！悄悄告诉大家，其实博主在吹牛B，这玩意还只是一个试验品，千万别上当）只需要一种语言就可以实现UI的设计。举个🌰（哈，博主太懒，这儿直接抄的官方亻，嘿！谁在喷我口水！尊重一下我好不嘞？）：

```qm
import QtQuick 2.0

Rectangle {
   width: 500; height: 200
   color: "lightgray"

   Text {
       id: helloText
       text: "Hello world!"
       anchors.verticalCenter: parent.verticalCenter
       anchors.horizontalCenter: parent.horizontalCenter
       font.pointSize: 24; font.bold: true
   }
}
```

这样我们就在浏览器绘制出了一个500*300px的浅灰色矩形，在矩形的中间有悬浮着文本"Hello World!"，看起来似乎有点酷，是不是想入坑了呢(*^__^*) 嘻嘻……，别着急，等我把坑挖好，大家排队跳。

<!-- more -->

###  下载安装

> 官方提供了npm、Bower等安装方法，不过我等Qt开发者貌似不怎么鸟node那一套，我们采用原始一点的安装方法，直接下载[Release](https://github.com/qmlweb/qmlweb/releases)版本到本地进行解压即可。

```shell
# linux
wget https://github.com/qmlweb/qmlweb/archive/v0.1.0.zip
unzip v0.1.0.zip
cd qmlweb-0.1.0

# mac和windows下载解压即可
```

### 新建项目

我们新建一个目录qmlweb-demo，将qmlweb-0.1.0目录里的lib目录复制到qmlweb-demo里，在目录中新建一个qml文件夹和一个index.html文件，最后在qml文件夹中新建一个main.qml，这时候你的项目结构看起来是酱紫的：

```
├─qmlweb-demo
│  ├─lib
│  ├─qml
│  │  └─main.qml
|  └─index.html
```

### 使用qmlweb

在index.html中插入代码：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>QML Auto-load Example</title>
  </head>
  <body style="margin: 0;" data-qml="qml/main.qml">
    <script type="text/javascript" src="lib/qt.js"></script>
  </body>
</html>
```

body标签中的data-qml属性指定了qml的入口文件是qml目录下的main.qml，大家看到这儿可能会疑惑啦，肯定又想骂我了：Amast，你到底靠不靠谱，这代码里啥实质性的东东的都没有啊，你不会又在忽悠人吧，你到底行不行啊（噗，其实我是无辜的呀💔，先等我说完好咩），我们看html文件中引入的lib/qt.js文件中有这样一段代码：

```javascript
global.addEventListener('load', function () {
    var metaTags = document.getElementsByTagName('BODY');

    for (var i = 0; i < metaTags.length; ++i) {
        var metaTag = metaTags[i];
        var source = metaTag.getAttribute('data-qml');

        if (source != null) {
            global.qmlEngine = new QMLEngine();
            qmlEngine.loadFile(source);
            qmlEngine.start();
            break;
        }
    }
});
```

我们引入qt.js的时候就已经就已经做了处理啦，所以我们只需要上边的html文件提供一个舞台就可以啦，剩下的就统统交给qml吧，接下来我们在main.qml中写入以下代码：

```
import QtQuick 2.0

Rectangle {
   color: "red";
   anchors.fill: parent;

   Rectangle {
       color: "blue";
       width: parent.width / 2;
       height: parent.height / 2;
       anchors.centerIn: parent;
   }
}
```

接下来在浏览器中打开index.html，你就会发现浏览器里面出现了两个矩形，蓝色的悬浮在红色的中间，宽和高都是红色的矩形的一半，是不是比html + css + js 简单多了呢？n(*≧▽≦*)n

教程到这儿就结束啦~\(≧▽≦)/~啦啦啦

> qmlweb加载qml文件和模块使用的是ajax技术，大家可能会遇到跨域的问题，qml文件最好是和html放在同一个服务器上，以规避浏览器同源策略。