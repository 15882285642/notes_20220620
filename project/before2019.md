就业以来遇到的项目开发过程中遇到的疑惑和问题总结
## 一 eagleweb&新闻 pugnode开发学习笔记

> node 项目启动 node *。js文件 
> pug和jsp的${}类似都是服务器渲染
程序员在开发过程中，发现 servlet（可以理解为java的一个类）做界面非常不方便就引入了jsp； jsp=html+java 片段+标签+javascript+css

1.JSP 全称是 Java Server Pages，Java 的服务器页面，就是服务器端的渲染技术

2.JSP 这门技术的最大的特点在于，写 JSP 就像在写 HTML
  · 相比 html 而言，html 只能为用户提供静态数据，而 JSP 技术允许在页面中嵌套 java 代码， 为用户提供动态数据
  · 相比 Servlet 而言，Servlet 很难对数据进行排版，而 jsp 除了可以用 java 代码产 生动态数据的同时，也很容易对数据进行排版。

3.jsp 技术基于 Servlet, 你可以理解成 JSP 就是对 Servlet 的包装.

4.会使用 JSP 的程序员, 再使用 thymeleaf 是非常容易的事情, 几乎是无缝接轨.
原文链接：https://blog.csdn.net/qq_44981526/article/details/123539536

pug模版引擎开发（jade）典型的服务端渲染 mvc架构（开发和维护流程在另一篇文章中进行详细整理）
* * *

## 二 visha和joga开发学习笔记
> 1 移动端调试方法
 1 移动端下载chrome （新版）数据线连接电脑  -- 设置 系统  开发者选项 webview实现进入并勾选  并手机端统一该计算机访问；
 2 之后在chrome网页输入网址  chrome://inspect/#devices  会搜索连接的手机 及打开的页面；
 3 点击inspect fallback 进行调试 在移动端刷新页面等操作 查看电脑的情况；
其实最简单的方式是开启一个服务，手机和电脑在同一个网络下就可以直接在手机端看网页内容

> 2 遇到 在web端可以进行接口的请求 ，而在移动端接口就是请求不到
可能原因：后台的接口是https的存在安全证书的问题 导致手机端是不能进行访问的；转化为http的接口就可以了


> 3 给span标签绑定onclick事件，会产生冒泡行为导致点击失效，需要禁止冒泡
```
// 原生
var btn = document.getElementsByClassName('btn')[0];
btnFollow.addEventListener('click',function(event){
    event.stopPropagation();
    your operations
});
```

> 4 手机预装app的打开或者跳转到googleplay
```
// open or down app 方式一
function isInstalled(type ,value){
        var urlFrag = '';
        if(type == 'user'){
            urlFrag = '?userId='+ value ;
        }else if(type == 'video'){
            urlFrag = '?type=video&id='+ value ;
        }else{
            urlFrag = window.location.search;
        }
        var the_href = 'market://whichpage?id=com.*.*&referrer=**';//获得下载链接
        window.location.href = "appname://start" + urlFrag;//打开某手机上的某个app应用
        setTimeout(function(){
            window.location.href = the_href;//如果超时就跳转到app下载页
        },800);
    }
```
对于这种方法可能存在问题：
1 对于对网页有拦截的浏览器，会弹出提示框，如果延迟很短很容易出现连续弹出两次的情况
解决方法：比如自己公司的浏览器可以通过加入白名单来实现，但是其他的浏览器没有办法避免只能增加延迟，但是增加延时对无拦截的浏览器体验就是变差，比如google的；
2 可能对很多手机都不能100%的准确进行打开
两种方式
```
// 方式二
    function isInstalled(type ,value) {
        var timeout, t = 1000,
             hasApp = true,
             urlFrag = '';
        if(type == 'user'){
            urlFrag = '?userId='+ value ;
        }else if(type == 'video'){
            urlFrag = '?type=video&id='+ value ;
        }else{
            urlFrag = window.location.search;
        }
        url = "jogalauncher://start" + urlFrag ;
         var openScript = setTimeout(function() {
             if (!hasApp) {
                 var durl = 'market://whichpage?id=com.*.*&referrer=**';
                 window.location.href = durl;
             }
             document.body.removeChild(ifr);
         }, 2000)

         var t1 = Date.now();
         var ifr = document.createElement("iframe");
         ifr.setAttribute('src', url);
         ifr.setAttribute('style', 'display:none');
         document.body.appendChild(ifr);
         timeout = setTimeout(function() {
             var t2 = Date.now();
             if (!t1 || t2 - t1 < t + 100) {
                 hasApp = false;
             }
        }, t);
    }
// 
```
第二种方式的优点在于，利用iframe打开APP，按返回键不会再重新加载页面 ，没有那个缓存进度条； 需要在增加判断条件经测试等反复修改下面的方法可以成功打开app
```
openAppGp(app_href, gp_href);
function openAppGp(ap, gp) {
    //检查app是否打开
    function checkOpen(cb) {
        var _clickTime = +(new Date());
        function check(elsTime) {
            if (elsTime > 2000 || document.hidden || document.webkitHidden) {
                cb(1);
            } else {
                cb(0);
            }
        }
        //启动间隔20ms运行的定时器，并检测累计消耗时间是否超过3000ms，超过则结束
        var _count = 0, intHandle;
        intHandle = setInterval(function () {
            _count++;
            var elsTime = +(new Date()) - _clickTime;
            if (_count >= 50 || elsTime > 2000) {
                clearInterval(intHandle);
                check(elsTime);
            }
        }, 20);
    }
    //在iframe 中打开APP
    const link = document.createElement('a');
    document.body.appendChild(link);
    link.setAttribute('href', ap);
    link.style.display = 'none';
    link.click();
    // if (1) {
    checkOpen(function (opened) {//checkOpen中的cbk参数 = function (opened)
        if (opened == 0) {
            //用户没有安装app 可以请求下载地址并跳转 跳转方法：window.location.href 即可
            window.location.href = gp;
        } else if (opened == 1) {
            //用户打开了app  用户有安装app 
            console.log("Opend app");
        }
    });
    // }

    setTimeout(function () {
        document.body.removeChild(link);
    }, 2000);
}
```
3 使用iframe可能浏览器部分不兼容（时而可行时而不可行）将iframe改为a标签既可解决

> 5 播放视频是可以先用缩略图进行占位

方式一：给video设置成全透明加载完成后自动设置为非透明；
方式二：有poster属性直接设置该占位的图片；

> 6 对于图片尺寸不同的情况，可以使用瀑布流，进行优化
```
// waterFall('recmd_recmd','img-wrap');
// 图片的瀑布流
function waterFall(parent,box) {
    if(typeof parent === 'string'){
         parent = document.getElementById(parent);
    }else{
        parent = parent;
    }
    var allBox = mygetId(parent).getElementsByClassName(box);
    var boxWidth = allBox[0].offsetWidth;
    var screenWidth = document.body.offsetWidth;
    var cols = Math.floor(screenWidth/boxWidth);
    parent.style.width = boxWidth*cols + 'px';
    parent.style.margin = '0 auto';

    var heightArr = [];
    for(var i=0; i<allBox.length;i++){
        var boxWidth = allBox[i].offsetWidth;
        if(i < cols){
            heightArr.push(boxHeight);
        } else {
            var minBoxHeight = Math.min.apply(this,heightArr);
            console.log(minBoxHeight);
            var minBoxIndex = getMinBoxIndex(minBoxHeight,heightArr);
            allBox[i].style.position = 'absolute';
            allBox[i].style.top = minBoxHeight + 'px';
            allBox[i].style.left = minBoxIndex * boxWidth +'px';
            heightArr[minBoxIndex] += boxHeight ;
        }
    }
}
// function mygetId(myid){
//     if(typeof myid === 'string'){
//          rerurn document.getElementById(myid);
//     }else{
//         return myid;
//     }
//     // rerurn typeof myid === 'string' ? document.getElementById(myid):myid;
// }
function getMinBoxIndex(value,array){
    for(var i in array){
        if(array[i] == value) {
            return i;
        }
    }
}

```


> 7 如何让一个固定宽高的div 内的图片（不定尺寸，可能横屏可能竖屏）不压缩的填充 且其他部分纯色填充
```
类似效果
直接 max-height：div高
	 max-width:div宽 
就能实现；
```

> 8 window.location = ‘’在有的手机里面不起作用（同是chrome浏览器）有的手机相当于页面刷新 ，setTimeout仍然会执行
```
var the_href = "market://"
```

> 9 

* * *

## wechat&vue&ajax  18/0/05

> ajax提交整理
1. jquery
* jquery文件上传



* * *

## 答题dspadx 开发学习笔记


* * *

## 



* * *

## 
