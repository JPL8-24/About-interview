# CommonJS和AMD和CMD

## 1.CommonJS

node.js的模块系统基于CommonJS规范实现的

CommonJS定义的模块分为:{模块引用(require)} {模块定义(exports)} {模块标识(module)}

require()用来引入外部模块；exports对象用于导出当前模块的方法或变量，唯一的导出口；module对象就代表模块本身。（module.exports={}最好的导出方式

依赖node的四个全局变量：

> - module
> - exports
> - require
> - global



## 2.AMD

因为CommonJs中require是同步的，这对服务器端不是一个问题，因为所有的模块都存放在本地硬盘，可以同步加载完成，等待时间就是硬盘的读取时间。但是，对于浏览器，这却是一个大问题，因为模块都放在服务器端，等待时间取决于网速的快慢，可能要等很长时间，浏览器处于"假死"状态。 

因此，浏览器端的模块，不能采用"同步加载"（synchronous），只能采用"异步加载"（asynchronous）。这就是AMD规范诞生的背景。 

AMD也采用require()语句加载模块，但是不同于CommonJS，它要求两个参数：

> 　　require([module], callback);

第一个参数[module]，是一个数组，里面的成员就是要加载的模块；第二个参数callback，则是加载成功之后的回调函数。如果将前面的代码改写成AMD形式，就是下面这样：

> 　　require(['math'], function (math) {
>
> 　　　　math.add(2, 3);
>
> 　　});

**RequireJS就是实现了AMD规范的呢。**

**详细概括：下面以RequireJS为例说明AMD规范**

一、为什么要用require.js？

最早的时候，所有Javascript代码都写在一个文件里面，只要加载这一个文件就够了。后来，代码越来越多，一个文件不够了，必须分成多个文件，依次加载。下面的网页代码，相信很多人都见过。

 

> 　　<script src="1.js"></script>
> 　　<script src="2.js"></script>
> 　　<script src="3.js"></script>
> 　　<script src="4.js"></script>
> 　　<script src="5.js"></script>
> 　　<script src="6.js"></script>

 

这段代码依次加载多个js文件。

 

这样的写法有很大的缺点。首先，加载的时候，浏览器会停止网页渲染，加载文件越多，网页失去响应的时间就会越长；其次，由于js文件之间存在依赖关系，因此必须严格保证加载顺序（比如上例的1.js要在2.js的前面），依赖性最大的模块一定要放到最后加载，当依赖关系很复杂的时候，代码的编写和维护都会变得困难。

 

require.js的诞生，就是为了解决这两个问题：

 

> 　　![img](http://image.beekka.com/blog/201211/bg2012110701.png)
>
> 　　（1）实现js文件的异步加载，避免网页失去响应；
>
> 　　（2）管理模块之间的依赖性，便于代码的编写和维护。

 

二、require.js的加载

 

使用require.js的第一步，是先去官方网站[下载](http://requirejs.org/docs/download.html)最新版本。

 

下载后，假定把它放在js子目录下面，就可以加载了。

 

> 　　<script src="js/require.js"></script>

 

有人可能会想到，加载这个文件，也可能造成网页失去响应。解决办法有两个，一个是把它放在网页底部加载，另一个是写成下面这样：

 

> 　　<script src="js/require.js" defer async="true" ></script>

 

async属性表明这个文件需要异步加载，避免网页失去响应。IE不支持这个属性，只支持defer，所以把defer也写上。

 

加载require.js以后，下一步就要加载我们自己的代码了。假定我们自己的代码文件是main.js，也放在js目录下面。那么，只需要写成下面这样就行了：

 

> 　　<script src="js/require.js" data-main="js/main"></script>

 

data-main属性的作用是，指定网页程序的主模块。在上例中，就是js目录下面的main.js，这个文件会第一个被require.js加载。由于require.js默认的文件后缀名是js，所以可以把main.js简写成main。

 

三、主模块的写法

 

上一节的main.js，我把它称为"主模块"，意思是整个网页的入口代码。它有点像C语言的main()函数，所有代码都从这儿开始运行。

 

下面就来看，怎么写main.js。

 

如果我们的代码不依赖任何其他模块，那么可以直接写入javascript代码。

 

> 　　// main.js
>
> 　　alert("加载成功！");

 

但这样的话，就没必要使用require.js了。真正常见的情况是，主模块依赖于其他模块，这时就要使用AMD规范定义的的require()函数。

 

> 　　// main.js
>
> 　　require(['moduleA', 'moduleB', 'moduleC'], function (moduleA, moduleB, moduleC){
>
> 　　　　// some code here
>
> 　　});

 

require()函数接受两个参数。第一个参数是一个数组，表示所依赖的模块，上例就是['moduleA',  'moduleB',  'moduleC']，即主模块依赖这三个模块；第二个参数是一个回调函数，当前面指定的模块都加载成功后，它将被调用。加载的模块会以参数形式传入该函数，从而在回调函数内部就可以使用这些模块。

 

require()异步加载moduleA，moduleB和moduleC，浏览器不会失去响应；它指定的回调函数，只有前面的模块都加载成功后，才会运行，解决了依赖性的问题。

 

下面，我们看一个实际的例子。

 

假定主模块依赖jquery、underscore和backbone这三个模块，main.js就可以这样写：

 

> 　　require(['jquery', 'underscore', 'backbone'], function ($, _, Backbone){
>
> 　　　　// some code here
>
> 　　});

 

require.js会先加载jQuery、underscore和backbone，然后再运行回调函数。主模块的代码就写在回调函数中。

 

四、模块的加载

 

上一节最后的示例中，主模块的依赖模块是['jquery',  'underscore',  'backbone']。默认情况下，require.js假定这三个模块与main.js在同一个目录，文件名分别为jquery.js，underscore.js和backbone.js，然后自动加载。

 

使用require.config()方法，我们可以对模块的加载行为进行自定义。require.config()就写在主模块（main.js）的头部。参数就是一个对象，这个对象的paths属性指定各个模块的加载路径。

 

> 　　require.config({
>
> 　　　　paths: {
>
> 　　　　　　"jquery": "jquery.min",
> 　　　　　　"underscore": "underscore.min",
> 　　　　　　"backbone": "backbone.min"
>
> 　　　　}
>
> 　　});

 

上面的代码给出了三个模块的文件名，路径默认与main.js在同一个目录（js子目录）。如果这些模块在其他目录，比如js/lib目录，则有两种写法。一种是逐一指定路径。

 

> 　　require.config({
>
> 　　　　paths: {
>
> 　　　　　　"jquery": "lib/jquery.min",
> 　　　　　　"underscore": "lib/underscore.min",
> 　　　　　　"backbone": "lib/backbone.min"
>
> 　　　　}
>
> 　　});

 

另一种则是直接改变基目录（baseUrl）。

 

> 　　require.config({
>
> 　　　　baseUrl: "js/lib",
>
> 　　　　paths: {
>
> 　　　　　　"jquery": "jquery.min",
> 　　　　　　"underscore": "underscore.min",
> 　　　　　　"backbone": "backbone.min"
>
> 　　　　}
>
> 　　});

 

如果某个模块在另一台主机上，也可以直接指定它的网址，比如：

 

> 　　require.config({
>
> 　　　　paths: {
>
> 　　　　　　"jquery": "https://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min"
>
> 　　　　}
>
> 　　});

 

require.js要求，每个模块是一个单独的js文件。这样的话，如果加载多个模块，就会发出多次HTTP请求，会影响网页的加载速度。因此，require.js提供了一个[优化工具](http://requirejs.org/docs/optimization.html)，当模块部署完毕以后，可以用这个工具将多个模块合并在一个文件中，减少HTTP请求数。

 

五、AMD模块的写法

 

require.js加载的模块，采用AMD规范。也就是说，模块必须按照AMD的规定来写。

 

具体来说，就是模块必须采用特定的define()函数来定义。如果一个模块不依赖其他模块，那么可以直接定义在define()函数之中。

 

假定现在有一个math.js文件，它定义了一个math模块。那么，math.js就要这样写：

 

> 　　// math.js
>
> 　　define(function (){
>
> 　　　　var add = function (x,y){
>
> 　　　　　　return x+y;
>
> 　　　　};
>
> 　　　　return {
>
> 　　　　　　add: add
> 　　　　};
>
> 　　});

 

加载方法如下：

 

> 　　// main.js
>
> 　　require(['math'], function (math){
>
> 　　　　alert(math.add(1,1));
>
> 　　});

 

如果这个模块还依赖其他模块，那么define()函数的第一个参数，必须是一个数组，指明该模块的依赖性。

 

> 　　define(['myLib'], function(myLib){
>
> 　　　　function foo(){
>
> 　　　　　　myLib.doSomething();
>
> 　　　　}
>
> 　　　　return {
>
> 　　　　　　foo : foo
>
> 　　　　};
>
> 　　});

 

当require()函数加载上面这个模块的时候，就会先加载myLib.js文件。

 

六、加载非规范的模块

 

理论上，require.js加载的模块，必须是按照AMD规范、用define()函数定义的模块。但是实际上，虽然已经有一部分流行的函数库（比如jQuery）符合AMD规范，更多的库并不符合。那么，require.js是否能够加载非规范的模块呢？

 

回答是可以的。

 

这样的模块在用require()加载之前，要先用require.config()方法，定义它们的一些特征。

 

举例来说，underscore和backbone这两个库，都没有采用AMD规范编写。如果要加载它们的话，必须先定义它们的特征。

 

> 　　require.config({
>
> 　　　　shim: {
> 　　　　　　'underscore':{
> 　　　　　　　　exports: '_'
> 　　　　　　},
>
> 　　　　　　'backbone': {
> 　　　　　　　　deps: ['underscore', 'jquery'],
> 　　　　　　　　exports: 'Backbone'
> 　　　　　　}
>
> 　　　　}
>
> 　　});

 

require.config()接受一个配置对象，这个对象除了有前面说过的paths属性之外，还有一个shim属性，专门用来配置不兼容的模块。具体来说，每个模块要定义（1）exports值（输出的变量名），表明这个模块外部调用时的名称；（2）deps数组，表明该模块的依赖性。

 

比如，jQuery的插件可以这样定义：

 

> 　　shim: {
>
> 　　　　'jquery.scroll': {
>
> 　　　　　　deps: ['jquery'],
>
> 　　　　　　exports: 'jQuery.fn.scroll'
>
> 　　　　}
>
> 　　}

 

七、require.js插件

 

require.js还提供一系列[插件](https://github.com/jrburke/requirejs/wiki/Plugins)，实现一些特定的功能。

 

domready插件，可以让回调函数在页面DOM结构加载完成后再运行。

 

> 　　require(['domready!'], function (doc){
>
> 　　　　// called once the DOM is ready
>
> 　　});

 

text和image插件，则是允许require.js加载文本和图片文件。

 

> 　　define([
>
> 　　　　'text!review.txt',
>
> 　　　　'image!cat.jpg'
>
> 　　　　],
> 　　　　function(review,cat){
>
> 　　　　　　console.log(review);
>
> 　　　　　　document.body.appendChild(cat);
>
> 　　　　}
>
> 　　);

 

类似的插件还有json和mdown，用于加载json文件和markdown文件。（完）

 

另一个人的概括(有点简单)：

AMD就只有一个接口：define(id?,dependencies?,factory);

 

它要在声明模块的时候制定所有的依赖(dep)，并且还要当做形参传到factory中，像这样：

 

```
1 define(['dep1','dep2'],function(dep1,dep2){...});
```

 

要是没什么依赖，就定义简单的模块，下面这样就可以啦：

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 define(function(){
2     var exports = {};
3     exports.method = function(){...};
4     return exports;
5 });
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

咦，这里有define，把东西包装起来啦，那Node实现中怎么没看到有define关键字呢，它也要把东西包装起来呀，其实吧，只是Node隐式包装了而已.....

这有AMD的WIKI中文版，讲了很多蛮详细的东西，用到的时候可以查看：[AMD的WIKI中文版](https://github.com/amdjs/amdjs-api/wiki/AMD-(%E4%B8%AD%E6%96%87%E7%89%88))

**三、CMD**

大名远扬的玉伯写了seajs，就是遵循他提出的CMD规范，与AMD蛮相近的，不过用起来感觉更加方便些，最重要的是中文版，应有尽有：[seajs官方doc](http://seajs.org/docs/#docs)

```
1 define(function(require,exports,module){...});
```

用过seajs吧，这个不陌生吧，对吧。

前面说AMD，说RequireJS实现了AMD，CMD看起来与AMD好像呀，那RequireJS与SeaJS像不像呢？

虽然CMD与AMD蛮像的，但区别还是挺明显的，官方非官方都有阐述和理解，我觉得吧，说的都挺好：

[官方阐述SeaJS与RequireJS异同](https://github.com/seajs/seajs/issues/277)

[SeaJS与RequireJS的最大异同（这个说的也挺好）](http://www.douban.com/note/283566440/)



 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

AMD则是通过<script>标签引入RequireJs。