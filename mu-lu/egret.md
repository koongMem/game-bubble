# [Egret](https://developer.egret.com/cn/)

> 简介

```
Egret是一套HTML5游戏开发解决方案
```

## 开发工具

[Egret Wing](https://www.egret.com/products/wing.html)

支持主流开发语言与技术的编辑器

通过可视化编辑,提高游戏开发效率

支持 Node.js 开发扩展插件，更好的定制化自有内容

## 开发语言

[TypeScript](https://www.tslang.cn/docs/home.html)

## 命令行版本

5.2.14

## 功能模块

```
egret 必备的核心库
game  制作游戏会用到的类库，比如MovieClip序列帧动画
tween 动画缓动类
assetsManager 资源管理器，完成加载资源，资源预加载 （5.1版本开始支持）

# 第三方
stomp 面向消息的简单文本协议
sockjs SockJS 是 WebSocket 技术的一种模拟。为了应对许多浏览器不支持WebSocket协议的问题
```

🔗 [第三方模块引入教程](https://www.jianshu.com/p/c3cb7548a4a3)         🔗 [生成d.ts（声明文件）文件教程](https://developer.egret.com/cn/docs/page/698)

## 目录结构

```
┗ src
  ┣ common // 存放一些共用的类
  ┃ ┗ GameUtil.ts // 游戏工具类，获取图片、舞台宽高等
  ┃ ┗ KeyBoardUtil.ts // 键盘事件捕获
  ┃ ┗ SocketUtil.ts // websocket通讯管理
  ┣ game // 游戏相关
  ┃ ┣ bean // 一些bean
  ┃ ┃ ┣ BallScore.ts // 得分器
  ┃ ┃ ┣ ColorNumber.ts // 分数值
  ┃ ┃ ┣ GridNode.ts // 单个泡泡网格
  ┃ ┃ ┣ Launcher.ts // 发射器，发射轨迹，完成发射动画
  ┃ ┃ ┣ Point.ts // 圆点（x，y）
  ┃ ┣ scene // 游戏场景
  ┃ ┃ ┣ BaseScene.ts // base场景，所有场景继承这个
  ┃ ┃ ┣ EndScene.ts // 结束场景
  ┃ ┃ ┣ PlayScene.ts // 游戏场景
  ┃ ┃ ┗ StartScene.ts // 开始场景
  ┃ ┣ GameData.ts // 存放游戏数据
  ┃ ┗ SceneControlloer.ts // 场景控制器，可理解为分发器
  ┣ LoadingUI.ts // 加载页
  ┣ Main.ts // 游戏主类（入口，所有场景都放在这个上面显示）
  ┗ Platform.ts // 可用于定义一些window上的对象，接口（比如微信登录等），暂时用不
```

目录结构根据下图而成，egret的[视觉编程](https://developer.egret.com/cn/article/index/id/566)逻辑

> 基础底层视觉为底，场景scene作为容器，可以放置各自相关的显示对象bean

![](http://sedn.egret.com/ueditor/20150527/5565305cb55a6.png)

