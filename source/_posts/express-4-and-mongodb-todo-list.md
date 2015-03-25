title: "用Express 4及MongoDB做todo List"
date: 2015-03-24 22:43:37
tags:
---
本篇文章主要參考自[DreamersLab - 用Express和MongoDB寫一個todo list](http://dreamerslab.com/blog/tw/write-a-todo-list-with-express-and-mongodb/)。原文的教學內容是使用Express 3.x版，在這邊做簡單Express 4.12.1的todo list實作。

### 功能介紹
  + 用cookie來判斷使用者
  + 新增、讀取、更新、刪除待辦事項(C.R.U.D)

### 安裝
##### 開發環境
  確定已安裝`node.js`, `Express`和`MongoDB`
  + 安裝Express 4.12.1
    
        npm install express@4.12.1
    
  + 我們使用[**Mongoose**](http://mongoosejs.com/)來當作我們的ORM

### 步驟
##### 用Express command line tool來建立基本框架
Express預設的template engine為[jade](http://jade-lang.com/)，在這範例我們採用[ejs](http://www.embeddedjs.com/)
    
        $ express -e todo
        create : todo
        create : todo/package.json
        create : todo/app.js
        create : todo/public
        create : todo/public/javascripts
        create : todo/public/images
        create : todo/public/stylesheets
        create : todo/public/stylesheets/style.css
        create : todo/routes
        create : todo/routes/index.js
        create : todo/routes/users.js
        create : todo/views
        create : todo/views/index.ejs
        create : todo/views/error.ejs
        create : todo/bin
        create : todo/bin/www

        install dependencies:
         $ cd todo && npm install

        run the app:
         $ DEBUG=todo:* ./bin/www

##### 將`ejs-locals`以及`mongoose`加入dependencies
> package.json
    
        {
          "name": "todo",
          "version": "0.0.0",
          "private": true,
          "scripts": {
            "start": "node ./bin/www"
          },
          "dependencies": {
            "body-parser": "~1.12.0",
            "cookie-parser": "~1.3.4",
            "debug": "~2.1.1",
            "ejs": "~2.3.1",
            "express": "~4.12.2",
            "morgan": "~1.5.1",
            "serve-favicon": "~2.2.0",
            "ejs-locals"      : "1.0.2",
            "mongoose"        : "3.8.17",
            "uid-safe"        : "1.1.0"
          }
        }

##### 安裝dependencies
    
        $ cd todo && npm install

##### Hello Wolrd!
開啟Express server然後打開瀏覽器瀏覽**`127.0.0.1:3000`**就會看到歡迎頁面
    
        $ DEBUG=todo ./bin/www
    
##### 檔案結構
此時專案的檔案結構大致為下：

        todo
        |-- node_modules
        |   |-- ejs
        |   |-- ejs-locals
        |   |-- express
        |   `-- mongoose
        |
        |-- public
        |   |-- images
        |   |-- javascripts
        |   `-- stylesheets
        |       |-- style.css
        |
        |-- routes
        |   `-- index.js
        |
        |-- views
        |   |-- index.ejs
        |   `-- layout.ejs
        |
        |-- .gitignore
        |
        |-- app.js
        |
        `-- package.json
    
  + node_modules
    + 包含所有project相關套件
  + public
    + 包含所有靜態檔案
  + routes
    + 所有動作及商業邏輯
  + views
    + 包含action views, partials及layout
  + app.js
    + 包含設定、middlewares以及routes的分配
  + package.json
    + 相關套件的設定檔


##### MongoDB以及Mongoose設定
在`Ubuntu`上MongoDB開機便會自動開啟，若是`Mac`上就輸入下面指令開啟
    
        $ mongod

並且在專案根目錄之下新增一個檔案叫做`db.js`來設定MongoDB和定義Schema
> db.js
    
        var mongoose = require('mongoose');
        var Schema = mongoose.Schema;

        var Todo = new Schema({
          user_id     : String,
          content     : String,
          updated_at  : Date
        });

        mongoose.model('Todo', Todo);
        mongoose.connect('mongodb://localhost/express-todo');
            
並且在app.js裡require剛剛新增的db.js
> app.js
    
        require('./db');

並且移除掉Express預設產生，我們用不到的user route，並且加上layout support
> `var engine = require('/ejs-locals');`
> `app.engine('ejs', engine);`
    
        var express = require('express');
        var path = require('path');
        var favicon = require('serve-favicon');
        var logger = require('morgan');
        var cookieParser = require('cookie-parser');
        var bodyParser = require('body-parser');

        var routes = require('./routes/index');

        var app = express();
        var engine = require('/ejs-locals');

        // view engine setup
        app.set('views', path.join(__dirname, 'views'));
        app.set('view engine', 'ejs');
        app.engine('ejs', engine);

        // uncomment after placing your favicon in /public
        //app.use(favicon(__dirname + '/public/favicon.ico'));
        app.use(logger('dev'));
        app.use(bodyParser.json());
        app.use(bodyParser.urlencoded({ extended: false }));
        app.use(cookieParser());
        app.use(express.static(path.join(__dirname, 'public')));

        app.use('/', routes);

        // catch 404 and forward to error handler
        app.use(function(req, res, next) {
          var err = new Error('Not Found');
          err.status = 404;
          next(err);
        });

        // error handlers

        // development error handler
        // will print stacktrace
        if (app.get('env') === 'development') {
          app.use(function(err, req, res, next) {
            res.status(err.status || 500);
            res.render('error', {
              message: err.message,
              error: err
            });
          });
        }

        // production error handler
        // no stacktraces leaked to user
        app.use(function(err, req, res, next) {
          res.status(err.status || 500);
          res.render('error', {
            message: err.message,
            error: {}
          });
        });


        module.exports = app;


##### 修改project title
> routes/index.js     

原本的預設產生的程式碼如下：
    
        var express = require('express');
        var router = express.Router();

        /* GET home page. */
        router.get('/', function(req, res, next) {
          res.render('index', { title: 'Express' });
        });

        module.exports = router;
    
在這個範例之中我們一律將router的分配寫在`app.js`當中，因此將程式碼修改為：
    
        var express = require('express');
        
        exports.index = function(req, res) {
          res.render('index', {title : 'Express Todo Example'});
        };
    
##### 修改index view
在這裡我們需要一個textbox來新增待辦事項，並且用`POST`來傳送資料，另外別忘了設定layout
> views/index.ejs
    
        <% layout( 'layout' ) -%>

        <h1><%= title %></h1>
        <form action="/create" method="POST" accept-charset="utf-8">
          <input type="text" name="content" />
        </form>

##### 新增待辦事項及存檔
> routes/index.js     

首先先`require mongoose`和`Todo model`
    
        var mongoose = require('mongoose');
        var Todo     = mongoose.model('Todo');
        
        //新增成功後將頁面導回首頁.
        exports.create = function(req, res) {
          new Todo({
            content    : req.body.content,
            updated_at : Date.now()
          }).save(function(err, todo, count) {
            res.redirect( '/' );
          });
        };
    
接著在app.js當中新增對應的route
> app.js
    
        // 新增下列語法到 routes
        app.post('/create', routes.create);

##### 顯示待辦事項
> routes/index.js
    
        // 查詢資料庫來取得所有待辦是事項.
        exports.index = function(req, res) {
          Todo.find( function(err, todos, count) {
            res.render('index', {
                title : 'Express Todo Example',
                todos : todos
            });
          });
        };

> views/index.ejs
    
        // 跑迴圈顯示待辦事項
        <% todos.forEach(function(todo) { %>
          <p><%= todo.content %></p>
        <% }); %>
    
##### 刪除待辦事項
在每一個待辦事項旁邊增加刪除的連結
> routes/index.js
    
        // 根據待辦事項的id來做移除
        exports.destroy = function(req, res) {
          Todo.findById(req.params.id, function(err, todo) {
            todo.remove( function(err, todo) {
              res.redirect( '/' );
            });
          });
        };
    
> views/index.ejs
    
        // 在迴圈裡加一個删除連結
        <% todos.forEach(function(todo) { %>
          <p>
            <span>
              <%= todo.content %>
            </span>
            <span>
              <a href="/destroy/<%= todo._id %>" title="Delete this todo item">Delete</a>
            </span>
          </p>
        <% }); %>
    
將destroy的動作新增到對應的route
> app.js
    
        // 新增下列語法到 routes
        app.get('/destroy/:id', routes.destroy);
    
##### 編輯待辦事項
當滑鼠點擊事項時，將它轉為一個text input達到編輯效果
> routes/index.js
    
        exports.edit = function(req, res) {
          Todo.find(function(err, todos) {
            res.render('edit', {
                title   : 'Express Todo Example',
                todos   : todos,
                current : req.params.id
            });
          });
        };

新增Edit view，基本上和index view差不多，唯一不同是在被選取的待辦事項變成text input
> view/edit.js
    
        <% layout( 'layout' ) -%>

        <h1><%= title %></h1>
        <form action="/create" method="post" accept-charset="utf-8">
          <input type="text" name="content" />
        </form>

        <% todos.forEach(function(todo) { %>
          <p>
            <span>
              <% if ( todo._id == current ) { %>
              <form action="/update/<%= todo._id %>" method="POST" accept-charset="utf-8">
                <input type="text" name="content" value="<%= todo.content %>" />
              </form>
              <% } else { %>
                <a href="/edit/<%= todo._id %>" title="Update this todo item"><%= todo.content %></a>
              <% } %>
            </code>
            <span>
              <a href="/destroy/<%= todo._id %>" title="Delete this todo item">Delete</a>
            </code>
          </p>
        <% }); %>

待辦事項新增可以連到edit的link
> views/index.ejs
    
        <% layout( 'layout' ) -%>

        <h1><%= title %></h1>
        <form action="/create" method="post" accept-charset="utf-8">
          <input type="text" name="content" />
        </form>

        <% todos.forEach(function(todo) { %>
          <p>
            <span>
              <a href="/edit/<%= todo._id %>" title="Update this todo item"><%= todo.content %></a>
            </code>
            <span>
              <a href="/destroy/<%= todo._id %>" title="Delete this todo item">Delete</a>
            </code>
          </p>
        <% }); %>
    
將連結到編輯的route新增到app.js中
> app.js
    
        app.get('/edit/:id', routes.edit);

##### 更新待辦事項
新增update動作來更新待辦事項
> routes/index.js
    
        exports.update = function(req, res) {
          Todo.findById(req.params.id, function(err, todo) {
            todo.content    = req.body.content;
            todo.updated_at = Date.now();
            todo.save(function(err, todo, count) {
              res.redirect( '/' );
            });
          });
        };
    
將更新的動作新增到routes
> app.js
    
        app.post('/update/:id', routes.update);

##### 排序
現在待辦事項的順序為：**較早創建/更新**的在上面，我們要將它相反
> routes/index.js

        exports.index = function(req, res) {
          Todo.
            find().
            sort('-updated_at').
            exec(function(err, todos) {
              res.render('index', {
                  title : 'Express Todo Example',
                  todos : todos
              });
            });
        };

        exports.edit = function(req, res) {
          Todo.
            find().
            sort('-updated_at' ).
            exec(function(err, todos) {
              res.render('edit', {
                  title   : 'Express Todo Example',
                  todos   : todos,
                  current : req.params.id
              });
            });
        };

##### 多重使用者
目前為止，所有使用者看到的都是同一組待辦事項，資料有可能會被外人所修改，因此，我們可利用cookie來記錄使用者資訊，讓每個人都有自己的todo list。
Express已經有內建的cookie，我們先在app.js當中新增一個middleware就好。
> app.js
    
        // 將抓取使用者資訊的middleware加入app.js
        app.use( routes.current_user );

接著在routes/index.js增加current_user的運作邏輯
> routes/index.js
    
        // 我們採用uid-safe package來替我們產生uid，別忘了要npm install uid-safe哦
        var uid = require('uid-safe');

        exports.current_user = function(req, res, next) {
          var user_id = req.cookies ?
              req.cookies.user_id : undefined;
          if ( ! user_id ) {
            uid(32).then(function(uid) {
              res.cookie('user_id', uid);
            });
          }
          next();
        };

##### Error handling
要處理錯誤我們需要新增`next`參數到每個action當中，一旦發生錯誤便將他傳給下一個middleware去做處理
> routes/index.js
    
        ... function ( req, res, next ){
          // ...
        };

        ...( function( err, todo, count ){
          if( err ) return next( err );

          // ...
        });

##### 最後一步，執行
    
        $ DEBUG=todo ./bin/www

