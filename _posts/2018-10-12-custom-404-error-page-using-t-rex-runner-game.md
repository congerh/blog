---
title: "使用Chrome浏览器彩蛋T-Rex Runner自定义404错误页面"
categories:
  - Blog
tags:
  - 404错误页面
  - T-Rex Runner
  - dino
---

当我们浏览网页的时候，有时候会遇到404 Not Found的错误页面，~~这时候有一个小游戏可以玩一玩，访客就不会因为找不到网页而那么迷茫~~。这篇文章简单介绍下如何提取Chrome浏览器的彩蛋T-Rex Runner，以及使用它配置404错误页面的方法。

## 介绍

### T-Rex Runner

T-Rex是谷歌Chrome浏览器中的一个彩蛋游戏，当你处于飞行模式或是断网的时候，浏览器会显示一个像素风格的小霸王龙图像，当你点击它时，它就会奔跑起来，期间不断躲避仙人掌和翼龙障碍物，跑的越远分数越高。你也可以直接在浏览器地址栏输入`chrome://dino`，这会直接打开街机模式全屏游戏而不用断网。

### 404 Not Found

404是HTTP状态码之一，通常表示客户端能正常与服务器通信，但服务器无法检索请求的页面。Web服务器通常可以配置自定义的404错误页面以显示更友好的描述，站点标识，站点地图等，许多网站开始装饰404页面，如谷歌会在页面上显示一个坏掉的机器人，也有些用来展示公益广告像是寻找失踪儿童。

## 提取Chromium中的彩蛋游戏源代码

T-Rex Runner源码位于Chromium项目的[offline.js](https://cs.chromium.org/chromium/src/components/neterror/resources/offline.js)文件中，目前文件在国内可能访问不了，官方也提供了Github上的[Chromium 镜像](https://github.com/chromium/chromium)。

你可以`Clone`整个Chromium项目或是仅下载和T-Rex Runner相关部分，文件位于`chromium/components/neterror/resources/`目录下。修改一下[offline.js](https://raw.githubusercontent.com/chromium/chromium/master/components/neterror/resources/offline.js)就可以运用到自己的页面中，你可以自行下载修改，也可以使用我修改过的版本，基于提交`24d8c44`，我已经上传到我的Github仓库上，详情请参阅[dino](https://github.com/congerh/dino)。

## 配置404页面

### Nginx的配置方法

将源码里的所有文件复制到你的服务器相应位置，把`index.html`修改为`404.html`或是任意你喜欢的名字，修改`server`或`http`，`location`上下文，添加以下代码，之后重启nginx或是执行`nginx -s reload`重新加载配置即可。

```sh
server {
    error_page 404 /404.html;
    location = /404.html {
        root /usr/share/nginx/html;
    }
}
```

### Github Pages配置方法

将源码里的所有文件复制到根目录，把`index.html`修改为`404.html`即可。

如果你使用Jekyll，要根据你的主题把css，js文件复制到相应的目录下，如`assets`等。
