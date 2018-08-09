# JavaScript原生

## 1.实现jquery中的`## 

```javascript
function $(selector){
	let element = document.querySelectorAll(selector)
	return Array.prototype.slice.call(element)
}
```

## 2.js原生ajax

```
    function ajax (info) {
      // var APIROOT = 'https://college.baicizhan.com/api'
      
      var url = info.path
      var method = info.method
      var data = info.data
      var xhr = new XMLHttpRequest()
      var timedout = false
      var timer = setTimeout(function () {
        timedout = true
        xhr.abort()
      }, 3000)
      xhr.onreadystatechange = function () {
        if (xhr.readyState !== 4) return
        if (timedout) {
          info.fail('timeout')
          return
        }
        clearTimeout(timer)
        if (xhr.status === 200) {
          var res = JSON.parse(xhr.responseText || '{}')
          if (res.code === 200) {
            info.success(res)
          } else {
            info.fail(res)
          }
        }
      }
      switch (method) {
        case 'POST': {
          xhr.open('POST', url)
          xhr.setRequestHeader('Content-type', 'application/json; charset=utf-8')
          xhr.send(JSON.stringify(data))
          break
        }
        case 'GET': {
          var query = '?'
          for (var key in data) {
            query += key + '=' + encodeURIComponent(data[key]) + '&'
          }
          xhr.open('GET', url + query.substring(0, query.length - 1))
          xhr.send()
          break
        }
      }
    }
```

## 3
[]==[]//false<br>
解析：两个引用类型， ==比较的是引用地址<br>
巩固：[ ]== `![]` 
     (1) ! 的优先级高于== ，右边Boolean([])是true,取返等于 false
     (2)一个引用类型和一个值去比较 把引用类型转化成值类型，左边0
     (3)所以 0 == false  答案是true
     
## 4

```
var a = /123/,
b = /123/;
a == b
a === b
```
```
答案：false, false
解析：正则是对象，引用类型，相等（==）和全等（===）都是比较引用地址
```
## 5
```
var a = [1, 2, 3],
b = [1, 2, 3],
c = [1, 2, 4]
a ==  b
a === b
a >   c
a <   c
```

```
答案：false, false, false, true
解析：相等（==）和全等（===）还是比较引用地址
     引用类型间比较大小是按照字典序比较，就是先比第一项谁大，相同再去比第二项。
```

## 6如何渲染几万条数据并不卡住界面
这道题考察了如何在不卡住页面的情况下渲染数据，也就是说不能一次性将几万条都渲染出来，而应该一次渲染部分 DOM，那么就可以通过 requestAnimationFrame 来每 16 ms 刷新一次。

```
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <ul>控件</ul>
  <script>
    setTimeout(() => {
      // 插入十万条数据
      const total = 100000
      // 一次插入 20 条，如果觉得性能不好就减少
      const once = 20
      // 渲染数据总共需要几次
      const loopCount = total / once
      let countOfRender = 0
      let ul = document.querySelector("ul");
      function add() {
        // 优化性能，插入不会造成回流
        const fragment = document.createDocumentFragment();
        for (let i = 0; i < once; i++) {
          const li = document.createElement("li");
          li.innerText = Math.floor(Math.random() * total);
          fragment.appendChild(li);
        }
        ul.appendChild(fragment);
        countOfRender += 1;
        loop();
      }
      function loop() {
        if (countOfRender < loopCount) {
          window.requestAnimationFrame(add);
        }
      }
      loop();
    }, 0);
  </script>
</body>
</html>
</body>
```

##7 offsetWidth/offsetHeight,clientWidth/clientHeight与scrollWidth/scrollHeight的区别
* offsetWidth/offsetHeight返回值包含content + padding + border，效果与e.getBoundingClientRect()相同
* clientWidth/clientHeight返回值只包含content + padding，如果有滚动条，也不包含滚动条
* scrollWidth/scrollHeight返回值包含content + padding + 溢出内容的尺寸

##8 Promise原理
首先先定义一个Promise函数，并且接受一个executor执行函数，将resolve和reject传参传进去，定义私有属性status，value，reason，onResolvedCallbacks，onRejectedCallbacks。promise有三个状态：初始状态为pending，成功为resolved，失败为reject。对promise的prototype挂载一个then方法（构造函数.prototype===构造函数的实例）,then方法中传入一个成功的回掉和失败毁掉，若成功去成功回调，失败取失败回调。onResolvedCallbacks存放成功异步函数的回调，存放失败异步函数回调。

### 初步实现

```
function mypromise(constructor){
	let self=this
	self.status="pending"
	self.value=undefined
	self.reason=undefined
	function resolve(value){
		if(self==="pending"){
			self.value=value;
			self.status="resolved"
		}
	}
	function reject(reason){
		if(self.status==="pending"){
			self.reason=reason
			self.status="rejected"
		}
	}
	try{
		constructor(resolve,reject)
	}catch(e){
		reject(e)
	}
	}
	//定义then方法
	mypromise.prototype.then=function(onFullfilled,onRejected){
		let self = this
		switch(self.status){
			case "resolved":
				onFullfilled(self.value)
				break
			case "rejected":
				onRejected(self.reason)
				break
			default:	
		}
	}
```
上述就是一个初始版本的myPromise，在myPromise里发生状态改变，然后在相应的then方法里面根据不同的状态可以执行不同的操作。但无法处理异步resolve

### 异步实现

为了处理异步resolve，我们修改myPromise的定义，用2个数组onFullfilledArray和onRejectedArray来保存异步的方法。在状态发生改变时，一次遍历执行数组中的方法

```
function mypromise(constructor){
    let self=this;
    self.status="pending" //定义状态改变前的初始状态
    self.value=undefined;//定义状态为resolved的时候的状态
    self.reason=undefined;//定义状态为rejected的时候的状态
    self.onFullfilledArray=[];
    self.onRejectedArray=[];
    function resolve(value){
    	if(self.status==="pending"){
    		self.value=value
    		self.status="resolve"
    		self.onFullfilledArray.forEach((item)=>{
    			item(self.value)
    		})
    	}
    }
    function reject(reason){
    	if(self.status==="pending"){
    		self.reason=reason
    		self.status="rejected"
    		self.onRejectedArray.forEach((item)=>{
    			item(self.reason)
    		})
    	}
    }
    try{
    	constructor(resolve,reject)
    }catch(e){
    	reject(e)
    }
}
//then实现
mypromise.prototype.then=function(onFullfilled, onRejected){
	let self = this
	switch(self.status){
		case "pending":
			self.onFullfilledArray.push(function(){
				onFullfilled(self.value)
			})
			self.onRejectedArray.push(funciton(){
				onRejected(self.reason)
			})
		case "resolved":
				onFullfilled(self.value)
				break
			case "rejected":
				onRejected(self.reason)
				break
			default:
				
	}
}

```

这样，通过两个数组，在状态发生改变之后再开始执行，这样可以处理异步resolve无法调用的问题。这个版本的myPromise就能处理所有的异步，那么这样做就完整了吗？

没有，我们做Promise/A+规范的最大的特点就是链式调用，也就是说then方法返回的应该是一个promise。

### then方法实现链式调用

```
mypromise.prototype.then=function(onFullfilled,onRejected){
	let self=this
	let promise2
	switch(self.status){
		case "pending":
			promise2 = new promise((resolve,reject)=>{
				self.onFullfilledArray.push(function(){
					resolve(onFullfilled(self.value))
				})
				self.onRejectArray.push(function(){
					resolve(onRejected(self.value))
				})
			})
		case "resolved":
			promise = new mypromise((resolve,reject)=>{
				try(
					resolve(onFullfilled(self.value))
				)catch(e){
					reject(e)
				}
			})
		case "rejected":
			promise2 = new mypromise((resolve,reject)=>{
				try(
					resolve(onRejectd(self.reason))
				)catch(e){
					reject(e)
				}
			})		
			
	}
	return promise2
}
```

### promise使用
基本使用

```
let a =new promise((resolve,reject)=>{
	resolve('11')//将结果传入resolve，reject，结果可以在then中获取，以实现链式调用
	reject('22')
})
a.then((data)=>{console.log(data)},(data)=>{console.log(data)})//then接受两个参数，第一个是resolve，第二个是reject
//输出 ‘11’ ‘22’
```

#### 使用promis封装ajax

```
const getJSON = function(url) {
  const promise = new Promise(function(resolve, reject){
    const handler = function() {
      if (this.readyState !== 4) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
    const client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
```

#### 异步加载图片

```
function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    const image = new Image();

    image.onload = function() {
      resolve(image);
    };

    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };

    image.src = url;
  });
}
var loadImage1 = loadImageAsync(url);
loadImage1.then(function success() {
    console.log("success");
}, function fail() {
    console.log("fail");
});
```

## JavaScript原型
通过new function()创建的对象都是函数对象，其他的都是普通对象。
### 构造函数

```
function Person(name, age, job) {
 this.name = name;
 this.age = age;
 this.job = job;
 this.sayName = function() { alert(this.name) } 
}
var person1 = new Person('Zaxlct', 28, 'Software Engineer');
var person2 = new Person('Mick', 23, 'Doctor');
```
上面的例子中 person1 和 person2 都是 Person 的实例。这两个实例都有一个 constructor （构造函数）属性，该属性（是一个指针）指向 Person。 即

```
  console.log(person1.constructor == Person); //true
  console.log(person2.constructor == Person); //true
```

**实例的构造函数属性(constructor)指向构造函数**
### 原型对象
每个对象都有 __proto__ 属性，但只有函数对象才有 prototype 属性

在默认情况下，所有的原型对象都会自动获得一个 constructor（构造函数）属性，这个属性（是一个指针）指向 prototype 属性所在的函数（Person）


**结论：原型对象（Person.prototype）是 构造函数（Person）的一个实例。**

即

```
person1.constructor == Person
Person.prototype.constructor == Person
```

原型对象其实就是普通对象（但function.prototype除外，它是函数对象向，而且他没有prototype属性）

```
function Person(){};
 console.log(Person.prototype) //Person{}
 console.log(typeof Person.prototype) //Object
 console.log(typeof Function.prototype) // Function，这个特殊
 console.log(typeof Object.prototype) // Object
 console.log(typeof Function.prototype.prototype) //undefined
```
原型对象主要用来继承，举个例子

```
var Person = function(name){
    this.name = name; //
  };
  Person.prototype.getName = function(){
    return this.name;  //
  }
  var person1 = new person('Mick');
  person1.getName(); //Mick
```
从这个例子可以看出，通过给 Person.prototype 设置了一个函数对象的属性，那有 Person 的实例（person1）出来的普通对象就继承了这个属性。具体是怎么实现的继承，就要讲到下面的原型链了。
### `__proto`
JS 在创建对象（不论是普通对象还是函数对象）的时候，都有一个叫做`__proto__` 的内置属性，用于指向创建它的构造函数的原型对象。
对象 person1 有一个 `__proto__`属性，创建它的构造函数是 Person，构造函数的原型对象是 Person.prototype ，所以：
person1.__proto__ == Person.prototype
通过上面将讲解可以得到：

```
Person.prototype.constructor == Person;
person1.__proto__ == Person.prototype;
person1.constructor == Person;
```
### 构造器
创建一个对象const obj ={}等同于const obj=new object(),所以 obj.constructor ===object,obj.__proto__===object.prototype

同理，可以创建对象的构造器不仅仅有 Object，也可以是 Array，Date，Function等。
所以我们也可以构造函数来创建 Array、 Date、Function

```
var b = new Array();
b.constructor === Array;
b.__proto__ === Array.prototype;

var c = new Date(); 
c.constructor === Date;
c.__proto__ === Date.prototype;

var d = new Function();
d.constructor === Function;
d.__proto__ === Function.prototype;
```

**注意事项：Object.prototype.__proto__ === null**

### 函数对象
所有函数对象的proto都指向Function.prototype，它是一个空函数（Empty function）

```
Number.__proto__ === Function.prototype  // true
Number.constructor == Function //true

Boolean.__proto__ === Function.prototype // true
Boolean.constructor == Function //true

String.__proto__ === Function.prototype  // true
String.constructor == Function //true

// 所有的构造器都来自于Function.prototype，甚至包括根构造器Object及Function自身
Object.__proto__ === Function.prototype  // true
Object.constructor == Function // true

// 所有的构造器都来自于Function.prototype，甚至包括根构造器Object及Function自身
Function.__proto__ === Function.prototype // true
Function.constructor == Function //true

Array.__proto__ === Function.prototype   // true
Array.constructor == Function //true

RegExp.__proto__ === Function.prototype  // true
RegExp.constructor == Function //true

Error.__proto__ === Function.prototype   // true
Error.constructor == Function //true

Date.__proto__ === Function.prototype    // true
Date.constructor == Function //true
```

**所有的构造器都来自于 Function.prototype，甚至包括根构造器Object及Function自身。所有构造器都继承了·Function.prototype·的属性及方法。如length、call、apply、bind**

但是Funciton.prototype也是一个对象，也是object的实例。即console.log(Function.prototype.__proto__ === Object.prototype) // true

### 举个列子

```
function person(){}
let per1 = new person()
```

可以得出

```
per1.contructor = person
person.prototype.contructor = person
per1.__proto__=person.prototype

person.__proto__=Function.prototype
person.constructor = Function
person.prototype.__proto__=Object.prototype

Function.__proto__=Function.prototype
Function.constructor = Function

Object.__proto__=Function.prototype
Object.contructor=Function
//Object 是函数对象，是通过new Function()创建的，所以Object.__proto__指向Function.prototype。

Function.prototype.__proto__===Object.prototype
Object.prototype.__proto__===null


```

## 9JavaScript闭包

理解闭包的关键在于：外部函数调用之后其变量对象本应该被销毁，但闭包的存在使我们仍然可以访问外部函数的变量对象。

闭包是在某个作用域内定义的函数，它可以访问这个作用域内的所有变量。闭包作用域链通常包括三个部分：

1. 函数本身作用域。
2. 闭包定义时的作用域。
3. 全局作用域。

闭包常见用途：

1. 创建特权方法用于访问控制
2. 事件处理程序及回调
3. 将变量始终保存在内存中，不会因为执行环境的销毁而销毁

通常，函数的作用域及其所有变量都会在函数执行结束后被销毁。但是，在创建了一个闭包以后，这个函数的作用域就会一直保存到闭包不存在为止。

```
function adder(x){
  return function(y){
  	return x+y
  }
}
var add5 = makeAdder(5);//5保存到参数y
var add10 = makeAdder(10);//10保存到参数y

console.log(add5(2));  // 输出7；2保存到参数x
console.log(add10(2)); // 12。  2保存到参数x

// 释放对闭包的引用
add5 = null;
add10 = null;
```
匿名函数的执行环境具有全局性，因此其 this 对象通常指向 window。

```
var name = "The Window";

var obj = {
  name: "My Object",
  
  getName: function(){
    return function(){
      return this.name;
    };
  }
};

console.log(obj.getName()());  // The Window
```

但当这样时, 将当前函数的this绑定到变量上时：

```
var name = "The Window";

var obj = {
  name: "My Object",
  
  getName: function(){
    var that = this;
    return function(){
      return that.name;
    };
  }
};

console.log(obj.getName()());  // My Object

```

### 闭包应用
在我看来，闭包作用可以是模块化编程，之前头条的面试题可以使用闭包这么实现

```
    function repeat(f,count,interval) {
            let i  = 0;
            let timer
            let re = function (str) {
                timer=setInterval(function () {
                    i=i+1
                    console.log(i)
                    console.log(count)
                    if(i===count){
                        clearInterval(timer)
                    }
                    f(str)
                },interval)
            }
            return{
                re:re
            }
    }
    let a = new repeat(alert,4,3000)
    a.re('jha')
```

#### 用法一：模块级作用域

```
(function(){
    var now = new Date();
    if (now.getMonth() == 0 && now.getDate() == 1) {
        alert('Happy new Year!');
    }
})();
```

#### 用法二：创建私有变量

```
(funcion() {
    var a = 1;
    setA = function(val){
        a = val;
    };
    getA = function(){
        return a;
    };
})();
console.log(a); //报错
console.log(getA()); //1
setA(2);
console.log(getA()); //2

```

## 10 apply call  bind
[https://www.cnblogs.com/Shd-Study/archive/2017/03/16/6560808.html](https://www.cnblogs.com/Shd-Study/archive/2017/03/16/6560808.html)
通俗易懂

## 相关Javascript代码
## 获取url中的query

```
  parseSearch: function (search) {
    let res = {}
    search = search.substr(1)
    search.split('&').forEach(item => {
      let splitted = item.split('=')
      if (splitted.length === 2) {
        res[splitted[0]] = splitted[1]
      }
    })
    return res
  }
```

## 实现倒计时

```
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>TEst</title>
</head>
<body>

    <span id="target"></span>


<script type="text/javascript">
    // 为了简化。每月默认30天
    function getTimeString() {
        var start = new Date();
        var end = new Date(start.getFullYear() + 1, 0, 1);
        var elapse = Math.floor((end - start) / 1000);

        var seconds = elapse % 60 ;
        var minutes = Math.floor(elapse / 60) % 60;
        var hours = Math.floor(elapse / (60 * 60)) % 24;
        var days = Math.floor(elapse / (60 * 60 * 24)) % 30;
        var months = Math.floor(elapse / (60 * 60 * 24 * 30)) % 12;
        var years = Math.floor(elapse / (60 * 60 * 24 * 30 * 12));

        return start.getFullYear() + '年还剩' + years + '年' + months + '月' + days + '日'
            + hours + '小时' + minutes + '分' + seconds + '秒';
    }

    function domText(elem, text) {
        if (text == undefined) {

            if (elem.textContent) {
                return elem.textContent;
            } else if (elem.innerText) {
                return elem.innerText;
            }
        } else {
            if (elem.textContent) {
                elem.textContent = text;
            } else if (elem.innerText) {
                elem.innerText = text;
            } else {
                elem.innerHTML = text;
            }
        }
    }

    var target = document.getElementById('target');

    setInterval(function () {
        domText(target, getTimeString());
    }, 1000)
</script>

</body>
</html>
```
##判断某数据类型，全浏览器兼容
```
Object.prototype.toString.call(arg)==="[object Type]"
```

##根据可视窗口宽度自动调节html根元素字体大小以调整rem大小，实现自适应页面

```
(function (win, doc) {
      var docEle = doc.documentElement,
        evt = 'onorientationchange' in window ? 'orientationchange' : 'resize',
        fn = function () {
          setTimeout(function () {
            var width = docEle.clientWidth
            width && (docEle.style.fontSize = 100 * (width / 750) + 'px')
          }, 170)
        }
      win.addEventListener(evt, fn, false)
      doc.addEventListener('DOMContentLoaded', fn, false)
    }(window, document))
```

## 事件委托

事件委托原理：事件委托就是利用事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。何为事件冒泡呢？就是事件从最深的节点开始，然后逐步向上传播事件，举个例子：页面上有这么一个节点树，div>ul>li>a;比如给最里面的a加一个click点击事件，那么这个事件就会一层一层的往外执行，执行顺序a>li>ul>div，有这样一个机制，那么我们给最外面的div加点击事件，那么里面的ul，li，a做点击事件的时候，都会冒泡到最外层的div上，所以都会触发，这就是事件委托，委托它们父级代为执行事件

js一般写法

```
  window.onload = function(){
     var oUl = document.getElementById('ul');
     var aLi = oUl.getElementsByTagName('li');

     for(var i=0; i<aLi.length; i++){
         aLi[i].onmouseover = function(){
             this.style.background = 'red';
         }
         aLi[i].onmouseout = function(){
             this.style.background = ' ';
         }
     }
  }
```

## 事件委托写法

```
window.onload = function(){
    var oUl = document.getElementById('ul');
    var aLi = oUl.getElementsByTagName('li');  
    /*这里用到事件源：event对象， 事件源，不管在哪个事件中，只要你操作的那个元素就是事件源
               ie： window.event.srcElent
            标准下： event.target
         nodeName: 找到元素的标签名；
     */
    oUl.onmouseover = function(ev) {
        var ev = ev||window.event;
        var target = ev.target || ev.srcElement;
       
        // console.log(target.innerHTML);
       
        if(target.nodeName.toLowerCase() == "li"){
           target.style.background = 'red';
        }
    }
    oUl.onmouseout = function(ev) {
        var ev = ev || window.event;
        var target = ev.target|| ev.srcElement;
       
        if(target.nodeName.toLowerCase() == 'li'){
           target.style.background = ' ';
        }
    }
}
```

## bind()简单原理

```
Function.prototype.testBind = function (scope) {
    var fn = this;                                // this 指向的是调用testBind方法的一个函数
    return function () {
        return fn.apply(scope, arguments);
    }
};
下面是测试的例子：
var foo = {x: "Foo "};
var bar = function (str) {
    console.log(this.x+(arguments.length===0?'':str));
};

bar();                                   // undefined

var testBindBar = bar.testBind(foo);     // 绑定 foo
testBindBar("Bar!");                     // Foo Bar!

```

## ajax的优点和缺点
优点：

```
（1）最大的优点就是页面无刷新，用户的体验非常好；

（2） 使用异步方式与服务器通信，具有更加迅速的相应能力；

（3）可以把以前的一些服务器负担的工作转嫁到客户端，利用客户端限制的能力来处理，减轻服务器和带宽的负担，节约空间和带宽租用成本，并且减轻服务器的负担，Ajax的原则是“按需取数据”，可以最大程度地减少冗余请求和相应对服务器造成的负担；

（4）基于标准化的并被广泛支持的技术，不需要下载插件和小程序；
```
缺点

```
（1）Ajax不支持浏览器返回按钮；

（2）安全问题，Ajax暴露了与服务器交互的细节；

（3）对搜索引擎的支持比较弱；

（4）破坏了程序的异常机制；

（5）不容易调试。
```
## mouseover/mouseout与mouseenter/mouseleave的区别与联系
1. mouseover/mouseout是标准事件，所有浏览器都支持；mouseenter/mouseleave是IE5.5引入的特有事件后来被DOM3标准采纳，现代标准浏览器也支持
2. mouseover/mouseout是冒泡事件；mouseenter/mouseleave不冒泡。需要为多个元素监听鼠标移入/出事件时，推荐mouseover/mouseout托管，提高性能
3. 标准事件模型中event.target表示发生移入/出的元素,vent.relatedTarget对应移出/如元素；在老IE中event.srcElement表示发生移入/出的元素，event.toElement表示移出的目标元素，event.fromElement表示移入时的来源元素

例子：鼠标从div#target元素移出时进行处理，判断逻辑如下：

```
<div id="target"><span>test</span></div>

<script type="text/javascript">
var target = document.getElementById('target');
if (target.addEventListener) {
  target.addEventListener('mouseout', mouseoutHandler, false);
} else if (target.attachEvent) {
  target.attachEvent('onmouseout', mouseoutHandler);
}

function mouseoutHandler(e) {
  e = e || window.event;
  var target = e.target || e.srcElement;

  // 判断移出鼠标的元素是否为目标元素
  if (target.id !== 'target') {
    return;
  }

  // 判断鼠标是移出元素还是移到子元素
  var relatedTarget = event.relatedTarget || e.toElement;
  while (relatedTarget !== target
    && relatedTarget.nodeName.toUpperCase() !== 'BODY') {
    relatedTarget = relatedTarget.parentNode;
  }

  // 如果相等，说明鼠标在元素内部移动
  if (relatedTarget === target) {
    return;
  }

  // 执行需要操作
  //alert('鼠标移出');

}
</script>
```

## Event和Event.target
* altKey	 返回当事件被触发时，"ALT" 是否被按下。
	 button	 返回当事件被触发时，哪个鼠标按钮被点击。
	 clientX	返回当事件被触发时，鼠标指针的水平坐标。
	 clientY	返回当事件被触发时，鼠标指针的垂直坐标。
	 ctrlKey	返回当事件被触发时，"CTRL" 键是否被按下。
	 metaKey	返回当事件被触发时，"meta" 键是否被按下。
	 relatedTarget	返回与事件的目标节点相关的节点。
	 screenX	返回当某个事件被触发时，鼠标指针的水平坐标。
	 screenY	返回当某个事件被触发时，鼠标指针的垂直坐标。
	 shiftKey	返回当事件被触发时，"SHIFT" 键是否被按下。

	 onabort	图像的加载被中断。
	 onblur	元素失去焦点。
	 onchange	域的内容被改变。
	 onclick	当用户点击某个对象时调用的事件句柄。
	 ondblclick	当用户双击某个对象时调用的事件句柄。
	 onerror	在加载文档或图像时发生错误。
	 onfocus	元素获得焦点。
	 onkeydown	某个键盘按键被按下。
	 onkeypress	某个键盘按键被按下并松开。
	 onkeyup	某个键盘按键被松开。
	 onload	一张页面或一幅图像完成加载。
	 onmousedown	鼠标按钮被按下。
	 onmousemove	鼠标被移动。
	 onmouseout	鼠标从某元素移开。
	 onmouseover	鼠标移到某元素之上。
	 onmouseup	鼠标按键被松开。
	 onreset	重置按钮被点击。
	 onresize	窗口或框架被重新调整大小。
	 onselect	文本被选中。
	 onsubmit	确认按钮被点击。
	 onunload	用户退出页面。

### 标准event属性

* bubbles	返回布尔值，指示事件是否是起泡事件类型。
	 cancelable	返回布尔值，指示事件是否可拥可取消的默认动作。
	 currentTarget	返回其事件监听器触发该事件的元素。
	 eventPhase	返回事件传播的当前阶段。
	 target	返回触发此事件的元素（事件的目标节点）。
	 timeStamp	返回事件生成的日期和时间。
	 type	返回当前 Event 对象表示的事件的名称

### 标准event方法
* initEvent()	初始化新创建的 Event 对象的属性。
	 preventDefault()	通知浏览器不要执行与事件关联的默认动作。
	 stopPropagation()	不再派发事件。

## 函数声明与表达式的区别

函数声明的语法：

```
function functionName(arg0,arg1){
//函数体
}
```

关于函数声明，他最重要的一个特征，就是函数声明提升，意思是会在执行代码前读取函数声明。这就意味着可以把函数声明放在调用它的语句后。如：

```
a();
function a(){alert("a");}
```

第二种创建函数的方式是函数表达式：

```
//以下为函数表达式
var myFunc = function(){};
myFunc(function(){
  return function(){};
});

(function namedFunctionExpression () { })();//立即执行函数

+function(){}();
-function(){}();
!function(){}();
~function(){}();
```

函数表达式与其他表达式一样，在使用前必须赋值

#### 关于区别
1、函数声明中函数名是必须的，函数表达式中则是可选的。

2、用函数声明定义的函数，函数可以在函数声明之前调用，而用函数表达式定义的函数则只能在声明之后调用。

根本原因在于解析器对于这两种定义方式读取的顺序不同：解析器会实现读取函数声明，即函数声明放在任意位置都可以被调用；而对于函数表达式，解析器只有在读到函数表达式所在那一行时才会开始执行（详情请看第一部分“函数定义的方式”）。

## 深拷贝实现



```javascript
    //定义检测数据类型的功能函数
    function checkedType(target) {
      return Object.prototype.tostring.call(target).slice(8, -1)
    }
    //实现深度克隆---对象/数组
    function clone(target) {
      //判断拷贝的数据类型
      //初始化变量result 成为最终克隆的数据
      let result, targetType = checkedType(target)
      if (targetType === 'object') {
        result = {}
      } else if (targetType === 'Array') {
        result = []不
      } else {
        return target
      }
      //遍历目标数据
      for (let i in target) {
        //获取遍历数据结构的每一项值。
        let value = target[i]
        //判断目标结构里的每一值是否存在对象/数组
        if (checkedType(value) === 'Object' ||
          checkedType(value) === 'Array') { //对象/数组里嵌套了对象/数组
          //继续遍历获取到value值
          result[i] = clone(value)
        } else { //获取到value值是基本的数据类型或者是函数。
          result[i] = value;
        }
      }
      return result
    }

```

## JS拖拽功能的实现

```
首先是三个事件，分别是mousedown，mousemove，mouseup
当鼠标点击按下的时候，需要一个tag标识此时已经按下，可以执行mousemove里面的具体方法。


clientX，clientY标识的是鼠标的坐标，分别标识横坐标和纵坐标，并且我们用offsetX和offsetY来表示元素的元素的初始坐标，移动的举例应该是：
鼠标移动时候的坐标-鼠标按下去时候的坐标。
也就是说定位信息为：
鼠标移动时候的坐标-鼠标按下去时候的坐标+元素初始情况下的offetLeft.


还有一点也是原理性的东西，也就是拖拽的同时是绝对定位，我们改变的是绝对定位条件下的left
以及top等等值。

```

## js中的job queue

js中的事件执行，首先执行主线程中的同步任务，当主线程任务执行完毕后，再从event loop(job queue)中读取异步任务

在Job queue中的队列分为两种类型：macro-task和microTask。我们举例来看执行顺序的规定，我们设

* macro-task队列包含任务: a1, a2 , a3(宏任务)
* micro-task队列包含任务: b1, b2 , b3（微任务）

执行顺序为，首先执行marco-task队列开头的任务，也就是 a1 任务，执行完毕后，在执行micro-task队列里的所有任务，也就是依次执行b1, b2 , b3 ，执行完后清空micro-task中的任务，接着执行marco-task中的第二个任务，依次循环。

macro-task队列真实包含任务：

script(主程序代码),setTimeout, setInterval, setImmediate, I/O, UI rendering

micro-task队列真实包含任务：

process.nextTick, Promises, Object.observe, MutationObserver

由此我们得到的执行顺序应该为：

script(主程序代码)—>process.nextTick—>Promises...——>setTimeout——>setInterval——>setImmediate——> I/O——>UI rendering

## 实现node中的Events模块

实现类似于

```
var events=require('events');
var eventEmitter=new events.EventEmitter();
eventEmitter.on('say',function(name){
    console.log('Hello',name);
})
eventEmitter.emit('say','Jony yu');
```

实现

```
function Events(){
	let self = this 
	self.on = function(events,callback){
		if(!self.handles){
			self.handles={}
		}
		if(!self.handles[events]){
			self.handles[events]=[]
		}
		this.handles[events].push(callback)
		
	}
	self.emit=function(events,payload){
		if(self.handles[events]){
			self.handle[evnets].forEache((item)=>{
				item(payload)
			})
		}
	}
	return self
}
```

## cookie

### coookie的构成

```
1. 名称 2.值 3.域 4.路径 5.失效时间 6.安全标志
例子：一个响应头
HTTP/1.1 200 OK
content-type:text/html
Set-Coookie:name=value;domain=.wrox.com;exoires=Mon,22-Jan-07 07:10:24;
```

### javascript中的cookie
使用`document.cookie`

1.使用获取cookie时返回：‘name=value1;name2=value2;name3=value3’。返回一系列由分号分隔开的名值对

2.使用设置cookie：

```
设置的cookie格式如下：
document.cookie="name=value;expires=有效时间;path=domain_path;domain=domain_name;secure"
设置secure表明这个cookie只有通过ssl连接才能传输
```

### 封装cookie
有JavaScript中读写cookie不方便，所以需要封装方法

```
红书p631
```

## apply和call的实现原理

```
Function.prototype.call2=function(context){
    var context = context || window;
	context.fn = this;
    var args = [];
	for(var i = 1, len = arguments.length; i < len; i++) {
	args.push(arguments[i])
    }
    context.fn(...args)
    delete context.fn
}
```
## 实现拖拽功能

```java
<div class="wrapper">
  <div id="dragCircle">来拖我呀</div>  
</div>

function posLeft(obj){
    var iLeft = 0;
    while(obj){
        iLeft += obj.offsetLeft;
        obj = obj.offsetParent;
        if(obj && obj!=document.body && obj!=document.documentElement){
            iLeft += getStyle(obj, 'borderLeftWidth');
        }
    }
    return iLeft;
}
function posTop(obj){
    var iTop = 0;
    while(obj){
        iTop += obj.offsetTop;
        obj = obj.offsetParent;
        if(obj && obj!=document.body && obj!=document.documentElement){
            iTop += getStyle(obj, 'borderTopWidth');
        }
    }
    return iTop;
}
function getStyle(obj,attr){
    if(obj.currentStyle){
        return parseFloat( obj.currentStyle[attr]) || 0;
    }
    return parseFloat( getComputedStyle(obj)[attr]) || 0;
}
window.onload = function() {
  var dragCircle = document.getElementById('dragCircle');
  var wrapper = document.querySelector(".wrapper")
  var range = {
    left    : posLeft(wrapper),  
    right   : posLeft(wrapper) + wrapper.offsetWidth - dragCircle.offsetWidth,
    top     : posTop(wrapper),
    bottom  : posTop(wrapper) + wrapper.offsetHeight - dragCircle.offsetHeight
  };
// 获取鼠标点击时在div中的相对位置
    dragCircle.onmousedown = function(ev) {
        var ev = ev || window.event; 
        var relaX = ev.clientX - this.offsetLeft;
        var relaY = ev.clientY - this.offsetTop;

// 获取当前鼠标位置，减去与div的相对位置得到当前div应该被拖拽的位置
        document.onmousemove = function(ev) {
            var ev = ev || window.event;
          if(ev.clientX - relaX < range.left) {
           dragCircle.style.left = range.left + 'px'
          } else if (ev.clientX - relaX > range.right) {
            dragCircle.style.left = range.right + 'px'
          } else {
           dragCircle.style.left = ev.clientX - relaX + 'px'
          }
          if(ev.clientY - relaY < range.top) {
           dragCircle.style.top = range.top + 'px'
          } else if (ev.clientY - relaY > range.bottom) {
            dragCircle.style.top = range.bottom + 'px'
          } else {
           dragCircle.style.top = ev.clientY - relaY + 'px'
          }
        };
        document.onmouseup = function(ev) {
          var ev = ev || window.event;
          document.onmousemove = null;
          document.onmouseup = null;
         }
     }
}
```

## 前后端数据传输的几种方法



```javascript
第一种：ajax

传给后台的数据通过json封装起来，再用ajax将json传到后台

2、通过form表单的action传值

一般情况下数值在传给后台之前需要校验，可以在form中的onsubmit调用js方法进行校验，当js方法返回值为true时，触发action，当js方法返回值为false时，action不触发。这样处理的好处在于当用户输入不正确时，不会刷新页面，表单仍然会保留用户之前的输入

3. 通过dom获取标签，触发标签的submit方法，直接提交数据到后台
```



