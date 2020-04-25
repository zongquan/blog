---
layout: post
title:  "Flash AS游戏引擎原理"
author: "Zongquan"
comments: true
---

其实一提到游戏引擎，很多初次接触游戏行业的人，会显得有一种畏惧感，会觉得这是一个非常高深的东西。曾经页游行业很乱，就像现在的手游一样，各种非计算机专业出生的同学，经过一个短暂的培训就开始了程序员之旅，编写游戏逻辑为主。

这篇文章仅仅从简单的角度讲解一下Flash游戏引擎在显示渲染上的原理，用位图渲染的方式来实现，供新手们学习交流，如有不足的，望指出。包括在Cocos 2D、Unity3D，其实都是大同小异。

最简单的图形引擎，主要三个部分：**摄影机（Camera）**，**场景画布（Canvas）**，**画布中的显示对象（Displayobject）**。

![](/assets/images/flash_game_engine_1.jpeg)

为了便于大家理解，我直接把Camera都忽略掉，假设我们FlashPlayer的整个窗口就是Camera，就剩下了这个样子。

![](/assets/images/flash_game_engine_2.jpeg)

Flash本身就是一款图形引擎，但现在我抛弃掉Flash的渲染，重新按自己的方式通过位图来渲染我们的游戏，来让大家更容易理解。

### 首先，定义Canvas画布：

假设我们的游戏帧频是基于FlashPlayer的。

```java
	public class Canvas extends Sprite
	{
		
		//画布上的位图  
		private var _canvasBitMap:Bitmap = null;
		
		//画布上的位图数据
		private var _canvasBitMapData:BitmapData = null;
		
		//画布上的显示对象
		private var _objList:Array = null;
		
		
		public function Canvas()
		{
			addEventListener(Event.ENTER_FRAME,onEnterFrame);
			
			//假设我们画布是1000*600的大小
			_canvasBitMapData = new BitmapData(1000,600);
			_canvasBitMap = new Bitmap(_canvasBitMapData);
			addChild(_canvasBitMap);
		}
		
		
		//每一帧的心跳入口
		private function onEnterFrame(e:Event):void
		{
			update();
			draw();
		}
		
		//数据的更新
		private function update():void
		{
			for(var i:int = 0;i<_objList.length;i++)
			{
				_objList[i].update();
			}
		}
		
		//渲染
		private function draw():void
		{
			clear();
			for(var i:int = 0;i<_objList.length;i++)
			{
				_objList[i].draw(_canvasBitMapData);
			}
		}
		
		//清空上一帧的位图数据
		private function clear():void
		{
			
		}
		
		//往画布上添加一个显示对象
		private function addChild(obj:Displayobject):void
		{
			_objList.push(obj);
		}
		
		
	}
```

### 其次，定义Displayobject

```java
	public class Displayobject
	{
		
		//显示对象自己的位图数据
		private var _bitMapData:BitmapData = null;
		
		public var x:Number;
		public var y:Number;
		public var height:Number;
		public var width:Number;
		
		public function Displayobject()
		{
			
		}
		
		//更新显示对象数据
		public function update():void
		{
			
		}
		
		//渲染显示对象
		public function draw(data:BitmapData):void
		{
			data.copyPixels(_bitMapData,); //将自己的位图数据拷贝到画布上
		}
		
	}
```

就这么简单的两个类，我们的基于位图的渲染引擎的出来了

### update()

```java
public function update():void
```

这个接口与draw()是在每一帧的时候都会调用，我们一般用它来处理一些什么东西呢？
1.  位置的更新，也就是坐标的更新，每一个显示对象在画布都有坐标，假设它是一个匀速运动的小车，那我们就可以根据它的运行速度，每一帧的时间，得到它当前的坐标。
2.  碰撞的检测，我们也可以把一些碰撞的检测放到这里来处理。
3. 动作的播放，比如我们有一个动作序列表，[1,2,3,4,5,6] 每一帧我们是播一个序列值，这个序列值是多少，决定了_bitMapData的具体计算方式。

### draw()

```java
public function draw(data:BitmapData):void
		{
			data.copyPixels(_bitMapData,); //将自己的位图数据拷贝到画布上
		}
```

这个接口的目的，主要是将**update()**计算好后的一个新的位图数据，把它画到画布场景中去，上面给的copyPixels是不对的，这样写的目的是告诉大家，拷贝这个位图数据到画布中去，主要与新的位图数据的坐标和宽高有关。

一个简单的游戏引擎，其实就是这个样子，再加上IO，声音，Socekt，然后再把它用MVC模式合理化，就可以拿来开发游戏了。

引擎的渲染，也就是draw()是非常消耗CPU资源的，我们常常会在这些地方做一些优化，比如我们有一些显示对象很明显能算出它是会被前面的显示对象完全档住，那这种对象我们是不是可以考虑在draw的时候不渲染它呢。 

现在Flash已经支持硬件加速了，在actionscript里增加了一个stage3D，只要我们理解到了图像引擎的原理，我们自己尝试去写一个游戏引擎来练习练习也不难的，同样我们拿到一款成熟的引擎比如Starling的时候，要把它理解清楚也很容易。