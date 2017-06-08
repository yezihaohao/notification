### notification
notification with socket.io
#### 1
npm install 
#### 2
node index.js
#### 3
open the browser and visit http://localhost:8080/

### 前言
> socket.io: 包含对websocket的封装，可实现服务端和客户端之前的通信。详情见[官网](http://socket.io) (虽然是英文文档，但还是通俗易懂)。
  Notification: Html5新特性，用于浏览器的桌面通知，只有部分浏览器支持。
  通过nodejs+Socket.io+Notification实现服务端往浏览器客户端发送自定义消息。
  若有问题可加群264591039与我讨论。
  转载请注明出处！
  
### 开发前提
本地安装nodejs以及npm
对nodejs以及express框架有一定了解。（本篇将和官方文档一样，采用express 4.10.2）

### 环境搭建
- 新建一个文件夹notification（以下操作都在该文件夹的根目录进行）
- npm初始化package.json文件 `npm init`
- 安装express(指定版本4.10.2，没有试过其他版本，感兴趣可以试下) `npm install --save express@4.10.2`
- 安装socket.io(本人安装的版本是1.7.3) `npm install --save socket.io`

### 编写代码
#### 构建通信环境
在根目录下新建一个index.html（注：index页面的样式来自socket.io的官方示例）
```
<!doctype html>
<html>
  <head>
    <title>Socket.IO Notification</title>
    <style>
      * { margin: 0; padding: 0; box-sizing: border-box; }
      body { font: 13px Helvetica, Arial; }
      form { background: #000; padding: 3px; position: fixed; bottom: 0; width: 100%; }
      form input { border: 0; padding: 10px; width: 90%; margin-right: .5%; }
      form button { width: 9%; background: rgb(130, 224, 255); border: none; padding: 10px; }
      #messages { list-style-type: none; margin: 0; padding: 0; }
      #messages li { padding: 5px 10px; }
      #messages li:nth-child(odd) { background: #eee; }
    </style>
  </head>
  <body>
    <ul id="messages"></ul>
    <form action="">
      <input id="m" autocomplete="off" /><button>Send</button>
    </form>
  </body>
</html>
```
新建一个index.js文件，并在js文件中构建相应的对象和变量，创建监听端口为8080 的服务器，以及添加映射到index.html的路由。

```bash
let express = require('express'),
	app = express(),
	http = require('http').Server(app),
	io = require('socket.io')(http);
app.use(express.static(__dirname + '/public'));

app.get('/', function(req, res){
	res.sendfile('index.html');
});

http.listen(8080, function(){
	console.log('listening on port 8080');
});
```
运行 `node index.js` 用浏览器打开http://localhost:8080 成功的话即可看到index.html页面的内容。
在index.js的监听端口代码上方添加socket.io的监听事件，监听用户连接的connection。
```bash
io.on('connection', function(socket){
	console.log('a user connected');
});
```
创建监听Event事件:notification，并用emit向客户端推送消息
```bash
io.on('connection', function(socket){
	console.log('a user connected');
	socket.on('notification', function(msg){
		console.log(msg);
	 	io.emit('notification', msg);
	});
});
```
在index.html页面中的</body>上方引入socket.io文件，并用emit向服务器提交数据以及监听事件notification，接收服务器推送的消息
注意，引入socket.io的方式在运行`node index.js`之后才有效果，直接打开index.html是找不到这个文件的
```bash
    <script type="text/javascript" src="/socket.io/socket.io.js"></script>
    <script src="http://code.jquery.com/jquery-1.11.1.js"></script>
    <script>
		let socket = io();
		$('form').submit(function(){
			socket.emit('notification', $('#m').val());
			$('#m').val('');
			return false;
		});
		socket.on('notification', function(msg){
			console.log(msg);
		});
    </script>
```
浏览器打开http://localhost:8080 后，在input框中输入，点击发送，在nodejs运行的控制台可以看到如下信息：
```bash
a user connected //以下数据是输入框输入的数据
hello   
test
测试
```
#### 实现自定义消息推送
完整代码：
```bash
    <script>
		let socket = io();
		$('form').submit(function(){
			socket.emit('notification', $('#m').val());
			$('#m').val('');
			return false;
		});
		socket.on('notification', function(msg){
			notice(msg);    //若允许通知，待输入消息后监听变化就会调用通知方法
		});

	    Notification.requestPermission(function(permission) {});    //询问浏览器是否允许通知
      
		function notice(msg) {  
			let _notification = new Notification(`消息通知`,{
				body:`${msg}`,
				icon:'http://localhost:8080/23539868.jpg'
			});

			setTimeout(function(){
				_notification.close(); //设置5秒后自动关闭通知框
			},5000);
		  
		}
    </script>
```
### 运行截图

完整示例代码见[GitHub](https://github.com/yezihaohao/notification)

![截图](https://raw.githubusercontent.com/yezihaohao/yezihaohao.github.io/master/imgs/TEST.gif)
