---
title: Nodejs学习
tags: [前端,Node.js]
---
web服务: express, koa, hapi
模板引擎: handlebars, ejs, jade
前端打包: webpak, fis,
任务管理: gulp
单元测试: karma, mocha, jasmine
包管理器: npm, cnpm, yarn
守护进程: pm2

<!--more-->

##### npm

###### 初始化

```powershell
$ npm init 
```

###### 依赖安装

```powershell

# 安装但不写入package.json； 
$ npm install xxx
 
# 安装并写入package.json的"dependencies"中；
$ npm install xxx –S 
 
# 安装并写入package.json的"evDependencies"中;
$ npm install xxx –D
 
```

S（等同于--save）表示项目打包时会将该依赖包一并打包；-D（等同于--save-dev）表示该依赖包仅在开发环境下使用，正式打包不会加到项目中。

##### express模块

###### 项目初始化

```powershell
$ npx express-generator --view=ejs 项目名字
```

###### 安装依赖

```powershell
$ nmp i
```

###### 启动项目

```powershell
$ npm start
```

###### 目录结构

- bin: 启动目录 里面包含了一个启动文件 www 默认监听端口是 3000 (不用)

- node_modules: 所有安装的依赖模块 都在这个文件夹里面

- public: 所有的前端静态资源 html css image js

- routes: 放的是 路由 文件 (默认有两个)路由主要定义 url 和 资源 的映射关系 ( 一一对应关系 )主要用来接收前端发送的请求 响应数据给前端

- views: 主要放置 ejs 后端模板文件

- app.js: 入口文件(主文件) 总路由 (其他的路由 要由它来分配)

- package.json: 包描述文件 最重要的是 依赖的模板列表 dependencies

###### app.js文件

```javascript
var indexRouter = require('./routes/index');//引入路由文件
var app = express(); //创建服务器

// 设置服务器模板渲染引擎
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

app.use(logger('dev')); //设置日为开发者模式

app.use(express.json()); //让express处理 json 数据

app.use(express.urlencoded({ extended: false })); //用于接收 POSt 请求数据

app.use(cookieParser()); //处理 cookie 数据

//静态资源设置；__dirnam代表当前文件的路径
app.use(express.static(path.join(__dirname, 'public'))); 
//路由(API)
app.use('/', indexRouter);
app.use('/users', usersRouter);
```

#####  连接mysql

###### 下载

```powershell
$ npm i mysql --save
```

###### 引用

```javascript
var mysql = require('mysql')
var myConnection = mysql.createConnection({
    host:'localhost',
    user:'root',
    password:'',
    database:'guoxinan',
    port:3306
});
function query(sql,option,callback){
    //打开连接
    myConnection.connect();
    //sql操作
    myConnection.query(sql,option,function(err,data){
        if(err){
            console.log("操作数据库失败",err);
            return;
        }
        callback(data);
    });
    myConnection.end();
}

```

###### 问题

```powershell
MYSQL：ER_NOT_SUPPORTED_AUTH_MODE:Client does not support authentication protocol
```

原因:8.0mysql引入了caching_sha2_password模块作为默认身份验证插件，nodejs还没有跟进

解决办法:进入mysql修改密码

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '自己的密码';
```

