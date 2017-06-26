---
title: websocket踩坑实践
date:
tags:
  - python
keywords:
  - websocket
  - python
  - socketio
  - flask-socketio
categories:
---
用的是flask-socketio，使用的eventlet，前几天被亲自喂了一口翔。每次emit信息的时候，这次“事件”结束后，所有的信息才会被发送。而且貌似“事件”过程中也不会响应心跳包。在前台很长一段时间收不到心跳包的情况下，就会发起重连请求。也就是说，你在一个响应处理的时间过长，虽然仍在处理，但所有的消息都要等到这一个响应全部处理结束之后才会发送，连心跳包都不会发送，前端时间过长会认为是连接断开了，从而发起重连。这时候所有的函数全部从头开始，真的很糟心。以下是示例说明。
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
像以上的代码，前端会一个信息都收不到，因为后台处理耗时太久，前端认为断开了。并且控制台会出现如下错误，然后重连，再次出现这样的错误，一直循环。
```error
test.html:1 XMLHttpRequest cannot load http://127.0.0.1:5000/socket.io/?EIO=3&transport=polling&t=1498485030898-7&sid=9b604f29e92d4b1dad74adc4409ecf45. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'null' is therefore not allowed access. The response had HTTP status code 500.
```
由于我的业务是遍历一个过滤一个列表，比较耗时，一开始对整个列表的过滤写在一个socket.on里面，还以为是有延迟，后面的消息和前面的消息冲突了，没有收到，[这篇文章](http://www.mamicode.com/info-detail-1667562.html)中的解决方案完全是 没有用的，无论是` socketio.sleep(10) `还是` time.sleep(10) `都是没有用的，只会徒增响应时间，使问题加剧。可以试下下面一段代码就知道了，socketio响应还是很快的，但关键是所有信息会在处理完后一起发送，造成了延迟的错觉。
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
      time.sleep(10)
```
[此issue](https://github.com/miguelgrinberg/Flask-SocketIO/issues/266)就是在讨论这个问题。gevent不维护了还真的是个糟心的问题。作者给出的建议是尽量使用较短的event handlers，实在不行的话就使用eventlet或gevent线程处理任务。我的建议是增加socketio的延时时间，如果你对于emit的“延迟”无所谓的话。
