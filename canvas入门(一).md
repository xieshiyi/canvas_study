## 发现canvas的有趣之处，可以在画布上任意作画，好像自己也可以成为小画家

### canvas入门
* **context 绘图环境** 
> 我们可以通过canvas标签新建一个画布，这个canvas画布使得我们在浏览器中操作画布内的元素来实现绘制图形、动画等功能。想要操作  canvas画布里的内容，只能使用JavaScript，其他都是无用的。>

想要操作canvas，首先要获取这个canvas对象
``` 
    <canvas id="canvas" width="250" height="50" ></canvas>
```
```
    //获取 canvas 对象
    let canvas = document.getELementById('canvas');
    //获取 canvas 绘图环境对象，参数为 2d
    context = canvas.getContext('2d');
```
context 是绘图环境对象，如果你在浏览器中输出 context，你会发现里面包含了许多属性，如：`fillStyle`  `strokeStyle`等，以及一些函数方法，`arc()` `rect()` `save()` `restore()` `clip()`等等。


* **绘制基本图形** 
#### 矩形
`context.rect($x,$y,$width,$height)`

> 矩形是最简单的绘制；

参数：
1. $x,$y: 分别对应矩形左上角在canvas画布上的坐标
2. $width ,$height: 矩形的宽高

*值得注意的是*，调用这个方法，~只是绘制矩形的路径~，该路径是不可见的，除非你在后面调用`context.fill()` 和 `context.stroke` 方法，进行填充或者描边。

填充描边的表现取决于 context 绘图环境的 `fillStyle` 和 `strokeStyle` 的值。

#### 圆形
`context.arc($x,$y,$radius,$start_radian,$end_radian,$clockwise)`

圆形比起矩形稍微复杂一点点，但也很简单。

参数：
1. $x,$y:代表圆心在canvas画布上的坐标点
2. $radius: 圆的半径   不是直径！
3. $start_radian,$end_radian:起始角和终止角，他们接收的值只能是弧度。

> 提示一下圆周率，在javascript中，Math.PI就是代表 π ；
> 简单的说，圆有360°，而 π 是180°。如果画一整个圆，就是从 0° 到 360° ，Math.PI*2;如果画半圆的话，就是 0° 到 180° ， 
> Math.PI;

4. $closewise: 布尔值，false表示圆由顺时针绘制，true表示圆由逆时针绘制。

和矩形相同，这个方法也只是绘制了一个路径，想要在页面显示，任然是需要调用 context.fill() 和 context.stroke() 两个方法。

#### 当前路径
`context.beginPath()`

我们在创建一个集合图形之前，都应当先调用该方法，该方法会清除上一次绘制是留下的路径，并将本次绘制的路径最为~当前路径~。

#### 绘制矩形和圆
```
let canvas = document.getElementById('canvas');
context = canvas.getContext('2d');
/**
 * 绘制一个矩形
 * @param {Num} x      矩形的左上角 x 轴的位置
 * @param {Num} y      矩形的左上角 y 轴的位置
 * @param {Num} width  矩形的宽度
 * @param {Num} height 矩形的高度
 */
 function drawRect(x, y, width, height) {
    context.beginPath();               // 清空上一次的路径
    context.rect(x, y, width, height); // 创建一个矩形路径
    context.fill();                    // 将该矩形路径填充为绘图环境指定的填充颜色 context.fillStyle
}
/**
 * 绘制一个圆形
 * @param {Num} centerX  圆心 x 轴坐标
 * @param {Num} centerY  圆心 y 轴坐标
 * @param {Num} radius   圆的半径
 */
function drawCircle(centerX, centerY, radius) {
    context.beginPath();
    context.arc(centerX, centerY, radius, 0, Math.PI * 2, false);
    context.fill();
}

drawRect(0, 0, 100, 100);
drawCircle(160, 50, 50);
```

#### 清除画布内容
`context.clearRect($x,$y,$width,$height)`

该方法，擦除规定矩形中的所有内容；参数同绘制矩形是一样的，只不过这是清除！

#### canvas 的状态
canvas 的状态包括了：
1. 当前坐标变换信息，`transform` 的东西
2. 剪辑区域 `context.clip()`
3. 所有绘图环境（context）属性，也就刚刚提到的，`fillStyle[规定填充的颜色]` `strokeStyle[规定描边的颜色]`，这些状态决定的 canvas 中绘制的图像的展示效果，他们是全局的，也就是说一旦规定了fillStyle为红色，那么除非在 javascript 后面重置这个属性，否则你将来绘制出来的几何图形永远都是红色的。
```
context.fillStyle = 'red';
drawRect(0, 0, 100, 100);   // -> red
drawRect(110, 0, 100, 100); // -> red

context.fillStyle = 'blue';
drawRect(220, 0, 100, 100); // -> blue

```
#### canvas 的状态保存与恢复
前面已经提到过，canvas 的状态是什么。
而这里引出的是绘图环境中的两个很重要的方法，save() 和 restore()。
> 如果有个需求，我想要先绘制一个红色的矩形，接着绘制一个蓝色的矩形，最后再绘制一个红色的矩形

我们知道 javascript 是逐条同步执行的，那么要实现上面的需求，我们需要先定义绘图环境的填充颜色为红色，绘制了第一个红色矩形后，再定义绘图环境的填充颜色为蓝色，绘制，再定义绘图环境的填充颜色为红色，就像这样：

```
context.fillStyle = 'red';
drawRect(0, 0, 100, 100);   // -> red

context.fillStyle = 'blue';
drawRect(110, 0, 100, 100); // -> blue

context.fillStyle = 'red';
drawRect(220, 0, 100, 100); // -> red

```
*是不是觉得这样做非常的傻~*
此时 save() 和 restore() 就站出来说:不！
`context.save()`：能将执行该方法之前的所有 *canvas 状态~* 保存下来；
`context.restore()` ：将 save 方法保存的 *canvas 状态~* 释放出来；
所以上面弱智的写法，我们可以写成这样:
```
context.fillStyle = 'red';
drawRect(0, 0, 100, 100);    // - red 

context.save();

context.fillStyle = 'blue';
drawRect(110, 0, 100, 100);  // -> blue

context.restore();
drawRect(220, 0, 100, 100);  // -> red

```
save 方法保存的是 `context.fillStyle = 'red'` 这个状态，当设置这个状态为蓝色时，context 绘图环境的 `fillStyle` 属性变成了蓝色，当执行 `restore` 方法时，又被恢复成了红色。

#### 剪辑区域 clip()
context 绘图环境对象提供了一个 `context.clip()` 剪辑区域方法，这也是本章实现橡皮擦功能的核心方法。

该方法会将 ~当前路径~ 作为一个剪辑区域，使该区域以外的图像不可见。

并且在剪辑区域中，所有填充，清除等操作不会影响剪辑区域以外的内容。

所以我们可以创建一个剪辑区域，在这个剪辑区域中执行清除操作，就可以擦除 canvas 画布中指定的内容。

```
drawRect(0, 0, 100, 100);

context.save();

// 创建一个圆形路径，并作为剪辑区域，擦除该区域中的所有内容
drawCircle(0, 0, 50);
context.clip();
context.clearRect(0, 0, canvas.width, canvas.height);

context.restore();

drawRect(110, 0, 100, 100);
```