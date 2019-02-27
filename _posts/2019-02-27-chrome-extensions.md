---
layout:       post
title:        "Chrome extensions"
subtitle:     "谷歌浏览器插件开发"
date:         2019-02-27
author:       "Rong"
header-mask:  0.3
catalog:      true
tags:
    - Chrome
    - JavaScript
---

- 在每天工作当中避免不了Google，很多零碎的知识点在开发过程中并没有消化完，而且当下班或者其它事打扰后就不记得了，所以萌生了开发个导出历史记录到服务器，每天或者每周复盘的想法。
- 之前我是遇到较好的博文或者知识点就点收藏，而且很多来不及看的就放到read later文件夹，假装自己会看~日积月累的一大堆收藏，很是不爽。
- 一般我使用chrome浏览器，搭配[小飞机](https://github.com/shadowsocks/shadowsocks/releases)进行fanqiang。当遇到某个问题会Google一下，再或者就是GitHub和[stackoverflow](https://stackoverflow.com/questions)查有没有相似的，写完代码关闭电脑后没法在大脑中记忆下。而且也想做个词云和常用网站分析的应用，所以需要基础数据。

既然有了想法就开干，先看下chrome[官网的教程](https://developer.chrome.com/extensions/getstarted)，这是相关api和应用实例，可以粗略的看下。再自己写个demo，先跑起来再说。需要点很lower的HTML、css、JavaScript的基础知识。

1. 创建个manifest.json

```json
{
   "browser_action": {
      "default_popup": "popup.html"
   },
   "description": "This extension allow to export your browser history",
   "icons": {
      "128": "icon_128.png"
   },
   "manifest_version": 2,
   "name": "Back my History ",
   "permissions": [ "history", "downloads" ],
   "version": "0.1",
   "web_accessible_resources": [ "css/style.css" ]
}
```

2. 新建popup.html，这是我们点击插件后的页面

   ```html
   <!DOCTYPE HTML>
   <html>
       <head>
           <title>Export history</title>
           <link href="css/style.css" rel="stylesheet" type="text/css">
           <script src='popup.js'></script>
       </head>
       <body>
           <div class="global-box">
               <h2>upload your browser history</h2>
               <div class="buttons-box">
                   <a href="#" class="action-button" id="btn-day">last day</a>
                   <a href="#" class="action-button" id="btn-week">last week</a>
                   <a href="#" class="action-button" id="btn-all">all history</a>
               </div>
               <span id="info-text"></span>
           </div>
       </body>
   </html>
   ```

   确实很lower。

3. js写规则

   ```js
   var History = function () {}
   History.prototype.upload = function (history) {
     var data = [];
     var index;
     for (index = 0; index < history.length; ++index) {
       data.push({
         'id': history[index].id,
         'lastVisitTime': new Date(history[index].lastVisitTime).toLocaleString(),
         'lastVisitTimeTimestamp': history[index].lastVisitTime,
         'title': history[index].title,
         'typedCount': history[index].typedCount,
         'url': history[index].url,
         'visitCount': history[index].visitCount
       });
     }
     const xhr = new XMLHttpRequest()
     xhr.open("POST", "http://localhost:8080/coupons/getJson", true);
     xhr.setRequestHeader("Content-Type", "application/json");
     xhr.send(JSON.stringify(data))
     xhr.onreadystatechange = function () {
       if (this.readyState === XMLHttpRequest.DONE && this.status == 200) {
         alert("ok");
       }
     }
   }
   
   History.prototype.getHistory = function (days) {
     var now = new Date();
     var startTime = this.daysBefore(now, days);
     chrome.history.search({
       'text': '',
       'maxResults': 1000000,
       'startTime': startTime,
     }, this.upload);
   }
   
   History.prototype.daysBefore = function (date, days) {
     date.setDate(date.getDate() - days);
     timestamp = date.getTime();
     return timestamp;
   }
   
   document.addEventListener('DOMContentLoaded', function () {
     var history = new History();
   
     document.getElementById('btn-day').onclick = function () {
       history.getHistory(1);
     };
     document.getElementById('btn-week').onclick = function () {
       history.getHistory(7);
     };
     document.getElementById('btn-all').onclick = function () {
       history.getHistory(100);
     };
   });
   ```

4. 加载插件

   打开`chrome://extensions/`,打开开发者模式，加载已解压的扩展程序。。。

5. 点击查看是否发送数据到服务器

   ![点击插件弹窗](/img/in-post/post-incloud/2019-02-27_175123.png)

6. 选择导出前一天的试下功能

![后端接受到请求，而且数据也有](/img/in-post/post-incloud/2019-02-27_175428.png)

虽然这个插件很水，也没有去上传到chrome插件库，将就自己先用，慢慢完善喽。

撒花*★,°*:.☆(￣▽￣)/$:*.°★* 。

