# 网络通信 - [WebSocket](http://www.runoob.com/html/html5-websocket.html)

> WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。
>
> WebSocket是一个相对比较新的规范。虽然它早在2011年底就实现了规范化，但即便如此，在Web浏览器和应用服务器上依然没有得到一致的支持。Firefox和Chrome早就已经完整支持WebSocket了，但是其他的一些浏览器刚刚开始支持WebSocket。
>
> 服务器端对WebSocket的支持也好不到哪里去。GlassFish在几年前就开始支持一定形式的WebSocket，但是很多其他的应用服务器在最近的版本中刚刚开始支持WebSocket。

### 使用WebSocket+STOMP+SockJS实现网络通信

### [SockJS](https://github.com/sockjs/sockjs-client)

> SockJS是一个浏览器JavaScript库，它提供了一个类似于网络的对象。SockJS提供了一个连贯的、跨浏览器的Javascript API，它在浏览器和web服务器之间创建了一个低延迟、全双工、跨域通信通道。
>
> 1、提供向下兼容方案: WebSocket、HTTP流和HTTP长轮询（按优秀选择的顺序分为3类）
>
> 2、自动启动可配置的心跳消息，防止长时间未通信而断开链接
>
> 3、对后端配置更为友好

### [STOMP](https://github.com/jmesnil/stomp-websocket)

> 直接使用WebSocket（或SockJS）就很类似于使用TCP套接字来编写Web应用。因为没有高层级的线路协议（wire protocol），因此就需要我们定义应用之间所发送消息的语义，还需要确保连接的两端都能遵循这些语义。
>
> STOMP在WebSocket之上提供了一个基于帧的线路格式（frame-based wire format）层，用来定义消息的语义。
>
> 与HTTP请求和响应类似，STOMP帧由命令、一个或多个头信息以及负载所组成。例如，如下就是发送数据的一个STOMP帧：

```
>>> SEND               // STOMP命令是send
transaction:tx-0       // 表示消息的的事务机制
destination:/app/marco // 表示消息要发送到哪里的目的地
content-length:20      // 表示包含了负载的大小

{"message":"Hello World!"} // 负载内容
```

### [Egret中使用WebSocket](https://developer.egret.com/cn/apidoc/egret/name/egret.WebSocket)

与普通WebSocket使用原理一致，需要注意的是：

1、连接服务器

```
//官方提供的
//连接服务器方式
this.socket.connect("echo.websocket.org", 80);

// 如果遇到链接是类似于ws://10.16.68.179:9113/cmimp/ws/defend-appliances/
// 请使用this.socket.connectByUrl
```

2、数据传输问题

[官方demo](https://developer.egret.com/cn/apidoc/egret/name/egret.WebSocket)是通过二进制传输。

如果需要使用字节流，会耗费更大传输量：

并需将传输格式设置为默认的字符串

```
 // this.socket.type = egret.WebSocket.TYPE_BINARY;  注释掉就是了
```

在接收数据可以看到，接收的数据在\_readMessage字段![](http://fcmms.midea.com/cmshop-sit/classification/20190424/890e9b6d-e05a-43a1-93e6-184dfdcc4d0c.png)官方封装的方法，

读取：

![](http://fcmms.midea.com/cmshop-sit/classification/20190424/77ab5300-c1e3-4102-9a2e-a5faa6fa0afc.png)

发送：

![](http://fcmms.midea.com/cmshop-sit/classification/20190424/757fdd7a-cbda-45fd-bb39-a1db214d2ea4.png)

![](http://fcmms.midea.com/cmshop-sit/classification/20190424/b9fb24ce-7f3e-44ce-b668-13adaca1e581.png)

### Egret中使用STOMP+SockJS

1、[需要引入第三方sockJs，stomp](/mu-lu/egret.md#功能模块)

项目中使用的是直接导入官方js文件，生成d.ts（接口声明文件）

生成方法，如果市面上有，直接ctrl+c，ctrl+v；如果缺少，需要查看第三方源代码，将其中模块对应导出，根据类型声明写法不同，

sockjs.d.ts

```
declare var SockJS: {
    new (s:string):any;
};
```

stomp.d.ts

```
declare var Stomp: {
    VERSIONS;
    Frame;
    client(url, protocols); //类似 class 的静态函数
    over(ws);
};
```

在egretProperties.json添加引用路径

```
{
  "name": "stomp",
  "path": "libs/stomp"
},
{
  "name": "sockjs",
  "path": "libs/sockjs"
}
```

[引用方法](https://www.jianshu.com/p/b8aa70bf1340)

    ...
    // 初始化
    private createWebSocket():void {
        let that = this;
        let socket = new SockJS(that.url);
        // 获取 STOMP 子协议的客户端对象
        that.stompClient = Stomp.over(socket);
        // 向服务器发起websocket连接并发送CONNECT帧
        that.stompClient.connect(
            {
                gameId: that.gameId,
                platform: 'tv',
                id: `tv${n.GameData.role}_${this.tvId}`
            },
            function connectCallback(frame) {
                // 连接成功时（服务器响应 CONNECTED 帧）的回调方法
                // console.log("连接成功");
                that.onSocketOpen();
                that.stompClient.subscribe('/user/topic/defend-appliances', function (response) {
                    that.toReceiveMessage(response.body);
                });

            },
            function errorCallBack(error) {
                // 连接失败时（服务器响应 ERROR 帧）的回调方法
                // console.log("连接失败");
                that.onSocketError();
            }
        );
    }
    ...





