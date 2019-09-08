## 什么是跨域？
>跨域指的是跨过同源策略，实现不同域之间进行数据交互的过程叫跨域。跨域的实现形式主要有JSONP方法、CORS方法、降域和postMessage等。

### 跨域的解决方式

### 1.JSONP

依托在html下的script可以加载任意域 下数据的特点，将script的src属性设置成欲请求的数据地址，先通过script去加载对应数据并将其作为js在当前页面运行，就实现了跨域操作，然而，对于某些json格式数据之间作为js执行时根本无法将这些数据进行完美解析渲染，所以可以通过在请求的url末尾上加个回调函数，并和后端进行约定好返回数据格式，将返回数据(如json数据)构造成能够作为js直接执行的格式，就实现了跨域同事前后端的良好交互，如下面例子：

js代码：
```
var http = require('http')
        var fs = require('fs')
        var path = require('path')
        var url = require('url')
        
        http.creatServer(function(req,res){
            var pathObj = url.parse(req,url,true)
            switch(pathObj.pathname){
                case'getNews':pathObj
                var news =[
                "第11日前瞻：中国冲击4金 博尔特再战200米羽球",

                "正直播柴飚/洪炜出战 男双力争会师决赛",

                "女排将死磕巴西！郎平安排男陪练模仿对方核心"
                ]
                res.setHeader('Content-Type','text/json';charset=utf-8)
                if(pathObj.query.callback){
                    res.end(pathObj.query.callback)+'(' + JSON.stringify(news) +')'    
                }else{
                    res.end(JSON.stringify(news))
                }
                break;

                default:
                fs.readFile(path.join(__dirname,pathObj.pathname),function(e,data){
                    if(e){
                        res.writeHead( 404，'not found')
                        res.end('<h1>404 not found</h1>')
                    }else{
                        res.end(data)
                          }    
                    })
            }
    }).listen(8080)
```
html代码：
```
<!DOCTYPE html>
<html>
<body>
  <div class="container">
    <ul class="news">
    </ul>
    <button class="show">show news</button>
  </div>

<script>

  $('.show').addEventListener('click', function(){
    var script = document.createElement('script');
    script.src = 'http://127.0.0.1:8080/getNews?callback=appendHtml';
    document.head.appendChild(script);
    document.head.removeChild(script);
  })

  function appendHtml(news){
    var html = '';
    for( var i=0; i<news.length; i++){
      html += '<li>' + news[i] + '</li>';
    }
    console.log(html);
    $('.news').innerHTML = html;
  }

  function $(id){
    return document.querySelector(id);
  }
</script>
</html>
```
演示效果：
![image](https://upload-images.jianshu.io/upload_images/3746979-1fce71f9c060ab48.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

### 2.CORS

CORS 全称是跨域资源共享（Cross-Origin Resource Sharing），是一种 ajax 跨域请求资源的方式，支持现代浏览器，IE支持10以上。 实现方式很简单，当你使用 XMLHttpRequest 发送请求时，浏览器发现该请求不符合同源策略，会给该请求加一个请求头：Origin，后台进行一系列处理，如果确定接受请求则在返回结果中加入一个响应头：Access-Control-Allow-Origin; 浏览器判断该相应头中是否包含 Origin 的值，如果有则浏览器会处理响应，我们就可以拿到响应数据，如果不包含浏览器直接驳回，这时我们无法拿到响应数据。所以 CORS 的表象是让你觉得它与同源的 ajax 请求没啥区别，代码完全一样。

js代码：发送AJAX请求

```
document.querySelector('.change').addEventListener('click', function () {
  var xhr = new XMLHttpRequest();
  xhr.open('get', '//a.test.com:8080/getNews', true);
  xhr.send();
  xhr.onreadystatechange = function () {
    if (xhr.readyState === 4 && xhr.status === 200) {
      appendHtml(JSON.parse(xhr.responseText))
    }
  }
  window.xhr = xhr
})
function appendHtml(news) {
  var html = ''
  for (var i = 0; i < news.length; i++) {
    html += '<li>' + news[i] + '</li>'
  }
  document.querySelector('.news').innerHTML = html
}
```
设置请求头"Access-Control-Allow-Origin":

```
var news = [
    "第11日前瞻：中国冲击4金 博尔特再战200米羽球",
    "正直播柴飚/洪炜出战 男双力争会师决赛",
    "女排将死磕巴西！郎平安排男陪练模仿对方核心"
  ]
  var data = [];
  for (var i = 0; i < 3; i++) {
    var index = Math.floor(Math.random() * news.length);
    data.push(news[index]);
  }
  res.header("Access-Control-Allow-Origin", "//test.com:8080"); // //test.com:8080表示只有//test.com:8080下发起请求才可以调用本域下的资源
  //res.header("Access-Control-Allow-Origin", "*");   //*表示任何域下发起请求都可以调用本域下的资源
  res.send(data);
```
###3.降域
降域是相对于iframe元素而言的 。
一般来说域名为http://b.test.com/b(A)的网页以iframe的形式嵌在域名为http://a.test.com/a(B)的网页中，浏览器发现该请求不符合同源策略，正常情况下不能进行跨域访问，但是当我们在两个页面中分别设置documet.domain = test.com的时候（注意观察这个方法有一个限制就是 域名中必须有相同的部分），B页面就可以访问到A页面中的资源了，比如下面的代码:

B网站：
```
<html>
<style>
    html,
    body {
        margin: 0;
    }

    input {
        margin: 20px;
        width: 200px;
    }
</style>

<input id="input" type="text" placeholder="http://b.test.com/b.html">
<script>
    // URL: http://b.test.com/b.html

    document.querySelector('#input').addEventListener('input', function () {
        window.parent.document.querySelector('input').value = this.value;
    })

    document.domain = 'test.com';

</script>

</html>
```
A网站：
```
<html>
<style>
    .ct {
        width: 910px;
        margin: auto;
    }

    .main {
        float: left;
        width: 450px;
        height: 300px;
        border: 1px solid #ccc;
    }

    .main input {
        margin: 20px;
        width: 200px;
    }

    .iframe {
        float: right;
    }

    iframe {
        width: 450px;
        height: 300px;
        border: 1px dashed #ccc;
    }
</style>

<div class="ct">
    <h1>使用降域实现跨域</h1>
    <div class="main">
        <input type="text" placeholder="http://a.test.com:8080/a.html">
    </div>

    <iframe src="http://b.test.com:8080/b.html" frameborder="0"></iframe>
</div>

<script>
    //URL: http://a.test.com:8080/a.html

    document.querySelector('.main input').addEventListener('input', function () {
        console.log(this.value);
        window.frames[0].document.querySelector('input').value = this.value;
    })

    document.domain = "test.com"

</script>

</html>
```
使用document.domain实现降域


