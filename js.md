#JavaScript原生恶补！
### 1.实现jquery中的$

```javascript
function $(selector){
	let element = document.querySelectorAll(selector)
	return Array.prototype.slice.call(element)
}
```

### 2.js原生ajax

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

###3
[]==[]//false<br>
解析：两个引用类型， ==比较的是引用地址<br>
巩固：[ ]== `![]` 
     (1) ! 的优先级高于== ，右边Boolean([])是true,取返等于 false
     (2)一个引用类型和一个值去比较 把引用类型转化成值类型，左边0
     (3)所以 0 == false  答案是true
     
### 4

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
### 5
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



