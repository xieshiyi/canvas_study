前面了解了如何使用canvas实现刮刮卡抽奖，以及canvas最基本的一些api方法。
这里开始一步步实现大转盘抽奖，大转盘是一个非常简单且实用的web特效，五脏俱全，其中涉及的知识点有 `圆的绘制及非零环绕原则，路径的绘制`，`canvas`，`transform`，`逐帧动画 requestAnimationFrame`方法，接下来开始实现。
先把整体代码贴出来，再一步步讲解每一块关键代码的作用。
*示例版本为ES6.请在现代浏览器下运行以下代码*
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>大转盘</title>
</head>
<body>
    <div id="spin_button" style="position: absolute;left: 232px;top: 232px;width: 50px;height: 50px;line-height: 50px;text-align: center;background: yellow;border-radius: 100%;cursor: pointer">旋转</div>
    <canvas id="canvas" width="500" height="500"></canvas>
</body>
<script>
    let canvas = document.getElementById('canvas'),
        context = canvas.getContext('2d'),

        OUTSIDE_RADIUAS = 200,   // 转盘的半径
        INSIDE_RADIUAS = 0,      // 用于非零环绕原则的内圆半径
        TEXT_RADIUAS = 160,      // 转盘内文字的半径

        CENTER_X = canvas.width / 2,
        CENTER_Y = canvas.height / 2,

        awards = [               // 转盘内的奖品个数以及内容
            '大保健', '话费10元', '话费20元', '话费30元', '保时捷911', '周大福土豪金项链', 'iphone 20', '火星7日游'
        ],

        startRadian = 0,                             // 绘制奖项的起始角，改变该值实现旋转效果
        awardRadian = (Math.PI * 2) / awards.length, // 每一个奖项所占的弧度

        duration = 4000,     // 旋转事件
        velocity = 10,       // 旋转速率
        spinningTime = 0,    // 旋转当前时间
        spinTotalTime,       // 旋转时间总长
        spinningChange;      // 旋转变化值的峰值


    /**
     * 缓动函数，由快到慢
     * @param {Num} t 当前时间
     * @param {Num} b 初始值
     * @param {Num} c 变化值
     * @param {Num} d 持续时间
     */
    function easeOut(t, b, c, d) {
        if ((t /= d / 2) < 1) return c / 2 * t * t + b;
        return -c / 2 * ((--t) * (t - 2) - 1) + b;
    };

    /**
     * 绘制转盘
     */
    function drawRouletteWheel() {
        // ----- ① 清空页面元素，用于逐帧动画
        context.clearRect(0, 0, canvas.width, canvas.height);
        // -----

        for (let i = 0; i < awards.length; i ++) {
            let _startRadian = startRadian + awardRadian * i,  // 每一个奖项所占的起始弧度
                _endRadian =   _startRadian + awardRadian;     // 每一个奖项的终止弧度

            // ----- ② 使用非零环绕原则，绘制圆盘
            context.save();
            if (i % 2 === 0) context.fillStyle = '#FF6766'
            else             context.fillStyle = '#FD5757';
            context.beginPath();
            context.arc(canvas.width / 2, canvas.height / 2, OUTSIDE_RADIUAS, _startRadian, _endRadian, false);
            context.arc(canvas.width / 2, canvas.height / 2, INSIDE_RADIUAS, _endRadian, _startRadian, true);
            context.fill();
            context.restore();
            // -----

            // ----- ③ 绘制文字
            context.save();
            context.font = 'bold 16px Helvetica, Arial';
            context.fillStyle = '#FFF';
            context.translate(
                CENTER_X + Math.cos(_startRadian + awardRadian / 2) * TEXT_RADIUAS,
                CENTER_Y + Math.sin(_startRadian + awardRadian / 2) * TEXT_RADIUAS
            );
            context.rotate(_startRadian + awardRadian / 2 + Math.PI / 2);
            context.fillText(awards[i], -context.measureText(awards[i]).width / 2, 0);
            context.restore();
            // -----
        }

        // ----- ④ 绘制指针
        context.save();
        context.beginPath();
        context.moveTo(CENTER_X, CENTER_Y - OUTSIDE_RADIUAS + 8);
        context.lineTo(CENTER_X - 10, CENTER_Y - OUTSIDE_RADIUAS);
        context.lineTo(CENTER_X - 4, CENTER_Y - OUTSIDE_RADIUAS);
        context.lineTo(CENTER_X - 4, CENTER_Y - OUTSIDE_RADIUAS - 10);
        context.lineTo(CENTER_X + 4, CENTER_Y - OUTSIDE_RADIUAS - 10);
        context.lineTo(CENTER_X + 4, CENTER_Y - OUTSIDE_RADIUAS);
        context.lineTo(CENTER_X + 10, CENTER_Y - OUTSIDE_RADIUAS);
        context.closePath();
        context.fill();
        context.restore();
        // -----
    }

    /**
     * 开始旋转
     */
    function rotateWheel() {
        // 当 当前时间 大于 总时间，停止旋转，并返回当前值
        spinningTime += 20;
        if (spinningTime >= spinTotalTime) {
            console.log(getValue()); return
        }

        let _spinningChange = (spinningChange - easeOut(spinningTime, 0, spinningChange, spinTotalTime)) * (Math.PI / 180);
        startRadian += _spinningChange

        drawRouletteWheel();
        window.requestAnimationFrame(rotateWheel);
    }

    /**
     * 旋转结束，获取值
     */
    function getValue() {
        let startAngle = startRadian * 180 / Math.PI,       // 弧度转换为角度
            awardAngle = awardRadian * 180 / Math.PI,

            pointerAngle = 90,                              // 指针所指向区域的度数，该值控制选取哪个角度的值
            overAngle = (startAngle + pointerAngle) % 360,  // 无论转盘旋转了多少圈，产生了多大的任意角，我们只需要求到当前位置起始角在360°范围内的角度值
            restAngle = 360 - overAngle,                    // 360°减去已旋转的角度值，就是剩下的角度值

            index = Math.floor(restAngle / awardAngle);     // 剩下的角度值 除以 每一个奖品的角度值，就能得到这是第几个奖品

        return awards[index];
    }


    window.onload = function(e) {
        drawRouletteWheel();
    }

    document.getElementById('spin_button').addEventListener('click', () => {
        spinningTime = 0;                                // 初始化当前时间
        spinTotalTime = Math.random() * 3 + duration;    // 随机定义一个时间总量
        spinningChange = Math.random() * 10 + velocity;  // 随机顶一个旋转速率
        rotateWheel();
    })
</script>
</html>
```

#### 🚶思路
1. 当页面加载时会执行 `drawRouletteWheel()`方法，这个方法将通过 `starRadian` ,`awardRadian`,`awards`等全局变量， 完成转盘的所有绘制操作，包括：转盘，奖品选块，指针；
2. 定义点击事件，当点击旋转按钮，执行`rotateWheel()`方法，该方法将动态改变全局变量`starRadian`的值，并调用`window.requestAnimationFrame()`方法实现逐帧旋转动画。

#### 绘制大转盘
我们进入`drawRouletteWheel()`方法，可以看到，该方法分为4步：
1. 清空页面内的所有元素
2. 绘制转盘
3. 绘制文字
4. 绘制指针

* 清空页面所有元素
之所以在绘制最开始对画布做清理，是为了完成逐帧动画。
我们可以想象一下，我们可以在很多页纸上画一个小人不同的行走状态，然后通过快速翻阅这些纸张，小人就会神奇的‘动’起来，你翻得越快，小人就跑的越快。
在canvas中，或者说在js中实现动画，同样是这个道理，我们就想像每一页纸就是动画里的每一帧，我们翻页的操作，在电脑屏幕上，实际就是清空整个画布了。

* 绘制圆盘
我们通过全局变量`awards`这个数组，指定了奖项的显示文字；
通过全局变量`startRadian`指定了起始角的弧度，也就是0°；
通过`awardRadian`指定了每一个奖品选块所占的弧度，该值是通过360°的弧度值除以奖品的个数计算来的。
