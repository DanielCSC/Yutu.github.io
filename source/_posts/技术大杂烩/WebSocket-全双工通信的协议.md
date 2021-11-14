---
title: WebSocket--全双工通信协议
categories:
  - 技术大杂烩
tags:
  - WebSocket
date: 2020-12-24 10:48:55
---





# 概述

> `WebSocket`是一种在单个`TCP`连接上进行全双工通信的协议.
>
> `WebSocket`使得客户端和服务器之间的数据交换变得更加简单，允许`服务端主动`向`客户端推送数据`。在`WebSocket API`中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。



## 特点



- 建立在 TCP 协议之上，服务器端的实现比较容易。

- 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

- 数据格式轻量，性能开销小。

- 没有同源限制，客户端可以与任意服务器通信
- 协议标识符是ws（如果加密，则是wss），请求的地址就是后端支持websocket的API。

> ```
> ws://localhost:8000/websocketlink/
> ```



## 几种与服务端实时通信的方法

1. **AJAX轮询**

> AJAX轮询也就是定时发送请求，也就是普通的客户端与服务端通信过程，只不过是无限循环发送，这样，可以保证服务端一旦有最新消息，就可以被客户端获取。

2. **Long Polling长轮询**

> Long Polling长轮询是客户端和浏览器保持一个长连接，等服务端有消息返回，断开。
> 然后再重新连接，也是个循环的过程，无穷尽也。。。
>
> 客户端发起一个Long Polling，服务端如果没有数据要返回的话，会hold住请求，等到有数据，就会返回给客户端。客户端又会再次发起一次Long Polling，再重复一次上面的过程。



3. **缺点**

>这两种方式都有个致命的弱点，开销太大，被动性。假设并发很高的话，这对服务端是个考验。
>
>而WebSocket一次握手，持久连接，以及主动推送的特点可以解决上边的问题，又不至于损耗性能。



# WebSocket连接过程

> 客户端发起HTTP握手，告诉服务端进行WebSocket协议通讯，并告知WebSocket协议版本。服务端确认协议版本，升级为WebSocket协议。之后如果有数据需要推送，会主动推送给客户端。
>
> 连接开始时，客户端使用HTTP协议和服务端升级协议，升级完成后，后续数据交换遵循WebSocket协议





1. 先搂一眼Request Headers



+++danger Request Headers

```html
Accept-Encoding: gzip, deflate, br		支持的数据压缩格式
Accept-Language: zh-CN,zh;q=0.9		浏览器所希望的语言种类
Cache-Control: no-cache			缓存控制，服务器通过控制浏览器要不要缓存数据
Connection: Upgrade				表示要升级协议
Host: localhost:8000				访问的主机名和端口
Origin: http://localhost:8080			发送请求的主机名和端口
Pragma: no-cache				指定“no-cache”值表示服务器必须返回一个刷新后的文档，即使它是代理服务器而且已经有了页面的本地拷贝
Sec-WebSocket-Key: xdT5lKSuUI5vXFucwiG9Ig==	对应服务端响应头的Sec-WebSocket-Accept，由于没有同源限制，websocket客户端可任意连接支持websocket的服务。这个就相当于一个钥匙一把锁，避免多余的，无意义的连接。
Sec-WebSocket-Version: 13			表示websocket的版本。如果服务端不支持该版本，需要返回一个Sec-WebSocket-Versionheader，里面包含服务端支持的版本号。
Upgrade: websocket				升级协议到websocket协议
```

+++



2. Response Headers

+++danger Response Headers



```html
Connection: Upgrade				
Sec-WebSocket-Accept: LKbtQ+1FimQ7NVxE5GJkpRMtFXo=		 用来告知服务器愿意发起一个websocket连接， 值根据客户端请求头的Sec-WebSocket-Key计算出来
Upgrade: websocket
```



+++



# WebSocket API

1.  客户端与服务端建立连接。

```js
var ws = new WebSocket("ws://localhost:8000/websocketlink/");
```



2. 返回的实例对象的属性

```js
WebSocket.onopen		// 连接成功后的回调
WebSocket.onclose		// 连接关闭后的回调
WebSocket.onerror		// 连接失败后的回调
WebSocket.onmessage		// 客户端接收到服务端数据的回调
webSocket.bufferedAmount	// 未发送至服务器的二进制字节数
WebSocket.binaryType		// 使用二进制的数据类型连接
WebSocket.protocol		// 服务器选择的下属协议
WebSocket.url			// WebSocket 的绝对路径
```



3. 方法

```js
WebSocket.close()		// 关闭当前连接
WebSocket.send(data)		// 向服务器发送数据
```





# Django + Vue +WebSocket实现聊天室



## Django

1. 安装,注册WebSocket

```python
pip install dwebscoket

INSTALLED_APPS = [
    'dwebsocket',
]
```



2. `views.py`

```python
from .models import *
from dwebsocket.decorators import accept_websocket

# 接收前端信息
@accept_websocket
def test_socket(request):
    if request.is_websocket():
        for message in request.websocket:
            c = eval(str(message, encoding='utf-8'))
            message = c.get('message')
            uid = c.get('uid')
            username = c.get('username')
            chat = Chat.objects.create(user_a=int(uid), user_b=4)
            messageobj = Message.objects.create(name=username, message=message, chat_id=chat.id)
            request.websocket.send(json.dumps(message))


# 主动推送消息
@accept_websocket
def test_websocket2(request):
    uid = request.GET.get('uid')
    # print(uid)

    if request.is_websocket():
        while 1:
            time.sleep(1) 
            chatobj = Chat.objects.filter(user_a=uid, user_b=4).all().order_by('id')
            try:
                a = []
                for i in chatobj:
                    message = Message.objects.filter(chat_id=i.id).first()
                    data = {"name": message.name, "message": message.message}
                    a.append(data)
                request.websocket.send(json.dumps(a))
            except Exception as e:
                print(e)
```



## Vue

1. 用户界面

```html
<template>
<div>
    <a-button type="primary" @click="showDrawer">
        客服
    </a-button>

    <a-drawer title="聊天室" width="600" placement="right" :closable="false" :visible="visible" @close="onClose">

        <a-row>
            <a-col :span="22 ">
                <table>
                    <tr v-for="i in message">
                        <th>{ { i.name:} }</th>
                        <td>{ { i.message } }</td>
                    </tr>
                </table>
            </a-col>

        </a-row>

        <a-row>
            <a-col :span="22 ">
                <a-input v-model="value" placeholder="请输入内容" />
            </a-col>

            <a-col :span="2">
                <a-button @click='sendmessage()' type="primary">发送</a-button>
            </a-col>
        </a-row>
    </a-drawer>
</div>
</template>

<script>
export default {
    data() {
        return {
            visible: false,
            value: '',
            message: '',
        };
    },
    mounted() {
        // 获取聊天记录
        
        var _this = this;
        //判断浏览器是否支持websocket
        if ("WebSocket" in window) {
            console.log("支持");
            
            // 连接服务器
            var ws = new WebSocket("ws://localhost:8000/user/websocket/?uid=" + localStorage.getItem('uid'));
            
            // 向服务器发送数据
            ws.onopen = function () {
                ws.send('xxx')
            }
            
            // 获取服务器返回数据
            ws.onmessage = function (evt) {
                var received_msg = evt.data;
                let data = [JSON.parse(received_msg)]
                _this.message = data[0]
            }
            
            //捕获断开链接
            ws.onclose = function () {
                console.log("链接已经关闭");
            }
            
        }
    },
    methods: {

        showDrawer() {
            this.visible = true;
        },
        onClose() {
            this.visible = false;
        },
        
		// 向服务器发送数据
        sendmessage() {
            var _this = this
            // 判断浏览器是否支持websocket
            if ("WebSocket" in window) {
                console.log("支持");
                
                // 连接服务器
                var ws = new WebSocket("ws://localhost:8000/user/socket/");
                
                // 向服务器发送数据
                ws.onopen = function () {
                    var data = JSON.stringify({
                        username: localStorage.getItem('username'),
                        message: _this.value,
                        uid: localStorage.getItem('uid')
                    })
                    ws.send(data);
                }
                
                // 获取服务器返回数据
                ws.onmessage = function (evt) {
                    // console.log(evt.data)
                }
                
                // 捕获断开链接
                ws.onclose = function () {
                    console.log("链接已经关闭");
                }
            };
        },
    }
};
</script>
```



2. 客服界面

```html
<template>
<div>
    <a-button type="primary" @click="showDrawer">
        客服
    </a-button>

    <a-drawer title="聊天室" width="600" placement="right" :closable="false" :visible="visible" @close="onClose">

        <a-row>
            <a-col :span="22 ">
                <table>
                    <tr v-for="i in a">
                        <th>{ { i.name } }:</th>
                        <td>{ { i.message } }</td>
                    </tr>
                </table>
            </a-col>
        </a-row>

        <a-row>
            <a-col :span="22 ">
                <a-input v-model="value" placeholder="请输入内容" />
            </a-col>
            <a-col :span="2">
                <a-button @click='sendmessage()' type="primary">发送</a-button>
            </a-col>
        </a-row>
        
    </a-drawer>
</div>
</template>

<script>
export default {
    data() {
        return {
            visible: false,
            value: '',
            message: '',
        };
    },
    mounted() {
        var _this = this;
        // 判断浏览器是否支持websocket
        if ("WebSocket" in window) {
            console.log("支持");
            
            // 连接服务器
            var ws = new WebSocket("ws://localhost:8000/user/websocket/?uid=" + localStorage.getItem('uid'));
            
            // 向服务器发送数据
            ws.onopen = function () {
                ws.send("xxx");
            }
            
            // 获取服务器返回数据
            ws.onmessage = function (evt) {
                var received_msg = evt.data;
                let data = [JSON.parse(received_msg)]
                _this.message = data[0]
            }
            
            // 捕获断开链接
            ws.onclose = function () {
                console.log("链接已经关闭");
            }
            
        }
    },
    methods: {

        showDrawer() {
            this.visible = true;
        },
        onClose() {
            this.visible = false;
        },
		
        sendmessage() {
            var _this = this
            // 判断浏览器是否支持websocket
            if ("WebSocket" in window) {
                console.log("支持");
                
                // 连接服务器
                var ws = new WebSocket("ws://localhost:8000/user/socket/");
	
                // 向服务器发送数据
                ws.onopen = function () {
                    var data = JSON.stringify({
                        username: '客服',
                        message: _this.value,
                        uid: localStorage.getItem('uid')
                    })
                    ws.send(data);
                }
				
                // 获取服务器返回数据
                ws.onmessage = function (evt) {
                    console.log(evt.data)
                }
                
                // 捕获断开链接
                ws.onclose = function () {
                    console.log("链接已经关闭");
                }

            };
        },
    }
};
</script>
```



3. 效果

![](聊天室.gif)