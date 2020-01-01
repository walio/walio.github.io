---
title: websocket踩坑实践
date: 2017-06-26
updated: 2017-12-14
tags:
  - python
keywords:
  - websocket
  - python
  - socket.io
  - flask-socket.io
  - tornado.websocket
categories:
  - fe
---
*个人感觉tornado的websocket要比flask-socketio要好得多，毕竟tornado是原生支持，flask-socketio要依靠gevent或者eventlet，这两个库当中貌似有不少的坑*
个人感觉python的socketio服务器一般有两种比较典型的选择，flask-socketio+socket.io-client和tornado.websocket+原生websocket，这两者之前我最近做了一个比较
<!-- more -->
## 接受消息
flask-socketio收到前端的消息后会启动定义的event handler，然而在这个event handler中发送的消息要等这个event handler全部执行完毕以后才会发送，而且执行过程中无法响应心跳包。这就导致，当单个event handler执行耗时操作时，会一直没有任何消息发送到前端，如果时间太长前端会判断为超时，然后发起重连请求，真的很糟心。以下是示例，后台收到testmessage的事件后，每隔10秒发送一个消息到前台，然而事实情况会是，差不多后台发送到第四第五个数字后前端就判断为超时了，而且一个包都没有收到。
后台代码
```python
from flask import Flask
from flask_socketio import SocketIO, emit
import time

app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret!'
socketio = SocketIO(app)


@socketio.on("testmessage")
def init(message):
    for x in range(10):
        emit("printn", x)
        print(x)
        time.sleep(10)

if __name__ == "__main__":
    socketio.run(app, debug=True)
```
前端
```html
<script src="../assets/socket.io.min.js"></script>
<script>
    var socket = io.connect("http://127.0.0.1:5000/");
    socket.on("connect", function (data) {
       console.log("connected");
       socket.emit("testmessage", "test")
    });
    socket.on("printn", function (data) {
        console.log(data)
    })
</script>
```
像以上的代码，前端会一个信息都收不到，因为后台处理耗时太久，前端认为断开了。并且浏览器控制台会出现如下错误，貌似是socketio延时后回退到使用ajax，然后禁止跨域了？
```error
test.html:1 XMLHttpRequest cannot load http://127.0.0.1:5000/socket.io/?EIO=3&transport=polling&t=1498485030898-7&sid=9b604f29e92d4b1dad74adc4409ecf45. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'null' is therefore not allowed access. The response had HTTP status code 500.
```
由于我的业务是遍历一个过滤一个列表，比较耗时，一开始对整个列表的过滤写在一个socket.on里面，还以为是有延迟，后面的消息和前面的消息冲突了，没有收到，实际是后台一直无法响应心跳包导致前端认为超时了。[这篇文章](http://www.mamicode.com/info-detail-1667562.html)中的异步切换方案完全是完全没有用的，无论是` socketio.sleep(10) `还是` time.sleep(10) `都是没有用的，只会徒增响应时间，使问题加剧。
[此issue](https://github.com/miguelgrinberg/Flask-SocketIO/issues/266)和[这个问题](https://stackoverflow.com/questions/26747253/why-does-my-code-disconnects-a-socket-io-connection-in-node)就是在讨论这个问题。作者给出的建议是尽量使用较短的event handlers，实在不行的话就使用eventlet或gevent线程处理任务。
至于tornado则完全没有缓冲区的问题，发送信息的函数执行后前端会立即收到信息。以下是实例代码。
```python
import tornado.web
import tornado.websocket
import tornado.httpserver
import tornado.ioloop
import time

class WebSocketHandler(tornado.websocket.WebSocketHandler):
    def check_origin(self, origin):
        return True

    def on_message(self, message):
        for x in range(10):
            self.write_message(u"Your message was: " + message)
            print("message sent!")
            time.sleep(20)


class Application(tornado.web.Application):
    def __init__(self):
        handlers = [
            (r'/ws', WebSocketHandler)
        ]

        tornado.web.Application.__init__(self, handlers)


if __name__ == '__main__':
    ws_app = Application()
    server = tornado.httpserver.HTTPServer(ws_app)
    server.listen(8080)
    print(123)
    tornado.ioloop.IOLoop.instance().start()
```
前端
```html
<script type="text/javascript">
    var ws;
    function onLoad(){
        ws = new WebSocket("ws://localhost:8080/ws");
        ws.onmessage = function(e){
            alert(e.data);
        }
    }

    function sendMsg(){
        ws.send(document.getElementById('msg').value);
    }
</script>
<body onload='onLoad();'>
    Message to send:   <input type="text" id="msg" />
      <input type="button" onclick="sendMsg();" value="发送" />
</body>
```
python控制台输出message sent!后前端会立即弹出收到消息，没有缓存的问题。我试过把睡眠时间增加到100秒也是完全没问题的。
## 超时时间
[这篇文章](http://www.cnblogs.com/1wen/p/5808276.html)是阐述地比较清楚的。
## socketio系列的优点
首先当然是可以兼容相当多的浏览器，第二是可以支持emit(event, message)这样指定事件名的发送和响应，而原生websocket只能使用简单的send(message)。如果你需要自定义很多不同类型的消息那么无疑从socketio转移到原生的websocket是十分困难的，通过手动判断数据内容会很消耗性能而且很麻烦，如果通过uri区分不同类型数据，一方面来说管理多个websocket连接是比较麻烦的，另一方面tornado是会给不同的uri添加不同的handler类，handler类的生命周期在client为关闭浏览器之前，换句话说，某一个websocket请求改变了这个handler实例的属性值，在另一个(针对相同URi的)websocket连接中是可以访问这个属性的，在发起开始/停止命令的时候是比较有用的。但跨URI就不行了毕竟不是一个类，不知道设置全局变量可不可行。
