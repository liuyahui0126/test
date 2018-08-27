#### 1. url解析 -> js时间线

##### url

1. 输入url后，进行DNS解析 找到对应ip地址

2. 建立Tcp连接 三次握手

3. 发起Http请求  请求报文

4. 服务端返回响应

5. 四次挥手，开始解析html文档

6. 浏览器布局渲染    布局、绘制  --  repaint 和 reflow

   ​

​    布局  根据渲染树信息 计算每个节点的几何值（在视口中的大小和位置） -所有相对单位都转化为屏幕上的绝对单位

​    绘制  在屏幕上绘制各个节点

​    repaint  -- 屏幕的一部分重画，但不影响整体布局 比如css背景变了 但不影响整体布局

​    reflow   -- 意味着元素的几何尺寸变了，需要重新验证并计算渲染树 这就是reflow 

- *引申*--怎么减少重绘和重排次数？

1. 不要一条一条修改dom样式 （可以加一个class 统一控制）
2. 把dom离线后设置  （1.文档碎片 2.display:none ）
3. 为动画设置 fixed 或者 absolute

##### js时间线

1. 创建document对象，开始解析web页面，创建htmlelement对象，将这个对象添加到 document中。这个时候document.readyState = "loading"
2. 遇见外部 link  创建线程加载 ，并继续解析文档，并发
3. 遇见外部 script 并没有设置 async ， defer  ，阻塞。等待js执行完该脚本 继续解析
4. async 属性的脚本 加载完立即执行  defer丢至尾部
5. 遇到 img 等浏览器创建线程加载 ，继续解析文档，并发
6. 当文档解析完成后 ，document.readyState = "interactive”
7. 文档解析完成后，所有defer 会按照顺序执行
8. 触发 DOMContentLoaded 事件 ，标志开始执行
9. 当所有async的脚本加载完成并执行后、img等加载完成后，document.readyState = 'complete'，window对象触发load事件。

---

- 引申  async 和 defer  区别

加载时间不同 ，执行方式不同（defer是顺序执行）

---

#### 2.js异步处理机制

- 首先明确一点  promise 和 setInterval 虽然都是异步，但不属于同一个队列中  

  渲染引擎线程：顾名思义，该线程负责页面的渲染

  JS引擎线程：负责JS的解析和执行

  定时触发器线程：处理定时事件，比如setTimeout, setInterval

  事件触发线程：处理DOM事件

  异步http请求线程：处理http请求

  注意： js引擎和渲染引擎是不能同时进行的 ，渲染引擎在执行过程中，js引擎会先挂起

- 消息队列和事件循环

栈      栈中主要存在 同步任务  就是那些能立即执行，不耗时的任务

队列   消息队列 一旦某个异步有了响应就会推入队列中去，一旦同步任务执行完毕，就会把异步任务的回调推入栈中，单线程开始执行新的同步任务

堆	堆中主要存在的是声明的变量、对象



事件循环 ：每次在栈被清空之后，就会在消息队列中读取新的任务，如果没有新的任务，就会等待，直到有新的任务，这就叫事件循环



`setTimeout`的作用是在间隔一定的时间后，将回调函数插入消息队列中，等栈中的同步任务都执行完毕后，再执行。因为栈中的同步任务也会耗时，**所以间隔的时间一般会大于等于指定的时间**。

setTimeout(fn,0)的意思是将函数立即插入消息队列，等待执行，而不是立即执行

- 引申 — setinterval 和 promise 谁先执行

``` (function test() {  ``` 

```  	setTimeout(function() {console.log(4)}, 0);    ```  

```	new Promise(function executor(resolve) {        ```

```	console.log(1);        ```	

```for( var i=0 ; i<10000 ; i++ ) {            ```

```i == 9999 && resolve();        ```

```}        ```

```console.log(2);    ```

```}).then(function() {        ```

```console.log(5);    ```

```});    ```

```console.log(3);})()```

解析问题：

1. Promise.then 是异步执行   而promise 的实例是同步执行的
2. settimeout和promise 不处于同一个异步队列中

两种异步执行队列

### macrotasks  

- setTimeout，setInterval，setImmediate， I/O, UI rendering

### microtasks

- process.nextTick, Promises, Object.observe(废弃), MutationObserver

**在每一次事件循环中，macrotask 只会提取一个执行，而 microtask 会一直提取，直到 microtasks 队列清空**。

---

### 3. em和rem区别

- em

  em相对于浏览器font-size值（默认为16px）进行计算 .em的值并不是固定的，他会继承父级元素的字体大小（相对于设置em的元素的大小）

1.2em ===  》 1em

- rem 

  相对于根元素的大小

---

#### 4.webpack怎么实现按需加载

首先说明一点 webpack使用的是 import/require和export方式进行导入导出，

- 引申— import和reuqire区别

无明确要求是import还是require，标准不同。import是基于es6。通过使用loader等插件进行转义（babel-loader）

而require是基于之前的commonjs规范。

commonjs — require —动态加载中使用

1. 通过require引入基础数据类型的时候。属于复制该变量
2. 通过require引入复杂数据类型的时候。数据浅拷贝该对象
3. 默认 export 是一个对象

es6 — import   

1. 不管是哪种数据类型，都是对变量进行只读引用。动态在于一个变量中的模快会影响到另一个模快。对于复杂数据类型，可以添加属性和方法，但不允许引入另一个内存空间
2. 出现模块之间的循环引用，只有模块存在某个引用，代码就能够执行
3. import静态编译，import的地址不能通过计算

- 引申—path和publicpath区别
  1. path 输出的目录，对应一个绝对路径
  2. publicpath对所有资源指定一个公共路径 （引入css，js等静态资源的一个基础路径）
  3. 静态资源最终访问路径 = output.publicPath + 资源loader或插件等配置路径
- 引申— 按需加载实现原理

```
require.ensure(dependencies, callback, chunkName)
```

这是一个按需加载的函数

这里顺带一提，打包后的js文件基础路径跟普通的资源(图片或字体文件之类)是一样的，就是publicPath， publicPath可以在运行时再去赋值，方法就是在应用入口文件对变量 `__webpack_public_path__` 进行赋值就行

可以使用函数打包三个js脚本 ，但有些脚本在不使用的时候，永远不去加载

（if(false)）

---

#### 5.前端性能优化

三个维度  静态资源优化、接口访问优化、页面渲染速度优化

1. 静态资源优化

- 合并css、js文件，制作雪碧图：减少http的请求次数，节省网络请求时间
- 静态资源cdn分发：客户端可以通过最佳的网络链路加载静态资源
- js、css文件压缩，图片压缩，gzip压缩：减少请求返回的数据量（gulp）

2. 接口访问优化 

- 减少请求资源的次数  合并多个请求 （webpack）

3. 页面渲染速度优化

- 首屏直接加载 ，懒加载，预加载方案
- 减少重排重绘次数（尽量使用文档碎片，或者display：none等方法）

---

### 6.

