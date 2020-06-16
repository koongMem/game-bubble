# 资源加载 - AssetsManager

### 配置文件default.res.json

![](http://fcmms.midea.com/cmshop-sit/7/20190423/1d85935c-6103-4a63-b5a4-a911fe82bf22.png)

### 资源加载

##### 1.任务监听式：

在Main类中

```
  // 资源的配置文件加载完成
  RES.addEventListener(RES.ResourceEvent.CONFIG_COMPLETE,this.onConfigComplate,this);
  //加载资源配置文件
  RES.loadConfig("resource/default.res.json?",'resource/')
```

资源文件加载完成，才可以开始按组加载资源文件

```
  //资源的配置文件加载完成
  private onConfigComplate(){
     // 资源加载完毕
     RES.addEventListener(RES.ResourceEvent.GROUP_COMPLETE, this.onResourceLoadComplete, this);
     // 资源加载错误
     RES.addEventListener(RES.ResourceEvent.GROUP_LOAD_ERROR, this.onResourceLoadError, this);
     // 资源加载进度
     RES.addEventListener(RES.ResourceEvent.GROUP_PROGRESS, this.onResourceProgress, this);
     // 单个资源加载错误
     RES.addEventListener(RES.ResourceEvent.ITEM_LOAD_ERROR, this.onItemLoadError, this);

     //加载组
     RES.loadGroup('preload')
  }
```

资源加载完成需要将监听事件移除，避免内存泄漏

```
 //组资源加载完成
 private onResourceLoadComplete(event: RES.ResourceEvent){
     RES.removeEventListener(RES.ResourceEvent.GROUP_COMPLETE,this.onResourceLoadComplete,this);
     RES.removeEventListener(RES.ResourceEvent.GROUP_LOAD_ERROR,this.onResourceLoadError,this);
     RES.removeEventListener(RES.ResourceEvent.GROUP_PROGRESS,this.onResourceProgress,this);
     RES.removeEventListener(RES.ResourceEvent.ITEM_LOAD_ERROR,this.onItemLoadError,this);

     this.stage.removeChild(this.LoadingUI);
     clearTimeout(this.timeoutLoading);
     this.createGameScene();
 }
```

##### 2.模块API：

在Main类中

```
// 实例化加载页面
const loadingView = new LoadingUI() 
// 渲染至舞台
this.stage.addChild(loadingView) 
// 加载资源文件
await RES.loadConfig("resource/default.res.json", "resource/") 
// 开始加载组，传入实例化的LoadingUI对象
await RES.loadGroup("preload", 0, loadingView)
```

在LoadingUI类中，每当RES.ResourceEvent.GROUP\_PROGRESS事件触发，会默认调用onProgress方法

    class LoadingUI extends egret.Sprite implements RES.PromiseTaskReporter
    ...
    // 声明、实例化textField
    private textField: egret.TextField = new egret.TextField();
    ...
    // 资源文件加载进度回调
    public onProgress(current: number, total: number): void {
        let per = current * 100 / total
        this.textField.text = `Loading...${per.toFixed(0)}%`
    }

由此次没有loading设计需求，demo：

![](http://fcmms.midea.com/cmshop-sit/classification/20190423/bd3554ab-a65e-45be-b180-3e2386212093.png)

### [音频加载](https://developer.egret.com/cn/article/index/id/156)

egret本身提供了egret.Sound类，配合 AssetsManager即可实现，预加载，自动播放、设置播放次数、播放时间等

1、如果已加载组资源加载好音乐的前提下

```
this.bgSound: egret.Sound = RES.getRes('bg_mp3')
this.bgSound.play(startTime = 0, loops = 0)
```

2、通过Sound加装音频

```
  var sound:egret.Sound = new egret.Sound();
  sound.addEventListener(egret.Event.COMPLETE, function loadOver(event:egret.Event) {
      sound.play();
  }, this);
  sound.addEventListener(egret.IOErrorEvent.IO_ERROR, function loadError(event:egret.IOErrorEvent) {
      console.log("loaded error!");
  }, this);
  sound.load("resource/sound/sound.mp3");
```

3、通过 URLLoader 加装音频

```
  var loader:egret.URLLoader = new egret.URLLoader();
  loader.addEventListener(egret.Event.COMPLETE, function loadOver(event:egret.Event) {
      var sound:egret.Sound = loader.data;
      sound.play();
  }, this);
  loader.dataFormat = egret.URLLoaderDataFormat.SOUND;
  loader.load(new egret.URLRequest("resource/sound/sound.mp3"));
```

PS：google浏览器早已在60+版本就禁止自动播放功能，需要用户行为才能触发播放等一系列操作（_addListerner_）  
另外一种实现方法：

##### 修改浏览器配置

打开 [chrome://flags/](chrome://flags/)进入谷歌页面：![](http://fcmms.midea.com/cmshop-sit/classification/20190424/fefad89e-ca35-4505-9452-84322377959b.png)将Autoplay policy设置为 No user gesture is required....重启就可以了..

##### Electron

修改Electron的谷歌配置，生成可执行文件，这种情形对业务场景要求较高

```
// Chrome 66 之后更新了，自动播放的策略以提供更友好的交互体验。使用 Electron打包的应用不能自动播放音频文件。
app.commandLine.appendSwitch('autoplay-policy', 'no-user-gesture-required');
```



