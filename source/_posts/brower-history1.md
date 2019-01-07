---
title: spa页面如何记录页面浏览位置【上】
date: 2019-01-04 17:49:02
tags: spa页面如何记录页面浏览位置，浏览器的history记录栈
---

#### **理论支持**
   >该需求是：我们的项目是微信上的spa应用，要求用户在A页面查看内容后，切换到B页面，C页面...在每个页面中查看的位置要保留，在下次切换为该页面时，是从上次浏览结束的位置开始查看，而不是从头开始，从Technology上说这里浏览位置的对应为滚动条距页面顶端的距离(document.body.scrollTop)。

   >起先我有两种可行性方案：
  >>1.假如有3个页面要实现我们上述的浏览位置记录，我们分别称为a页面，b页面和c页面，我们可以在将三个页面用3个div全部放在同一个地方，当我们的hash改变时，把对应的display:block，其余的两个display:none，这样其实三个页面的浏览位置相当于同一个页面的浏览位置，而我们所做的只是单纯的显示和隐藏，
  >>> 代码如下：
  `		<div class='pageA' style='dispaly:block'>...</div>
		<div class='pageB' style='dispaly:none'>...</div>
		<div class='pageC' style='dispaly:none'>...</div>
`

>>这样就能实现想要的结果，将想要的保存记录的页面写在一起，通过显示和隐藏来完成想要的结果，虽然这种方案虽然实现起来比较简单，但是弊端也显而易见，这种方案会改变以前项目的原有的一个结果，第二，这种方案不适合多个页面记录位置。如果是需要两三个页面的记录保存，这种方案未尝不可选择。
>
>>2.同样我们假设有abc3个页面，现在要实现不同页面间的浏览位置保存，无疑要使用本地存储，我使用的是sessionStorage(没人会愿意退出应用后再进来还是使用上次的位置浏览)，而存储的值就是每个页面的scrollTop值，用来区分每个页面的scrollTop值则用的是自己的hash。



由于具体的项目要求和局限，我们下面使用的是第二种方案

#### **具体实现**
 1.  监听hashchange，在页面hash变化的时候，把当前页面的scrollTop值存到本地


```
window.addEventListener('hashchange',function(e){
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

})
```
一顿操作，现在应用中只要hash一发生变化，那么每次就会存入当前路由的页面scrollTop值，如果存在就覆盖，如果不存在就添加，接下来就是具体在每个页面中dom加载全部完成后，取出对应的pagesRecord值，然后执行documen.body.scrollTop = scroll值  就ok了

```
function setPagesRecord(hash){  //传入当前hash

	//和之前存入pagesRecord的操作类似，为逆操作
	if(hash.lastIndexOf('/')!=1){//
		var key = hash.substring(2,hash.lastIndexOf('/')+'_pageRecord')
	}else{
		var key = hash.substring(2,hash.length)+'_pageRecord'
	}
	if(sessionStorage.getItem('pagesRecord')){ //如果存在页面记录
		var pagesRecord = JSON.parse(sessionStorage.pagesRecord) //取出数据
		if(!!pagesRecor[key]){  //如果存在的话
			document.body.scrollTop = pagesRecord[key] //改变页面scrollTop
		}
	}
}
```

#### **实现结果**
>nice，我在切换页面，在hash change的时候，每次都能记录我上次浏览的位置，退出会话然后重现进入应用，会重现记录，一切貌似到目前为止都是朝着我们的预期进行着，而且也都基本实现了预期，就在我满心欢喜的准备提交代码over的时候，在我们的页面点了一下返回键，卧槽，说好的位置记录呢，说好的预期呢。

   > 在点手机上的物理返回键或者页面上点击页面前进或者后退的时候，之前记录的位置全部发生错乱了，该bug出现的原因和解决方案，就留在下篇博客了

