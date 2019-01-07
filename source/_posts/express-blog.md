---
title: express-blog
date: 2017-01-09 17:12:42
tags:
---
### 项目简介
本项目是基于nodeJs开发的一个个人博客系统，由于此前对于node的不熟，总共历时3周时间(平时有点懒...)，一边学习其语法，一边开发博客系统，由于其博客是仿照CSDN博客开发的，所以如果有一种CSDN的既视感，请不要吐槽...


- ** 该博客支持多人注册，首页按照博客发表时间，点击次数排行来更新首页内容 **
- ** 用户可注册账户并登陆，可以发表自己的博客文章，选择或者添加自己的分类 **
- ** 个人中心(由于当时开发的着重点是node后台，所以个人中心部分前端页面有点low)，用户可以修改用户名和密码，也可以上传个人头像(如果不上传，系统分配默认头像) **
- ** 后台部分，用户可自由上传修改删除博客文章、分类 **
- ** 系统包括首页、登录注册页、博客列表页、博客详细页、博客编辑修改页、个人中心页几个模块 **

-------------------

### 项目结构

{% img  http://wickhamxu.cn/1546854109373.png 这里写图片描述%}

### 开发所用技术

整个项目结构是基于express搭建的，前台css使用bootstrap框架，数据库采用mongoDB，操作数据库使用的是mongoose模块，用户上传博客的编辑器是使用的是百度的ueditor插件

### 开发流程
1.开发环境的搭建
 由于博客是基于node的，所以首先必须要搭建node环境，下载node和npm   ，然后使用npm下载express && mongoose ,引用一些常用插件放在public/plugins目录下，我使用的是jQuery和bootstrap。
 2.后台服务器的部署
 当安装完express后，会有一个项目目录，express已经帮我们搭好了服务器，这时候在根目录下使用命令node bin/www即可开启服务器，在浏览器输入localhost:3000应该就会有一行欢迎界面(默认端口是3000)，那么基本的服务器已经可以用了，如果出现错误或者无法访问的话，检查其端口是否被占用(我当时是被一个系统应用给占用了端口)。
 3.服务器搭好了，我接下来是写了一个首页 ，首页上加了一个动画，看上去让自己的博客叼叼的，然后实际并没有起到什么作用。

 {% img  http://wickhamxu.cn/1546854231553.png 这里写图片描述%}
 4.接下来 根据一些简单的需求开始设计数据库，我的数据库只有三个表(因为当时只想在学习node的时候实现一些小功能就好了)，分别是users(用户基本信息表)、contents(用户博客内容表)、cats表(用户博客分类表)
 users表
 {% img  http://wickhamxu.cn/1546854303757.png 这里写图片描述%}
 contents表
 {% img  http://wickhamxu.cn/1546854364088.png 这里写图片描述%}
 cats表
 {% img  http://wickhamxu.cn/1546854414511.png 这里写图片描述%}
5.数据库表设计好了，然后使用mongoose模板，连接数据库，

```
var mongoose=require("mongoose");
var db='mongodb://localhost:27017/blog';
mongoose.connect(db);

mongoose.connection.on('connected',function () {
    console.log("connected..");
});
mongoose.connection.on('error',function (err) {
    console.log("connected err"+err);
});
mongoose.connection.on('disconnected',function () {
    console.log("disconnected");
});
process.on('SIGINT',function () {
    mongoose.connection.close(function () {
        console.log("close");
        process.exit(0);
    });
});

exports.mongoose=mongoose;
```
该代码在models目录下，然后该将模板导出(连接数据库)
操作数据库模板，使用的是mongoose的Schema，通过实例化该方法，构建model，以user为例，如果使用user表的增删改查，可以导入该声明好的模板，添加find,update等方法

```
//user.js
var mongoose=require("mongoose");
var Schema = mongoose.Schema;
//声明Schema
var _User = new Schema({
    name: String,
    password:String,
    pic:String
});
//构建model
var pmodel=mongoose.model('user', _User);

exports.User = pmodel;
```

6.接下来就是路由设置，该博客模板有如下：
{% img  http://wickhamxu.cn/1546854473232.png 这里写图片描述%}
express在配置路由时，使用app.method("url",function(req,res){})
每个模板页面，在入口文件app.js里面配置路由，控制器部分，我写在路由里面，只要访问该路由，即使用该控制器里的办法。

```
//配置页面路由
app.use('/', index);
app.use('/', git);
// app.use('/', blog);
app.get("/blog",blog.blog1);
app.get("/blog/user/login",blog.login);
app.get("/loginError",blog.loginError);
app.use('/', contact);
app.all('/blog/detail',multipartMiddleware,blog.detail);
app.get('/blog/user/reg',blog.reg);
app.get('/blog/user',user.user);
app.post('/blog/user',user.userUpdate);
app.get('/blogEdits',blogEdit.blog_edit);
app.post('/blogEdits',blogEdit.blog_updateBlog)
app.post('/blog/fwNum',blog.fwNum)
app.get('/blog/detail/Update',blogEdit.updateC)
app.all('/blog/detail/Article',blog.readAll)
```
所有路由通过模板导出，供入口文件使用

7.相同路由下的不用问题处理解决方法
 这种情况下 我使用的是通过发送"其他参数"，我这里是role参数，通过role参数的不同来响应不同的请求方法


```
async.parallel({   //parallel函数是并行执行多个函数，每个函数都是立即执行，不需要等待其它函数先执行
          getContentAll:function (callback) {
              content.Content.find(function (err, docs) {
                  if (err) {
                      res.render('error');
                  } else {
                      // console.log(docs)
                      callback(null, docs)
                  }
              })
          },
          getContent: function (callback) {
              content.Content.find({name:req.session.username},function (err, docs) {
                  if (err) {
                      res.render('error');
                  } else {
                      callback(null, docs)
                  }
              });
          },
          getContentByClick:function (callback) {
              content.Content.find({}).sort({'click':-1}).limit(5).exec(function (err, docs) {
                  if (err) {
                      res.render('error');
                  } else {
                      callback(null, docs)
                  }
              })
          }
      }, function(err, results){
          res.render('detail', {
              title:"博客详细页",
              username: query1.name,
              userSession:req.session.username,
              imgUrl:"/images/touxiang/Koala.jpg",
              contentsAll:results['getContentAll'],
              contents:results['getContent'],
              contentsByClick:results['getContentByClick'],
              date: new Date()
          });
      });

  }else if(req.body.sRole==2){    //处理ajax（用户名是否注册过）
      var query2 = {name: req.body.name};
      user.User.find(query2,function (err, docs) {
          if(err){
              res.render('error');
          }else {
                  if(docs.length==1){
                      res.json({msg:1});
                  }else{
                      res.json({msg:0});
                  }
          }
      });
  }
  else if(req.body.sRole==3){   //处理登录
      var imgur2="";
      var query3={name:req.body.user,password:req.body.password};
      async.parallel({   //parallel函数是并行执行多个函数，每个函数都是立即执行，不需要等待其它函数先执行
          getContentAll:function (callback) {
              content.Content.find(function (err, docs) {
                  if (err) {
                      res.render('error');
                  } else {
                      callback(null, docs)
                  }
              })
          },
          getContent: function (callback) {
              content.Content.find({name:query3.name},function (err, docs) {
                  if (err) {
                      res.render('error');
                  } else {
                      callback(null, docs)
                  }
              });
          },
          getImgUrl:function (callback) {
              user.User.find(query3,function (err, docs) {
                  if(err){
                      res.render('error');
                  }else {
                      if(docs.length==0){
                          res.redirect("/loginError");
                      }else {
                          if(docs[0]['pic']==""){
                              imgur2="/images/touxiang/Koala.jpg"
                          }else{
                              imgur2=urlHandle(docs[0]['pic'])
                          }
                          req.session.username=query3.name;
                          req.session.password=query3.password;
                          req.session.imgURl=imgur2;
                       callback(null,imgur2);
                      }
                  }
              });
          },
          getContentByClick:function (callback) {
              content.Content.find({}).sort({'click':-1}).limit(5).exec(function (err, docs) {
                  if (err) {
                      res.render('error');
                  } else {
                      callback(null, docs)
                  }
              })
          }
      }, function(err, results){
           console.log(results['getContentByClick'])
          res.render('detail',{
              title:"博客详细页",
              username:query3.name,
              userSession:req.session.username,
              imgUrl:results['getImgUrl'],
              contentsAll:results['getContentAll'],
              contents:results['getContent'],
              contentsByClick:results['getContentByClick'],
              date:new Date()
          });
      });


  }
  else if(req.body.sRole==4) {
      var imgurl3 = "";
      //处理头像上传
      var query4 = {name: req.session.username};
      var file = req.files.pic;
      var path = file.path;
      //存储路径
      user.User.update(query4, {$set: {pic: path}}, function (err, docs) {

      });
      if (path == "") {
          imgurl3 = "/images/touxiang/Koala.jpg"
      } else {
          imgurl3 = urlHandle(path)
      }
      req.session.imgURl = imgurl3;


      async.parallel({   //parallel函数是并行执行多个函数，每个函数都是立即执行，不需要等待其它函数先执行
          getContentAll:function (callback) {
              content.Content.find(function (err, docs) {
                  if (err) {
                      res.render('error');
                  } else {
                      callback(null, docs)
                  }
              })
          },
          getContent: function (callback) {
              content.Content.find({name: query4.name}, function (err, docs) {
                  if (err) {
                      res.render('error');
                  } else {
                      callback(null, docs)
                  }
              });
          },
          getContentByClick:function (callback) {
              content.Content.find({}).sort({'click':-1}).limit(5).exec(function (err, docs) {
                  if (err) {
                      res.render('error');
                  } else {
                      callback(null, docs)
                  }
              })
          }
      }, function (err, results) {
          res.render('detail', {
              title: "博客详细页",
              username: query4.name,
              userSession: req.session.username,
              contentsAll:results['getContentAll'],
              contents: results['getContent'],
              contentsByClick:results['getContentByClick'],
              imgUrl: imgurl3,
              date: new Date()
          });
      });
```


8.在实现基本的查找功能时，由于node独特的回调异步处理机制，出现情况，在查找过程中，有时候还没等到结果查找出来就处理了该结果，期初解决的方法是嵌套，将处理结果的操作放在查询的回调函数里，这样就能等到数据库结果查询出来后再处理操作，但是如果查询存在嵌套的话 ，那们嵌套的层数将会越来越多，很容易产生混乱，也不美观。于是我采用了async插件来管理node的异步回调(可以使用npm下载)，async.parallel,async.waterfull等。
### 项目总结

再本次开发过程中，令我感受最深的就是node的回调机制，几乎所有的操作，所有的方法函数，都存在着回调。这种回调可以处理高并发的处理，但是也会给我们开发带来一些不友好的地方，但是可以根据async插件或者es6新特性来避免这些trouble。博客还有很多功能有待完善，包括文章转载功能，文章评论功能，在以后的日子里，我一定会更加完善这些地方。但是自己对于node的兴趣，却高涨不下，希望自己以后能够更多的接触node，学习node。

项目地址:[https://github.com/xuweikang/node-express-mongo-blog](https://github.com/xuweikang/node-express-mongo-blog)