# 游戏主要算法

主要介绍bean文件夹中的显示对象算法

### 泡泡GridNode

泡泡网格gridNodeList

> 所有格子实例化之后，均存储在此队列中，往后全部操作均在此队列中，网格的列，宽，大小，死亡线，初始渲染行数等均在GameData中配置

```
/**
* 存放格子数据
*/
public static gridNodeList: Array<Array<GridNode>>
```

泡类型管理

```
enum GridNodeStatus {
    NORMAL_BALL, // 普通球
    REWARD_BALL, // 奖励球
    SHELL_BALL // 发射球
}
```

泡泡位置pos

> 管理泡泡位置\(x, y\)，根据index索引代入计算公式得出，泡泡后期的运动，爆炸动画显示位置等均用到
>
> x = x缩进值 + 列数 \* 直径; y = 行数 \* 直径 - y缩进值 \* 行数
>
> x缩进值 = 半径
>
> ![](http://fcmms.midea.com/cmshop-sit/classification/20190426/eacbb65e-ceb5-4f0a-b65d-0fc14e42b8ee.png)
>
> y缩进值 = 直径 - 直径\* Math.cos\(\(30 \* Math.PI\) / 180\);  //重叠部分的高度 cos30°的值： 3相同的圆相切，60度，换算成直角就是30度
>
> ![](http://fcmms.midea.com/cmshop-sit/classification/20190426/1f444e7a-c1c0-434d-bcc7-e9e4d8b15a8a.png)

```
/**
* 格子的坐标
*/
private pos: Point
```

泡颜色管理

> 根据类型的不同，在不同的颜色数组中获取图片索引
>
> 颜色生成由大屏端，在任意一个游戏端渲染资源完成时，生成一个可设置的行列二元数组
>
> 包括发射球颜色，惩罚泡泡颜色，均从这个数组中获取，确保两个游戏端颜色一致

    /**
    * 颜色
    */
    public color
    /**
    * 颜色对象
    */
    public grid
    /**
      *设置泡泡颜色，生成对应颜色图片，并添加到显示对象的容器中
    */
    public setColor(color?: string){
        this.color = color;
        this.removeChild(this.grid)、
        this.grid = GameUtil.createBitmapByName(`ball_${this.color}_png`)
        ...
        this.addChild(this.grid);
    }

泡索引管理

```
/**
* 格子数组中的下标
*/
public index
```

##### ⭐**邻居管理**

> 对泡泡相邻六个球做管理，一元数组，存储相邻六个泡泡的index值，用于
>
> 碰撞、泡泡爆炸、泡泡掉落等重要算法提供基础

```
/** 球相邻neibors的索引
*   1  2
* 0  球  3
*   5  4 
* dir < 0 轨迹向左  > 0 轨迹向右
*/
public neibors = new Array<Point>(6)
```

##### ⭐**泡泡爆炸**

> 利用**责任链算法**
>
> 先检索泡泡是否邻居是否有一样颜色泡泡
>
> 有则激活对应邻居泡泡的检索函数，无则跳过，让每一颗相邻泡泡查询自身以邻居颜色，直到积累的相同颜色数 n 达到爆炸临界点 max 时，即可返回通知最开始的泡泡，时间复杂度\[2， 8\]
>
> 当积累的相同颜色数n === max，则激活最开始泡泡爆炸函数，同样使用责任链算法，让每一个泡泡负责其以及其邻居的爆破行为；

##### ⭐**泡泡掉落**

> 在执行爆破行动之后，遍历剩余全部泡泡，通过特定的标识数 linkId，同样利用责任链算法的思路，让每颗球对彼此直接有联系（有相邻关系）标记上linkId，往后再被同一个linkId查询到则返回，不执行标记事件。
>
> 同时维护一个数组gridNodeTempList管理这些被标记的球，最后gridNodeList网格数据中存在的均是没有被标记上的泡泡，这些泡泡就是掉落的泡泡，执行其掉落函数，再将gridNodeTempList赋值回gridNodeList

```
/**
 * 自行查询连接关系
*/
public getLink(linkId){
    let neibors = this.neibors
    let gridNodeList = n.GameData.gridNodeList; 

    this.linkId = linkId;

    for(let i = 0, len = neibors.length; i < len; i++){
        let neibor = neibors[i];
        if(neibor.x < 0){// 顶部球的，必定黏住
            continue;
        }else if(neibor.x >= 0 && neibor.x < n.GameData.rowOver && neibor.y >= 0 && neibor.y < n.GameData.col){
            if(gridNodeList[neibor.x] && gridNodeList[neibor.x][neibor.y]){
                let node = gridNodeList[neibor.x][neibor.y];
                if(node.linkId === linkId){// 已经被查询过
                    continue;
                }
                if(node.getLink(linkId)){ // 交付邻居泡泡继续执行任务
                    if(!n.GameData.gridNodeTempList[neibor.x]){
                        n.GameData.gridNodeTempList[neibor.x] = new Array<GridNode>(n.GameData.col)
                    }
                    n.GameData.gridNodeTempList[neibor.x][neibor.y] = gridNodeList[neibor.x][neibor.y]
                    delete gridNodeList[neibor.x][neibor.y];
                }
            }
        }
    }

    return true;// 继续
}
```

### 发射器Launcher

发射器

> 完成发射球的装填动画，根据H5手柄发送的信息，旋转发射器方向，重置发射轨迹出射点，出射角度等

**⭐发射轨迹**

> 每次移动，计算好即将运动的轨迹，并绘制出来
>
> 通过三角函数，得出轨迹与左右两边墙壁的碰撞点，直到发生碰撞或者碰撞点到达顶部，即y = 0，如图
>
> ![](http://fcmms.midea.com/cmshop-sit/classification/20190425/8d62691f-fe16-41e5-8288-bc4400b9218b.png)
>
> 在绘制过程，对每一个绘制的线段，
>
> 已知：最高点\(x1, y1\)与最低点\(x2, y2\)
>
> 得出碰撞行数范围 \[A, B\]
>
> 以及线段方程
>
> l1 =&gt; \(x-x1\)/\(x2-x1\)=\(y-y1\)/\(y2-y1\)，
>
> 求：这个范围的泡泡进行判断是否发生碰撞。
>
> 根据另一道直线方程 l2 =&gt; Ax+By+C=0，代入l1方程求出ABC的值，
>
> 由圆到直线的最短距离
>
> ![](https://gss2.bdstatic.com/-fo3dSag_xI4khGkpoWK1HF6hhy/baike/s%3D102/sign=24e840d0f7d3572c62e298dcb8136352/d52a2834349b033b758cebe310ce36d3d539bd86.jpg)
>
> 如果小于泡泡半径，则可排除；
>
> 第一个发生碰撞的泡泡，即是碰撞点，此时只要将检测的线段最高点修改为碰撞点的坐标即可
>
> ![](http://fcmms.midea.com/cmshop-sit/classification/20190425/9f830a71-7339-459e-bc4e-5ce0da9c9017.png)
>
> 并且，将运动轨迹的每段点存储起来，以供之后发射运动使用

# 

> 当然，**官方也提供了一套\[碰撞检测API\]**\([https://developer.egret.com/cn/article/index/id/105](https://developer.egret.com/cn/article/index/id/105)\)
>
> 可以在tween动画中的onChange触发回调时，进行检测（不过碰撞检测中开启精准碰撞灰常耗性能）
>
> ```
> egret.Tween.get( obj, { onChange:funcChange, onChangeObj:obj } )
> ```
>
> 下面是可以检测俩物体是否发生碰撞，官方文档目前查不到，因为一查就报错~~~ 👻
>
> ```
> /**检测碰撞*  传两个物体对象*/
>     public static hitTest(obj1:egret.DisplayObject,obj2:egret.DisplayObject):boolean
>     {
>         var rect1: egret.Rectangle = obj1.getBounds();
>         var rect2: egret.Rectangle = obj2.getBounds();
>         rect1.x = obj1.x;
>         rect1.y = obj1.y;
>         rect2.x = obj2.x;
>         rect2.y = obj2.y;
>         return rect1.intersects(rect2);
>     }
> ```

# 

> 得出发生碰撞的泡泡之后，就要计算泡泡是会落在泡泡的哪个位置，
>
> ```
> /*
> *   1  2
> * 0  泡  3
> *   5  4 
> */
> ```
>
> 这时还要加上发射方向 dir 进行考虑，将泡泡分为是个象限，根据发射方向，坐标轴 \* -1 ，进行水平旋转
>
> ```
> /**
> * (x<xr,y<yr)  |(x>=xr,y<yr)          |           
> *      2       |     1             1  |  2           (xr, yr)为被碰撞泡泡的圆点坐标  
> * —————————————O———————————      —————O————— 水平    (x, y)为线段与圆相交的点坐标
> *      3       |     4             4  |  3 
> * (x<xr,y>=yr) |(x>=xr,y>=yr)         |      
> *                              
> * →→→→→→→→→→(dir>0)→→→→→→→→→→   ←←(dir>0)←←    
> */
> ```
>
> 根据线段与圆的两个交点，入射点\(x1, y1\)，出射点\(x2, y2\)，即可判断坐落在那个位置，之后判断该位置是否有球，再进行位置微调
>
> 类似下图从左到右三条射线，球最后应该分别落在4、3、水平/2的位置
>
> ![](http://fcmms.midea.com/cmshop-sit/classification/20190425/baba71d3-cfb6-44c5-84c6-69bd2efa75a3.png)
>
> > ps:
> >
> > 俩交点可以通过求根公式** b² - 4ac &gt;= 0**，可以求出；
> >
> > abc则可以通过圆方程\(x - oa\)² +\(y - ob\)² = r² 与上方得出的直线方程，代入求出二元一次方程，得出

发射球运动

> 只要根据运动轨迹轨迹使用tween动画，运动完毕之后执行泡泡爆炸计算以及对应的分数、掉落计算即可



