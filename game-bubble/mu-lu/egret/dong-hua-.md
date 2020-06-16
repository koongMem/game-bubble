# [动画 - Tween](https://developer.egret.com/cn/apidoc/index/name/egret.Tween)

```
/// 代码段 B
var obj = { x:0 };

var funcChange = function():void{
    console.log( this.x );
}

egret.Tween.get( obj, { onChange:funcChange, onChangeObj:obj } )
.to( {x:600}, 1000 , egret.Ease.backInOut )
.call( function(){});
```

> 以下这种书写方式， 在下一次运动需要通过上一次运动结果得出的情况下，会触发不了call回调函数，需要增加wait延迟。

 ```
   let tw = egret.Tween.get(obj);
   let speed;
   let x;
   let twAni = () => {
       tw
       .to( {x: x}, speed)
       .call(function(){
           speed++;
           x++;
           twAni();
       });
   }
 ```

> 这种情况建议以第一种书写方法来执行，不对动画对象定义化~~let tw = egret.Tween.get\(obj\);~~

#### tween实现贝塞尔曲线运动

```
/**
  * 二次贝塞尔曲线运动  
*/
public get factor():number {
  return 0;
}
/**
  * 二次贝塞尔曲线运动  
  * 公式 (1 - t)^2 P0 + 2 t (1 - t) P1 + t^2 P2
  * P0 P1 02 形成曲线的三个点
*/
public set factor(value:number) {
  this.x = (1 - value) * (1 - value) * this.pos.x + 2 * value * (1 - value) * (n.GameData.shellPos.x + this.diameter) + value * value * n.GameData.shellPos.x;
  this.y = (1 - value) * (1 - value) * this.pos.y + 2 * value * (1 - value) * (n.GameData.shellPos.y - this.diameter * 2) + value * value * n.GameData.shellPos.y;
}


/**
  * 执行动画 
*/
...
//在1秒内，this的factor属性将会缓慢趋近1这个值，这里的factor就是曲线中的t属性，它是从0到1的闭区间。
egret.Tween.get(this).to({factor: 1}, 1000);
...
```

鉴于tween除了基础的动画外（Linear缓动都没？？？）

这里有个民间大佬给的[WTween缓动动画](http://www.u3d8.com/?p=1571)

