---
title: spa页面如何记录页面浏览位置【下】
date: 2019-01-04 17:49:05
tags:  spa页面如何记录页面浏览位置，浏览器的history记录栈
---



### **回顾**
>  上篇博客，我使用hashchange函数来监听每次hash改变的时候，记录当前的scrollTop值到sessionStorage中，然后再进入该页面时就看sessionStorage中是否有记录值，如果有就在页面html内容全部加载完成后取出记录值使页面加载到同样的位置，如果没有记录就记录当前的位置。一波简单的操作之后，页面在hash变化的时候，确实能按照我们预期的那样进行着，非第一次进入一个页面，会保存上次浏览该页面的位置继续浏览，但是！当我在切换多个hash后，按物理键返回或点击页面的返回键的时候，页面并没有按照我当前记录的那样保存位置，而是发生了错乱。

### **问题调研**
> 在一系列的断点调试的时候，终于找到了问题所在。假如有两个页面a(#a) 和b(#b)，先从a页面跳到b页面，然后在b页面返回到a页面的时候，注意了！浏览器的行为这时候有点诡异，返回的时候，首先a的scrollTop值赋给当前页面！首先a的scrollTop值赋给当前页面！首先a的scrollTop值赋给当前页面！然后再改变hash，然后再渲染dom加载内容，那么一切都变得合理了
由于首先会把a的滚轮值给b页面，那其实这时候b的scrollTop值其实已经是a的了，然后再hash改变的时候，执行我们的hash监听函数，把b的scrollTop值(其实是a的位置记录)错误的更新了，当我们一直点返回，就会一直记录错的位置，只要返回到已经错误记录过的那个页面时，就会出现我们前面说的那种错误现象了。
那么找到问题所在了，原来这是单页面的特性使然，并不是代码问题，那解决也就变的简单了，只要让页面返回或者前进的时候，不执行我们定义的hashchange函数，也就是只要是页面返回或者前进的时候，就不记录当前的scrollTop值，让他只执行他的页面变化，，就OK了。
避免再次踩坑，我还是不急着解决该问题，而是决定彻底把浏览器的记录行为弄清楚先！！！

### **理解浏览器的历史记录**
> 1.浏览器有一个类似栈的网页记录的数据结构，我把他叫做记录栈，在同一个窗口下，改变网页的时候，会把当前的网页记录推到记录栈中，刷新页面不会增加记录栈。浏览器有一个history对象，可以记录这种行为，但是只能通过length属性查看当前窗口消息历史记录的总数，不能查看当前栈里的情况，对于开发者来说，操作记录栈相当于操作一个黑箱子，所以管理记录栈显得尤为重要。
>

> 3.浏览器可以通过返回前进来改变当前记录栈中记录的位置，但是不会改变记录栈，可以知道，一定有一个类似于指针的东西，在表示当前网页在记录栈中的位置，例如：还是上面的那个流程，在我从b.html跳到c.html时，此时的指针是指向最高层的c.html的，没有前进只有后退，因为此时指针处于栈的最高层，当返回的时候，指针下移，指向b.html，此时页面为b.html，（[图片引用于流云诸葛的博客图片](http://www.cnblogs.com/lyzg/p/5941919.html)）
{% img  http://wickhamxu.cn/1546599916955.png 图片引用于流云诸葛的博客%}
>4.这里还有一个影响记录栈的情况(和pushstate类似)，还是从空-->a.html-->b.html-->c.html，此时的记录栈的大小为4，如果在c页面返回到b页面，再从b页面返回到a页面，此时的记录栈的大小还是为4，只是指针发生了变化，然后在a页面，我在url中输入c.html，此时会跳到c页面，此时的记录栈中的大小变为3了，空 a.html c.html，当在a页面输入地址跳到c页面时，此时会吧c页面推到当前指针的上方，然后将指针指到c页面的记录，如果再推入新记录之前当前指针的上方还有别的记录栈，推入新记录后也会被删除。（[图片引用于流云诸葛的博客图片](http://www.cnblogs.com/lyzg/p/5941919.html)）
{% img  http://wickhamxu.cn/1546599938362.png 这里写图片描述%}

### history API
historyapi有两个api和一个事件方法

1.history.pushState(stateObj,title,url)
>往当前记录栈中当前指针的后面push一个新的记录，然后将指针移到当前这个记录，如果在push新记录的时候，当前指针后面还有旧的记录，则后面的记录都会丢弃。stateObj参数是一个和当前记录条目绑在一起的一个对象，可以根据这个对象来标示记录栈中的记录条目，第二个参数title目前浏览器没有实现，第三个参数url是要改变新条目的地址，如果为空则表示当前浏览器地址。

2.history.replaceState(stateObj,title,url)
>和上述api使用方法完全一样，只是有几个不同点。差异1：repalceState只会改变当前记录条目，不会改变该记录条目的位置，也不会改变记录栈；差异2：不会改变指针的位置

3.window.onpopstate
>该事件是window下的一个事件，触发条件为当前记录栈发生变化的时候(前进，后退，hash改变)
>pushState 和 repalceState都不会触发该事件
>在popstate的事件对象里面，有一个state属性，会返回这个激活条目关联的stateObj对象的拷贝。一个历史记录条目只有当它是被pushState创建的，或者用replaceState改过的，才可能有关联的stateObj对象，所以当某些非这2种条件的历史记录条目被激活的时候，可能拿到的stateObj就是null，这样就可以根据pushState和replaceState，可以监听返回和前进了


### 解决问题
说了那么多，其实问题的解决也很简单了，就是监听浏览器的返回(前面出现的问题就是因为浏览器的返回也会触发我之前写的hashchange方法来记录下了错误的位置)，不让他去记录新的位置信息

```
window.addEventListener('hashchange',function(e){

//监听返回前进键
    window.addEventListener('popstate',function (e) {
        if(e.state && e.state.data == 'backTag'){  //如果是前进或者后退操作
            window.isRecordPages = false
        }else {
            window.isRecordPages = true
        }
    },false)
 //为当前导航页附加一个tag
   this.history.replaceState({data:'backTag'},'','')
   if(window.isRecordPages){
	setRecords()
	}
   function setRecords(){
	 var currentHash = e.oldURL.split('?#/')[1]//e.oldURL返回的是跳转前的页面，比如A－>B，那么e.oldURL的值就是A页面的URL，由于我这里的url规则是 main.html?#/xxx/a=xx,所以我做了一个字符串分割，具体因自己的要求所定，这样我就取到了跳转前页面的部分hash值(不带#/)
     if(currentHash.indexOf('/')!=-1){ //例如:#/aa/bb/cc=xxx  这种情况
         var sessionName = currentHash.substring(0,currentHash.lastIndexof('/'))  + '_pageRecord'
     }else{    //例如 #/aaa  这种情况
         var sessionName = currentHash.substring(0,currentHash.length)  + '_pageRecord'
     }
  //上述操作只是想标准化我存入到本地的数据的键的规范，例如 main.html#/a  存入的键为a   main.html#/a/b=xx存入的键为 a  main.html#/a/b/c=xxx  存入的键为a/b来加以区分
    var obj ={}
    if(!sessionStorage.pagesRecord){ //如果不存在 pagesRecord  存入当前的scrollTOp
        obj[sessionName] = document.body.scrollTop
        try{
        sessionStorage.setItem('pagesRecord',JSON.stringify(obj))
        }catch(err){
            console.log(err)
        }
     }else{
        try{
            obj = JSON.parse(sessionStorage.getItem('pagesRecord'))
            obj[sessionName] = document.body.scrollTop
            sessionStorage.setItem('pagesRecord',JSON.stringify(obj))
        }catch(err){
            console.log(err)
        }
    }
	}

})
```




