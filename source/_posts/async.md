---
title: Promise/Generator/async 的理解
date: 2019-01-07 15:01:57
tags:
---
# Promise => Generator => async

## Promise
早期js的异步回调方案采用的是回调函数的方法，例如：

    fs.readFile(path, function (err,bytesRead) {
        if (err) throw err;
        console.log(bytesRead);
    })
但是这种模式，在回调比较多的情况下，容易引起可怕的“回调地狱”，造成回调里面套用回调，回调里面再调用回调......这样做的后果就是代码的极不易阅读，而且也容易出错。<br/>
es6提出来的Promise，应运而生，是一种异步编程的新的方案。我理解的Promise简单来说就是一个容器，里面装着一些未来才会结束的一些操作结果，本质上是一个对象，提供一些api供我们使用。<br/>
Promise对象只有3种状态：

 1. 异步操作未完成（pending）
 2. 异步操作已完成（resolved/fulfilled）
 3. 异步操作失败（rejected）


<pre><code>
//new Promise
var promise = new Promise(function(resolve, reject) {
  // 异步操作的代码

  if (/* 异步操作成功 */){
    resolve(value)
  } else {
    reject(error)
  }
})

//Promise对象生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数。
promise.then(function (data){
    console.log(data) //打印上面resolve出的结果
})
promise.catch(function (data){
    console.log(data)  //打印上面reject出的结果
})
</code></pre>

总结：Promise提供了一种类似状态仓库的概念，优势是可以链式调用一些异步操作，例如promise.then().then()，避免了回调函数的多层“回调地狱”，但是又会陷入新的“then跑火车”，出现一大堆的then，虽然相对于之前的方案有所改善，但是在异步操作特别的多的情况下，还是会出现语义不清楚。

## Generator（生成器）
Generator是一个函数，也称为生成器函数。生成器函数的语法为function*，在其函数体内部可以使用yield关键字。
<pre><code>function* gen(x){
    console.log(1)
    var y = yield x + 2
    console.log(2)
    return 2
}
var g= gen(1)
g.next()  //{value:3, done:false}
g.next()  //{value: undefined, done: true}
</code></pre>
可以看出，直接调用Generator函数并不会立即执行内部代码，而是会返回一个遍历器g，然后调用这个遍历器的next方法会移动内部指针到出现yield的地方。即每次调用next(),都会执行到有yield的地方，继续next()继续执行，直到没有了yield或者return，执行才会结束并返回{value: undefined, done:true} <br/>
注意：next方法还可以接收参数，代表上一个yield表达式的值。
<pre><code>function* gen(x){
    console.log(1)
    var y = yield x + 2
    console.log(2)
    return y
}
g.next()  //{value:3, done:false}
g.next(2)  //{value: 2, done: true}
</code></pre>
下面是一个使用Generator请求接口的异步操作：
<pre><code>function* fetchApi(x){
     var url = "https://api.test.com/getUsers"
     var res = yield fetch(url)
     console.log(res)
}
</code></pre>
上述代码，模拟了一个异步请求操作，这段代码除了加了一个yield外，更像一个同步操作。现在我们再手段执行上面的方法：
<pre><code>var g = fetchApi()
var res = g.next()   //开始移动指针到yield处,此时res de value
应该会被赋值为一个拿到异步结果的Promise
res.value.then( (data) => {
    console.log(data.json())   // 打印出接口返回数据
})
</code></pre>
上述代码，实现了一个简单的异步任务，但是还有一个问题，就是整个流程需要手动去实现管理，那么如何自动化异步任务的流程管理呢？
举个栗子：
<pre><code>
function autoRun(fn){
    var g = fn()
    function next(err, data){
        var res = g.next(data)
        if(res.done)  returen      //如果执行到return，或者没有了yield
        res.value(next)     //迭代，直到res.done为true
    }
    next()
}
autoRun(fetchApi)
</code></pre>
有了这个执行器，不管内部有多少个异步操作，直接把Generator函数传入autoRun函数即可。注意：Generator函数里面的每一个异步操作，都必须是Thunk函数。<br/>
co模块是著名程序员 TJ Holowaychuk 于 2013 年 6 月发布的一个小工具，用于 Generator 函数的自动执行。<br/>
下面是一个 Generator 函数，用于依次读取两个文件。

    var gen = function* () {
      var f1 = yield readFile('/etc/fstab');
      var f2 = yield readFile('/etc/shells');
      console.log(f1.toString());
      console.log(f2.toString());
    }
    var co = require('co')
    co(gen).then(function(data){
        console.log(data)
    })

co模块可以让你不用编写Generator函数的执行器。Generator函数只要传入co函数，就会自动执行。co函数返回一个Promise对象，因此可以用then方法添加回调函数。 <br/>
自己实现一个简答的co模块，如下：
<pre><code>
function co(generator){
   var g = generator()
   function next(data){
        var res = g.next(data)
        if(res.done) return
        if(res.value instanceof Promise){
            res.value.then(function(val){
                next(val)
            }).catch(functin(err){
                next(err)
            })
        }else{
            next()
        }
   }
   next()
}
</code></pre>


## async/await
ES2017 标准引入了 async 函数，使得异步操作变得更加方便。async函数其实就是Generator函数的语法糖。<br/>
例如上面的genetator栗子：
<pre><code>
function* fetchApi(x){
     var url = "https://api.test.com/getUsers"
     var res = yield fetch(url)
     console.log(res)
}
function autoRun(fn){
    var g = fn()
    function next(err, data){
        var res = g.next(data)
        if(res.done)  returen      //如果执行到return，或者没有了yield
        res.value(next)     //迭代，直到res.done为true
    }
    next()
}
autoRun(fetchApi)
</code></pre>
如果写成async函数，就是下面这样。
<pre><code>
async fetchApi(){
     var url = "https://api.test.com/getUsers"
     var res = await fetch(url)
     console.log(res)
}
</code></pre>
async函数就是将 Generator函数的星号（*）替换成async，将yield替换成await。相比于Generator,async有如下几个特点:

 1. 内置执行器。类似于co模块，async函数在js生态中已经内置了一个执行器，不需要再去手动引入；
 2. 返回值是Promise；
 3. 更好的语义性。




## 总结
js在处理异步任务的时候，callback是最早的处理方式之一，弊端是容易引起“回调地狱”。Promise很好的弥补了回调方法的缺陷，可以采用链式调用的方式去处理异步，但是太多的“then”，在语义上无法很好的表达出所要表达的流程。async/await看上去是个不错的原则，以同步展现的方式去处理一些异步任务，便于阅读和理解，但是需要注意浏览器的兼容性（可以babel或者其他的一些方法去解决）。
