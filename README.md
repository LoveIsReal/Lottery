To see how it work，you could download the html file and run it on your browser.
If anything wrong, you should give a issue,thanks.






> #### 需求：
奖品数量、文字可以通过修改数据去改变，当然数据也可以是接口获取。
每次抽奖抽出一个奖品，且将奖品从转盘上移除。
适用于奖品数量较少的年会抽奖。


> #### 效果：
![抽奖.gif](https://upload-images.jianshu.io/upload_images/1349448-ccecfcdb6a4de94e.gif?imageMogr2/auto-orient/strip)
##### 效果描述：每次抽奖旋转的角度是1080度和3240度之间的随机值，抽奖每完成一次，奖品立刻从数据里去除，且反向旋转到起始角度（之所以旋转到起始角度是为了方便计算下一次指向的奖品）

> #### 实现
过程略多，我分个步骤

1. 初始化数据
2. 绘制转盘和数据
3. 绘制扇形颜色块
4. 定义随机角度的旋转
5. 奖品判断
6. 奖品删除和重新初始化


### 1. 初始化数据
我的数据类型很简单，对象数组，两个字段，label 和  图片url
![image.png](https://upload-images.jianshu.io/upload_images/1349448-08b4c4dfae90d03b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
这个demo中没有用到图片url，而是用SVG path绘制扇形颜色块，因为我们把绘制扇形背景图放在单独的一篇文章中讲解



数据可以写死在程序中，或者网络接口得到。

### 2. 绘制转盘和数据
转盘整体是一个div，包含转盘主体和转盘中间不动的指针：
![image.png](https://upload-images.jianshu.io/upload_images/1349448-bf143254aa53d638.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
主体和指针都是absolute定位，转盘主体设置border-radius：50%；显示成一个圆。
程序中配置半径radius为指定值（方便以后转盘扩展成响应式）

```
var radius = 400;
```
接着是指针left top的确认  
我们为了方便，就先将指针的宽高写死在程序中（毕竟demo）

```
// 定义pointer的位置
$('.pointer').css('top', (radius / 2 - 110) + 'px');
$('.pointer').css('left', (radius / 2 - 60 + 8) + 'px'); // 8是边框
```
这时一个空的转盘就出来了，接着的事情就是分割线和文字放哪呢？？？  
我们首先解决怎么放，就是确定文字块宽高和位置。  
我的做法是画一个大矩形，absolute定位
![image.png](https://upload-images.jianshu.io/upload_images/1349448-42530ea91eaca282.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
在矩形的上部绘制文字，这样可以最大限度的显示文字。  
宽高：高度是radius，宽度需要用三角函数算一下  

```
var section_width = Math.sin(2 * Math.PI / 360 * 180 / num) * (radius / 2) * 2
要注意的是Math.sin传入的是弧度值
其中：2*Math.PI/360表示1度的弧度值 
```
接着就是矩形的top left值的计算  
top可以看出来为0  
left嘛,是这样的

```
var left = radius / 2 - section_width / 2;// section_width是文字矩形宽度
```
接着是分割线的绘制，这个就是很瘦很瘦的矩形，也是absolute定位  
位置嘛，top 0，left
```
var line_left = radius / 2 - 1; // 因为line的宽度是2px
```
接着就是画多个矩形和分割线，然后分别做不同角度的旋转。  
首先选定一个为旋转基点，比如分割线的基点
```
transform-origin：50% 100%;
```

```
$('.line').eq(index).css('transform', 'rotate(' + (360 / num * index - 180 / num) + 'deg)');
$('.line').eq(index).css('transform-origin', '50% 100%');
$('.section').eq(index).css('transform', 'rotate(' + 360 / num * index + 'deg)');
$('.section').eq(index).css('transform-origin', section_width / 2 + 'px ' + radius / 2 + 'px');
```
矩形的基点就不解释了，自己可以简单算一下。  
经过这些，一个完整的大转盘就出来了，接下来画扇区。




### 3. 绘制扇形颜色块
绘制扇形颜色块我们可以用SVG的path标签去绘制。  
![image.png](https://upload-images.jianshu.io/upload_images/1349448-591a1f9f534c7921.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
先画两条边，然后画一个圆弧。  
最后是：   
![image.png](https://upload-images.jianshu.io/upload_images/1349448-edb389a07f00653d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
这里有必要解释一下圆弧怎么画，它的参数不太好理解   
绘制圆弧用 **A** 指令  
参数7个，依次是：
1. 椭圆X半径 ，   
2. 椭圆Y半径 ，   
3. 椭圆横轴相对于Canvas X轴的偏移角度，  
4. 确定是否是大小圆弧（前面的条件确定之后，我们要想这个椭圆经过起始点和终点有两种情况，每种情况包含两个圆弧，当然两种情况都有一个大圆弧一个小圆弧，这两个大圆弧的区别就是是否是顺时针从起始点绘制到终点）0 ：小圆弧 1 ：大圆弧
5. 确定是否是顺时针 0：逆时针 1：顺时针  
6. 终点X坐标  
7. 终点Y坐标   

我们想绘制上面的圆弧，就得：  

```
$('#path' + (index + 1)).attr('d', 'M' + left + ',' + top_value + ' A' + radius / 2 + ' ' + radius / 2 + ' 0 0 1 ' + (left + section_width) + ' ' + top_value + ' L' + radius / 2 + ' ' + radius / 2 + '');
```

扇区颜色我们就定义了一个颜色数组，每次从0位置开始取颜色，当然每个扇区都得转一次，转的角度和文字矩形一样  


### 4. 定义随机角度的旋转
首先是一个方法，获取一个区间的随机值  

```
var random_deg = Math.floor(Math.random() * (3240 - 1080 + 1) + 1080);
// 这个就是获取3240 - 1080 的随机值，数值是int float我们不管，一样用
```
为了防止旋转的时候还去点击，我们设置一个布尔值isRotating，旋转完成才置为false  
得到了旋转角度，还需要计算一下旋转时间，毕竟不能每次相等的时间吧，这样旋转起来转盘速度看起来差不多。我们就取一圈0.5s。  

```
var random_time = random_deg / 360 * 0.5;
```
现在转起来  

```
$('.circle').css('transition', 'all ' + random_time + 's');
$('.circle').css('transform', 'rotate( ' + total_rotate_deg + 'deg)');
```
采用定时的方式进行抽奖的回调  

```
setTimeout(function () {
 ... 抽奖回调方法
}, random_time * 1000)
```

### 5. 奖品判断
记得我们每次抽奖都要回到起点么，就是为了方便的计算出相对的旋转角度，而且我们还做了一层处理，就是给得到的 **随机旋转角度** 加上了扇形角度的一半，相当于每次旋转没开始我们就将转盘向右掰了半个扇形，那掰了半个扇形之后转盘是什么造型呢？  它的第一个扇区完全在 **第一象限** 了（以圆心为原点），指针指向第一个奖品和最后一个奖品的分割线。  
这样的目的是什么？  

**举个例子**：  五个奖品，我们第一次转了顺时针的1090度，和360取余数，得到绝对的旋转角度，就是相当于起始位置10度，而且我们一开始就掰了半个扇去的角度，想象一下，现在最后一个扇区被指针指到了，那奖品就是最后一个扇去：  

```
var real_deg = (total_rotate_deg - (360 / num / 2)) % 360;
var index = Math.floor(real_deg / (360 / num));
var reward_index = num - index - 1; // 奖品索引值
```

### 6. 奖品删除和重新初始化
删除嘛，不是很简单，用数组的splice方法即可，然后定个时，就定成1s钟后重绘（总得让人看一下刚刚抽中的是啥，当然你也可以改成点击重置），重复执行1 2 3 三个步骤即可。补充说一下：里面的旋转动画是一个很普通的CSS transition。


如有错误，请多指教
（留言区告诉我哦）

