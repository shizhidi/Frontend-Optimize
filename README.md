Web前端性能优化总结
===
亲爱的前端工作者朋友们：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里是在总结在多年工作中，遇到的一些问题，和前端性能优化的一些技巧，包括JavaScript，CSS，HTML，Cache，网络方面的优化，但个人水平难免有限，不免出现一些有错误或者不够完善的地方，欢迎大家提ISSUE指正，同时更加欢迎大家Fork下来提交自己的技巧，一起为前端的优化作出贡献。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;同时也提醒大家，过早地优化是万恶的源泉，对项目合理的规划才能充分的提高效率。

Thanks  
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
  - 设置cookie的过期时间expires，合理利用cookie可以实现用户免登录的功能，在流程上减少操作
  - 压缩cookie的大小，减少不必要的cookie
  - cookie的domain级别要设置合理
  - 静态的资源就不要使用cookie了，减少不必要的数据量
- Web Storage 包括sessionStorage，localStorage，IndexDB，Web SQL，根据业务逻辑，存储需要的数据，但由于都是HTML5的标准，所以还有些兼容性的问题。
- Cache 包括Application Cache和Cache Storage，利用浏览器的应用缓存功能，可以完成离线应用的制作，一个完全离线的App，自然对服务器没有压力
- websocket 根据自己的业务合理的使用websocket替换掉Ajax，JSONP，传输性能好于HTTP，传输数据量也会少很多

JavaScript 脚本代码
---

- 把JavaScript脚本放在页面的底部或者使用defer属性进行延时下载，因为JavaScript脚本的下载，会阻塞其他资源的下载，也会阻塞浏览器的渲染，defer可以用于内联的script标签，也可以用于外部的script，在HTML5中增加了一个async异步下载功能，只能用于外部的script，异步下载的脚本在下载的过程中，不会阻挡浏览器渲染其他元素，下载完之后再立即执行。当延时下载的JavaScript文件中有Dom元素操作的时候，使用这种方法，可能造成页面闪烁问题，也会造成多次的重绘，注意合理编写代码。另外需要注意的是，这里的async和JSONP中的async，不是一回事。
  ```html
  <script type="text/javascript" defer="defer"></script>
  <script type="text/javascript" src="main.js" async="async"></script>
  ```

- 使用严格模式 'use strict'，严格模式可以消除JavaScript代码中一些不合理的地方，提高安全性，加速浏览器的编译速度，也对程序员代码风格有规范作用。关于严格模式和普通模式的区别，可以参考其他文档的说明
  ```javascript
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
- 按需加载，如果使用ES6，以及Webpack打包，可以使用异步加载模块的方法，加载需要的资源，不用将所有的资源都打包到一个文件里。
  ```javascript
  const util = () => import '../lib/Util';
  ```
- 使用ES7 的语法 async／await 将异步代码，变为同步的逻辑，避免回调嵌套，能让代码逻辑更清晰（现阶段需要使用babel等工具转换），async／await其实是Generators的语法糖。
  ```javascript
  // ES7 写法
  const asyncFunc = async () => await ajax.post();
  // ES6 写法
  // 暂时不支持箭头函数和generator一起的写法，辣鸡
  const asyncFunc = function *() => yield ajax.post();
  ```
- 使用函数式编程的方法，能够更好的组织代码的逻辑，例如pipeline的方法，不用将每一次的结果都用数据保存，不用函数嵌套，逻辑更清晰。
  ```javascript
  const pipeline = (...args) => args.reduce((l, r) => r(l));

  // example
  const plusOne = (x) => x + 1;
  const minusTwo = (x) => x - 2;
  const multiplyThree = (x) => x * 3;

  const result = pipeline(1, plusOne, plusOne, minusTwo, multiplyThree);
  console.log(result); //结果为 3
  ```
- 不要在循环体里面定义变量
  ```javascript
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
  ```javascript
  // 使用局部变量保存全局变量
  var location = window.location;
  console.log(location.href);
  console.log(location.host);

  // 以下为错误示范
  console.log(window.location.href);
  console.log(window.location.host);
  ```

- 在不影响代码阅读和理解的前提下减少冗余代码
  ```javascript
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
  ```javascript
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

- 使用memorize()编程技巧，缓存结果，可以用Node运行代码测试效果
  ```javascript
  // 在参数一致的情况下返回一样的结果时，可以使用memorize
  function memorize(f) {
    var cache = {}; //将值存在闭包中
    return function () {
      var key = arguments.length + Array.prototype.join.call(arguments, ",");
      if (key in cache) {
        return cache[key];
      } else {
        cache[key] = f.apply(this, arguments);
        return cache[key];
      }
    };
  }
  // 斐波那契数列
  var fibonacci = function (n) {
    return n < 2 ? n: fibonacci(n - 1) + fibonacci(n - 2);
  };

  // 普通fibonacci的时间
  console.time("普通fibonacci的时间");
  console.log("结果：" + fibonacci(40));
  console.timeEnd("普通fibonacci的时间");

  // 使用memorize优化过fibonacci的时间
  var fibonacci = memorize(fibonacci);
  console.time("优化过fibonacci的时间");
  console.log("结果：" + fibonacci(40));
  console.timeEnd("优化过fibonacci的时间");
  ```

  > 结果：102334155  
  > 普通fibonacci的时间: 1714.656ms  
  > 结果：102334155  
  > 优化过fibonacci的时间: 1.001ms  


- 使用Promises防止回调地狱， [原文地址](https://developers.google.com/web/fundamentals/getting-started/primers/promises)
  ```javascript
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

- 删除无用的CSS样式，例如删除行内元素的中关于块级元素的属性
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
  -webkit-backface-visibility: hidden;
  -webkit-perspective: 1000;
  -webkit-transform: translatez(0);
     -moz-transform: translatez(0);
      -ms-transform: translatez(0);
       -o-transform: translatez(0);
          transform: translatez(0);
  }

  /* 使用 translate3d */
  .box2 {
    -webkit-backface-visibility: hidden;
    -webkit-perspective: 1000;
    -webkit-transform: translate3d(0, 0, 0);
       -moz-transform: translate3d(0, 0, 0);
        -ms-transform: translate3d(0, 0, 0);
         -o-transform: translate3d(0, 0, 0);
            transform: translate3d(0, 0, 0);
  }
  ```

- 小型的.css文件，可以直接以内联样式的形式，写到.html文件中，减少.css文件的请求，不过这样会造成

HTML 标签
---

- 延时加载 非首屏的图片元素，可以延时加载，例如使用lazyLoad技术，在用户滚动屏幕的时候，再启用下载，减少服务器压力
  ```javascript
  var imgs = document.getElementsByTagName('img');
  // 获取视口高度与滚动条的偏移量
  function lazyload() {
    var scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop;
    var viewportSize = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight;
    for (var i = 0; i < imgs.length; i++) {
      var x = scrollTop + viewportSize - imgs[i].offsetTop;
      if (x > 0) {
        imgs[i].src = imgs[i].getAttribute('loadpic');
      }
    }
  }
  ```

- 预先加载 预测用户行为，在页面渲染完毕后可以预先加载下一个页面需要的内容
  - 搜索引擎首页打开后，可以预先下载搜索结果页面的需要的内容
  - 在线看小说，可以预先下载下一个章节的文字，图片等
  - 流程类的，在Step 1 渲染结束后，可以预先下载Step 2 的内容
- 减少Dom节点的数量
- 比较小的文件，又做不了Sprit的时候，可以考虑通过base64转码之后，写在代码里面，减少一次请求
  ```css
  background: url(data:image/png;base64,);
  ```

网络优化
---

- 符合HTTP规范的用法，例如GET是用来获取数据等，POST是用来发送数据的，GET相对于POST速度会更快，减少了Header的部分，POST相对GET能发送更大的数据
- 添加Accept-Encoding头，减少浏览器判断Encoding的时间
- 开启服务器Gzip压缩文件大小，一般能获得70%以上的压缩效果，速度提升非常明显，不过对于小于1k的资源不建议压缩，意义不大而且增加服务器的负担，静态文件可以在服务器设置预先Gzip压缩缓存，不用每次请求都压缩一次
- 使用JSON代替XML传输数据
- 避免404的产生，favicon.icon 是一定要存在的
- 使用keep-alive
- 在后端服务器中使用flush，刷新缓冲区，让页面尽早的输出，包括让Head部提前输出，下载样式和脚本文件，让首页可视的部分提前输出给用户，例如在node.js，php, JSP中可以这么使用
  ```javascript
  // Node.js
  var http = require('http');

  http.createServer(function (request, response) {
    // 先把Head部分输出，浏览器会下载CSS文件，js文件
    response.write('<html><head><title>Title</title><script>alert("Head加载完毕！")</script></head>');

    // 处理完body部分再输出，这里为了举例子，设置10秒之后响应
    setTimeout(function () {
      response.write('<body><script>alert("10s后body加载完毕！")</script></body></html>');
      response.end();
    }, 10000);
  }).listen(8888);
  ```

  ```php
  // PHP
  <head>
    <!-- css, js, meta -->
  </head>
  // 将Head部分的标签先输出到浏览器上面，提早.css文件,.js文件下载的时间
  <?php flush(); ?>  
  <body>
    <!-- body -->
  </body>
  ```

  ```jsp
  <!-- JSP -->
  <jsp:include page="included.html" flush="true" />
  ```

- CDN
  - 将静态的资源放在CDN上面，可以减少用户下载的时间，更能节省服务器宝贵的带宽，更可以根据用户的网络情况，自动选择合适的网络运营商下载静态资源，提高用户的访问速度
  - 使用公共CDN中的.js文件，例如某个版本的jQuery，内容是一样的，在用户访问别的网站缓存下来的.js文件，在自己的网站上也可以使用。不过这样会丢失掉代码的可控性，有一定的风险，当Google无法访问的时候，Google提供的CDN文件，也无法访问了，缺少了代码库的支持，网站会出现重大的问题。这时候可以添加以下代码，这样就提供了一个备选的方案，当Google的CDN无法使用的时候，去调用Microsoft的CDN
    ```html
    <script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.x.x/jquery.min.js"></script>
    <script type="text/javascript">
    !window.jQuery && document.write('<script src=http://ajax.microsoft.com/ajax/jquery/jquery-1.x.x.min.js><\/script>');
    </script>
    ```

  - 也有很多人提出过，在浏览器中内置常用的.js库文件，但并没有被纳入w3c标准中
- DNS
  - 根据浏览器对每个域名的并行线程数（一般为6个），设置更多的域名，最大化并发的下载树，不过一般两三个域名为佳，过多的域名，会增加DNS查询带来的延时
  - DNS预解析，可以对未来需要使用的域名进行预先的解析
    ```html
    <meta http-equiv="x-dns-prefetch-control" content="on" />
    <link rel="dns-prefetch" href="http://cdn.domain.com" />
    ```

- HTTP/2.0
- SPDY
- TCP Fast Open
- hybla htcp拥塞控制算法，适用在延迟低的环境
- Google TCP BBR 拥塞控制算法
  这个BBR拥塞控制算法，能极大的提高网站的下载速度，适用在延迟比较高低地方，例如国外低服务器，需要更新Linux kernel 4.9 内核
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
- Github Google BBR
