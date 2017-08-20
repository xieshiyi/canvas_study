#### 开始实现橡皮擦
> 上一节讨论过canvas的基础知识，在这里做一个demo，刮刮乐。
🚶  思路：
1. 定义一个公众绘制剪辑区域的方法 `drawEraser` ,该方法会将当前的路径作为剪辑路径绘制，并 清除该路径内的所有内容；
2. 手指按下时，出发 `touchstart` 事件，开启 `dragging` 状态，调用 `drawEraser()` 方法，绘制第一个剪辑区域；
3. 手指移动过程中，不断的调用 `drawEraser()` 方法，绘制剪辑区域，实现橡皮擦效果。
```
<!DOCTYPE html>
<html>

<head>
    <title>橡皮擦</title>
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1.0, user-scalable=no">
</head>

<body>
    <canvas id="canvas" width="300" height="300" style="background: #eee">
        Canvas not supported
    </canvas>
    <script>
        var canvas = document.getElementById('canvas'),
            context = canvas.getContext('2d'),
            ERASER_SIZE = 15,  // 橡皮擦的大小
            dragging = false;  // 是否处在拖动状态
        /**
         * 转换坐标值
         * 将鼠标点击或移动时获取的坐标值，减去 canvas 相对窗口的坐标值，就是在 canvas 画布中的坐标值
         * @param {Obj} e 手指当前相对窗口的坐标位置
         */
        function windowToCanvas(e) {
            let x = e.targetTouches[0].clientX,
                y = e.targetTouches[0].clientY,
                bbox = canvas.getBoundingClientRect();

            return {
                x: x - bbox.left,
                y: y - bbox.top
            }
        }
         /**
         * 绘制剪辑区域，并清除该区域中的内容
         * @param {Obj} loc 手指当前相对 canvas 画布中的坐标位置
         */
        function drawEraser(loc) {
            context.save();
            context.beginPath();
            context.arc(loc.x, loc.y, ERASER_SIZE, 0, Math.PI * 2, false);
            context.clip();
            context.clearRect(0, 0, canvas.width, canvas.height);
            context.restore();
        }
        /**
         * 页面加载时，绘制一个铺满 canvas 画布的矩形
         * 该矩形用于被擦除
         */
        window.onload = function (e) {
            context.save();
            context.fillStyle = '#666';
            context.beginPath();
            context.fillRect(0, 0, canvas.width, canvas.height);
            context.restore();
        }
        /**
         * ① 手指按下时，开启 dragging 状态，并绘制剪辑区域
         */
        canvas.addEventListener('touchstart', function (e) {
            var loc = windowToCanvas(e);
            dragging = true;
            drawEraser(loc);
        })
        /**
         * ② 手指移动时，不断进行剪辑区域的绘制，以及路径更新，实现擦除的效果
         */
        canvas.addEventListener('touchmove', function (e) {
            var loc;

            if (dragging) {
                loc = windowToCanvas(e);
                drawEraser(loc);
            }
        })
        /**
         * ③ 手指离开，结束擦除过程
         */
        canvas.addEventListener('touchend', function (e) {
            dragging = false;
        })
    </script>
</body>
</html>

``` 

#### 结语

* 使用 canvas 实现橡皮擦功能是非常简单的。我们可以把橡皮擦的形状换成矩形，多边形，五角星等等；只需要改变绘制剪辑区域的路径就行了；

* 刮刮卡的功能，也完全是由橡皮擦功能实现的，我们可以把 canvas 的背景图片设置为开奖的内容，文字等，用一个数组将这些图片的路径包含进去，每次加载页面时，随机调用一个数组元素；