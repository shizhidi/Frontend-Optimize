Web前端性能优化总结
===
亲爱的前端工作者朋友们：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里是在总结在多年工作中，遇到的一些问题，和前端性能优化的一些技巧，包括JavaScript，CSS，HTML，Cache，网络方面的优化，但个人水平难免有限，不免出现一些有错误或者不够完善的地方，欢迎大家提ISSUE指正，同时更加欢迎大家Fork下来提交自己的技巧，一起为前端的优化作出贡献。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;同时也提醒大家，过早地优化是万恶的源泉，对项目合理的规划才能充分的提高效率。

Thanks<br/>
小丘
基本原理
---
前端性能的优化，大致要有这几个原则，只要遵守了这几个原则，基本上性能不会太差。

- 减少HTTP请求数量
- 减少文件请求的大小，包括图片，.js，.css等文件
- 降低请求的延时，包括DNS延时，服务器延时
- 减少对Dom树的操作
- 减少页面重绘的次数
- 避免浏览器不必要的阻塞
- 正确的编程技巧，例如不在循环体里面定义变量
- 充分利用客户端缓存
- 最重要一点，遵守代码约定，编写其他人可维护的代码

优化工具
---

- 使用工具，压缩.js和.css文件，删除注释，删除空格回车，优化变量名，使用app.min.js, style.min.css
- YUI compressor ： 雅虎出品的压缩工具
- Closure Compiler ： Google出品的压缩工具
- UglifyJS ：基于Node的一款很不错的压缩工具
- html-minify : 压缩HTML文件
- 使用grunt，gulp，webpack，browserify 等包管理软件实现自动的合并，压缩，测试，发布

Cache 缓存／Storage 存储
---

- Headers设置浏览器缓存
  - "Expires" 设置缓存的过期时间，超过这个时间强制发起请求，不管其他的Etag，last-modified是否一致
  - "Cache-Control" : "max-age=200"，与Expires作用一样，但是Cache-Control功能更多,可以设置no-cache等等
  - "last-modified" : 指的是文件上一次被修改的时间，如果时间与缓存文件的时间不一致，则会发起完整的请求
  - "Etag" : 文件的一个"指纹"，如果Etag没有变化，则返回304状态码，如果不一致则发起完整的请求，使用CDN的时候有时候会造成Etag不一致，此时要避免使用Etag
  - last-modified 和 Expires 要配合着使用
  - Etag 和 Expires 要配合着使用
- Cookies
- LocalStorage
- sessionStorage
- IndexDB
- Web SQL
- Cache Storage
- Application Cache

JavaScript 脚本代码
---

- 使用严格模式 'use static'，严格模式可以消除JavaScript代码中一些不合理的地方，提高安全性，加速浏览器的编译速度，也对程序员代码风格有规范作用。关于严格模式和普通模式的区别，可以参考其他文档的说明
    ```JavaScript
    // IIFE模块中的写法
    (function(window, document) {
        'use strict';
        // body...
    })(window, document);

    // 普通函数中使用严格模式
    function functionName() {
        'use strict';
        // body...
    }
    ```
- 不要在循环体里面定义变量
    ```JavaScript
    // 在循环体外面定义变量，也不用在循环体里面多次获取array的长度
    var msg = "Message";
    for (var i = 0, length = array.length; i < length; i++) {
    	console.log(array[i], msg);
    }

    // 以下为错误示范
    for (var i = 0; i < array.length; i++) {
    	var msg = "Message";
    	console.log(array[i], msg);
    }
    ```
- 多次使用的全局变量使用局部变量保存起来，全局变量查找会很慢
    ```JavaScript
    // 使用局部变量保存全局变量
    var location = window.location;
    console.log(location.href);
    console.log(location.host);

    // 以下为错误示范
    console.log(window.location.href);
    console.log(window.location.host);
    ```
- 在不影响代码阅读和理解的前提下减少冗余代码
    ```JavaScript
    // 一般写法
    function max(a, b) {
    	if (a > b) {
    		return a;
    	} else {
    		return b;
    	}
    }

    // 简单写法 1 ，减少了else的书写
    function max(a, b) {
    	if (a > b) {
    		return a;
    	}
    	return b;
    }

    // 简单写法 2 ，使用三目运算
    function max(a, b) {
    	return a > b ? a : b;
    }
    ```
- 在使用自有属性的时候使用hasOwnProperty，typeof检查，防止报错
    ```JavaScript
    // hasOwnProperty
    for (var variable in object) {
        if (object.hasOwnProperty(variable)) {
            // body...
        }
    }

    // typeof
    if (typeof object.variable !== undefined) {
        // body...
    }
    ```
- 使用memorize()编程技巧，缓存结果
    ```JavaScript
    // 在参数一致的情况下返回一样的结果时，可以使用memorize
    function memorize(f) {
    	var cache = {}; //将值存在闭包中
    	return function () {
    		var key = arguments.length + Array.prototype.join.call(arguments, ",");
    		if (key in cache) {
    			return cache[key];
    		} else {
    			return cache[key] = f.apply(this, arguments);
    		}
    	}
    }
    // 改造后的函数
    var memFun=memorize(function (args) {
        // body...
    });
    // 或者
    ```
- 使用Promises防止回调地狱， [原文地址](https://developers.google.com/web/fundamentals/getting-started/primers/promises)
    ```JavaScript
    // Google developer 中关于promise的示范例子
    var promise = new Promise(function (resolve, reject) {
    	// 做某些事情
    	if ( /* 判断 */ ) {
    		resolve("Stuff worked!");
    	} else {
    		reject(Error("It broke"));
    	}
    });

    promise.then(function (result) {
    	console.log(result); // 正确结果
    }, function (err) {
    	console.log(err); // 错误结果
    });
    ```

CSS 样式
---

- 删除无用的CSS样式，例如删除行内元素的中关于块级元素的属性，
- 使用.Class来改变样式，避免使用element.style的方法来改变样式，因为在有多个element.style操作的时候，会产生多次的重绘，.Class只是会发生一次重绘。实在必须使用element.style方法的时候，可以使用以下方法，减少重绘次数
  ```javascript
  // 好的例子，发生一次重绘
  element.style.cssText += ";color:black;font-size:14px;";
  // 好的例子，发生一次重绘
  element.className = "active";
  // 以下为坏的例子，发生三次重绘
  element.style.padding = "2px";
  element.style.color = "black";
  element.style.fontSize = "14px";
  ```
- 不要使用CSS中的 _@import_ 来加载.css文件，需要的样式应该提前合并成一个文件
- 减少offsetXXX,scrollXXX,clientXXX的获取，这些方法会导致浏览器的重绘
- 使用CSS开启GPU加速，提升性能，CSS在做动画的时候，默认是没有使用GPU的，在进行3D变化的时候，才会使用GPU加速，这时候我们可以使用一些技巧，强制开启GPU加速2D的动画，以下两种方法都能开启GPU加速。
    ```css
     /* 使用 translateZ */
    .box1 {
        -webkit-transform: translateZ(0);
        -moz-transform: translateZ(0);
        -ms-transform: translateZ(0);
        -o-transform: translateZ(0);
        transform: translateZ(0);
    }

    /* 使用 translate3d */
    .box2 {
        -webkit-transform: translate3d(0,0,0);
        -moz-transform: translate3d(0,0,0);
        -ms-transform: translate3d(0,0,0);
        -o-transform: translate3d(0,0,0);
        transform: translate3d(0,0,0);
    }
    ```
- 小型的.css文件，可以直接以内联样式的形式，写到.html文件中，减少.css文件的请求，不过这样会造成

HTML 标签
---

- Item 1
- Item 2
- Item 3

网络优化
---

- 添加Accept-Encoding头，减少浏览器判断Encoding的时间
- 开启服务器gzip压缩文件大小
- 使用JSON代替XML传输数据
- 避免404的产生
- 使用keep-alive
- CDN
- DNS
    - DNS预解析
    ```html
    <meta http-equiv="x-dns-prefetch-control" content="on" />
    <link rel="dns-prefetch" href="http://cdn.domain.com" />
    ```
- HTTP/2.0
- SPDY
- TCP Fast Open
- hybla htcp发包算法
- 增加服务器并发连接数
    ```shellscript
    * soft nofile 51200
    * hard nofile 51200
    ```

参考资料
---

- 《高性能JavaScript》
- 《JavaScript高级程序设计》
- 《JavaScript权威指南》
- 《JavaScript函数式编程》
- 《CSS权威指南》
- 雅虎14条优化原则
- Google PageSpeed
- Google Developer
- Mozilla Developer Network
