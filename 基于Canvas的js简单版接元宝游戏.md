本游戏是基于canvas的简单版接元宝游戏v1.0.0版本，往后还会进一步完善，游戏代码git地址：https://github.com/luqiren/Canvas.git

里面的gold_v1.0.0.html就是这个游戏的代码了。废话不多说，下面先介绍涉及代码的关于canvas的一些知识，然后再介绍游戏的实现。

1.涉及到的相关canvas知识介绍

如果对canvas已经很熟悉了可以忽略这一段。

<canvas>是HTML5的标签，<canvas>可以说是一个画布，其本身并没有绘制图像的能力，所以我们只能通过脚本来绘图，游戏中所有的图像都是利用getContext()方法返回的一个对象来进行绘制的，该对象提供了用于在画布上绘图的方法和属性。

如何获取画图对象？

<canvas id="canvas" width="400" height="500"></canvas>
var canvas = document.getElementById('canvas');
var ctx = canvas.getContext('2d');
这个ctx就是用来绘图，这个游戏中的绘图任务都交由它来完成。另外你可以用以下代码来判断自己的浏览器支不支持canvas

var canvas = document.getElementById('canvas');
if (canvas.getContext) {
    console.log('你的浏览器支持Canvas!');
} else {
    console.log('你的浏览器不支持Canvas!');
}
如何在画布上画图片？

游戏中的小人、礼物和炸弹都是用的现成的图片，所以需要知道如何在画布上绘制图片。

使用drawImage()方法可以将图片画到画布上，这个方法有9个参数，但有些参数是可选的

参数	描述
img	规定要使用的图像、画布或视频。	 
sx	可选。开始剪切的 x 坐标位置。
sy	可选。开始剪切的 y 坐标位置。
swidth	可选。被剪切图像的宽度。
sheight	可选。被剪切图像的高度。
x	在画布上放置图像的 x 坐标位置。
y	在画布上放置图像的 y 坐标位置。
width	可选。要使用的图像的宽度（伸展或缩小图像）。
height	可选。要使用的图像的高度（伸展或缩小图像）。
根据这些参数的组合，将图片画到画布上共有三种方式：

第一种：将图片完全画在画布上，只规定在画布上的位置，不规定大小不剪切图片。

ctx.drawImage(img,x,y);
第二种：规定图片在画布上的位置和大小，不剪切图片。

ctx.drawImage(img,x,y,width,height);
第三种：剪切图片，并将剪切后的图片定位在画布上并且设置大小。

ctx.drawImage(img,sx,sy,swidth,sheight,x,y,width,height);
游戏中用的就是第三种。

如何在画布上写一段文字？

游戏结束时在画布上写GAME OVER，这时就要用到fillText()来写文字了。

ctx.font="30px Arial";
ctx.fillText("Hello World",10,50);
第一行代码用来设置文字的大小和字体，第二行就设置文字内容以及在画布的位置，先x坐标然后y坐标。

如何设置渐变？

为了字体能够好看些，我们设置个渐变

var grd=ctx.createLinearGradient(0,0,170,0);
grd.addColorStop(0,"red");
grd.addColorStop(0.25,"orange");
grd.addColorStop(0.5,"yellow");
grd.addColorStop(0.75,"blue");
grd.addColorStop(1,"purple");
ctx.fillStyle=grd;
ctx.fillRect(20,20,150,100);
效果：



这是一段设置渐变效果的代码，看代码和效果应该能猜到怎么使用了，使用context.createLinearGradient(x0,y0,x1,y1);设置渐变，参数如下：

参数	描述
x0	渐变开始点的 x 坐标
y0	渐变开始点的 y 坐标
x1	渐变结束点的 x 坐标
y1	渐变结束点的 y 坐标


使用gradient.addColorStop(stop,color)可以设置渐变的颜色，参数如下：

参数	描述
stop	介于 0.0 与 1.0 之间的值，表示渐变中开始与结束之间的位置。
color	在 stop 位置显示的颜色。


2.接元宝游戏实现

简单版的接元宝游戏里就有小人、元宝、炸弹三个实体类，另外还加了一个小人接到炸弹后爆炸的类，前面三个实体类都有一些共同属性，所以写一个GameObject游戏对象基类包括共有的属性：x坐标、y坐标、速度、宽度和高度，让前三个实体类通过对象冒充来继承GameObject。

游戏过程很简单，通过操作键盘的向左←向右→两个键来操作小人左右移动来接元宝，而元宝和炸弹从上面随机一个地方开始下落，小人的任务就是接住元宝，避开炸弹，小人接到炸弹则游戏结束。

游戏中的几个实体类：

GameObject（游戏对象类）

        function GameObject (x, y, speed, w, h) {
            this.x = x;
            this.y = y;
            this.speed = speed;
            this.w = w;
            this.h = h;
            this.isLive = true;
        }
People（小人）



        function People (x, y, speed, w, h) {
            this.score = 0;
            this.gameObject = GameObject;
            this.gameObject(x, y, speed, w, h);
     
            this.moveLeft = function () {
                this.x = this.x - this.speed < 0 ? 0 : this.x - this.speed;
            }
     
            this.moveRight = function () {
                this.x = this.x + this.speed + this.w > 400 ? 400 : this.x + this.speed;
            }
        }
Money（元宝）

        function Money (x, y, speed, w, h) {
            this.gameObject = GameObject;
            this.gameObject(x, y, speed, w, h);
            this.timer = null;
            this.score = 5;
            
            this.run = function () {
                if (this.y > 500) {
                    window.clearInterval(this.timer);
                    this.isLive = false;
                } else {
                    this.y += this.speed;
                }
            }
        }
Bomb（炸弹）

        /** 炸弹类 */
        function Bomb (x, y, speed, w, h) {
            this.gameObject = GameObject;
            this.gameObject(x, y, speed, w, h);
            this.timer = null;
     
            this.run = function () {
                if (this.y > 500) {
                    window.clearInterval(this.timer);
                    this.isLive = false;
                } else {
                    this.y += this.speed;
                }
            }
        }
Boom（爆炸）

        /** 爆炸类 */
        function Boom (x, y) {
            this.x = x;
            this.y = y;
            this.isLive = true;
            this.blood = 0;
            this.bloodUp = function () {
                if (this.blood <= 4) {
                    this.blood++;
                } else {
                    this.isLive = false;
                }
            }
        }
实体类已经有了，小人我们可以一开始就创建好，而从天而降的元宝和炸弹需要每隔一段时间生成然后下落，所以还需要两个生成元宝和炸弹，生成后还要启动其定时器。代码如下：

       function moneyFactory () {
            var money = new Money(Math.random() * (400 - 32), 0, 5, 40, 40);
            moneyArray.push(money);
            window.setInterval('moneyArray[' + (moneyArray.length - 1) + '].run()', 50);
        }

 

        /** 生成炸弹 */
        function bombFactory () {
            var bomb = new Bomb(Math.random() * (400 - 32), 0, 5, 40, 40);
            bombArray.push(bomb);
            window.setInterval('bombArray[' + (bombArray.length - 1) + '].run()', 50);
        }
使用Math.random()来随机生成x坐标。

实体和生成实体的函数都有了，接下来就是将这些实体画到画布上了。

        /** 刷新游戏地图 */
        function flashMap () {
            ctx.fillStyle = '#eee';
            ctx.fillRect(0, 0, 400, 500);
     
            drawPeople(people);
     
            catchMoney();
            catchBomb();
     
            drawMoney();
            drawBomb();
            drawBoom();
        }
        
        function drawPeople (people) {
            people.isLive && ctx.drawImage(peopleImg, 0, 0, 780, 1070, people.x, people.y, people.w, people.h);
        }
     
        function drawMoney () {
            moneyArray.forEach(function(item) {
                if (item.isLive) {
                    ctx.drawImage(yuanbao, 0, 0, 512, 512, item.x, item.y, item.w, item.h);
                }
            })
        }
     
        function drawBomb () {
            bombArray.forEach(function (bomb) {
                if (bomb.isLive) {
                    ctx.drawImage(bombImg, 160, 160, 480, 480, bomb.x, bomb.y, bomb.w, bomb.h);
                }
            })
        }
     
        /** 画接住炸弹的爆炸过程 */
        function drawBoom () {
            boomArray.forEach(function(boom) {
                if (boom.isLive) {
                    ctx.drawImage(boomImg, boom.blood * 64, 0, 64, 64, boom.x, boom.y, people.w, people.h);
                    boom.bloodUp();
                } else {
                    // 将爆炸过程画完再游戏结束清空画布
                    gameOver();
                }
            })
        }
接下来就是编写游戏流程的代码，接住礼物积分增加，接住炸弹爆炸游戏结束。

        /** 接住礼物 */
        function catchMoney () {
            moneyArray.forEach(function(item) {
                if (item.isLive) {
                    if (isCatch(item)) {
                        people.score += item.score;
                        item.isLive = false;
                        window.clearInterval(item.timer);
                        flashScore();
                    }
                }
            })
        }
     
        /** 接住了炸弹 */
        function catchBomb () {
            bombArray.forEach(function (bomb) {
                if (bomb.isLive) {
                    if (isCatch(bomb)) {
                        bomb.isLive = false;
                        people.isLive = false;
                        window.clearInterval(bomb.timer);
                        var boom = new Boom(people.x, people.y);
                        boomArray.push(boom);
                    }
                }
            })
        }
     
        /** 判断物品（礼物、炸弹）有没有进入小人范围 */
        function isCatch (item) {
            return (item.x + (item.w / 2) <= people.x + people.w && item.x + (item.w / 2) >= people.x && item.y + (item.h / 2) <= people.y + people.h && item.y + (item.h / 2) >= people.y);
        }
     
        /** 刷新积分 */
        function flashScore () {
            var score = document.getElementById('score');
            score.innerHTML = '积分: ' + people.score;
        }
     
        /** 游戏结束 */
        function gameOver () {
            /** 游戏结束关闭所有定时器 */
            window.clearInterval(moneyTimer);
            window.clearInterval(bombTimer);
            window.clearInterval(flashTimer);
     
            moneyArray.forEach(function (item) {
                clearInterval(item.timer)
            })
     
            bombArray.forEach(function (item) {
                clearInterval(item.timer)
            })
     
            ctx.fillStyle = '#eee';
            ctx.fillRect(0, 0, 400, 500);
     
            ctx.font = '50px Verdana';
            var gradient = ctx.createLinearGradient(0, 0, 500, 0);
            gradient.addColorStop('0', 'magenta');
            gradient.addColorStop('0.5', 'blue');
            gradient.addColorStop('1', 'red');
     
            ctx.fillStyle = gradient;
            ctx.fillText('GAME OVER!', 50, 200);
        }
然后就是在进入页面的时候先生成两个元宝，然后定义三个定时器，分别用来定时刷新画布，这样就能造成元宝炸弹小人移动的感觉，另外两个定时器就是定时生成元宝和炸弹，代码如下：

        onload = function () {
            var canvas = document.getElementById('canvas');
            ctx = canvas.getContext('2d');
            ctx.fillStyle = '#eee';
            ctx.fillRect(0, 0, 400, 500);
     
            peopleImg = document.getElementById('people');
            yuanbao = document.getElementById('money');
            bombImg = document.getElementById('bomb');
            boomImg = document.getElementById('boom');
     
            for (var i = 0; i < 2; i++) {
                var money = new Money(Math.random() * (400 - 32), 0, 5, 40, 40);
                moneyArray.push(money);
                window.setInterval('moneyArray[' + i + '].run()', 50);
            }
     
            moneyTimer = window.setInterval('moneyFactory()', 5000);
            bombTimer = window.setInterval('bombFactory()', 10000);
            flashTimer = window.setInterval('flashMap()', 50);
        }
最后要让小人动起来，还需要接受键盘事件：


        /** 键盘事件 */
        document.onkeydown = function () {
            var e = event || window.event || arguments.callee.caller.arguments[0];
            var flag = true;
            switch (e.keyCode) {
                case 37:
                    if (people.x > 0) {
                        people.moveLeft();
                    }
                    break;
                case 39:
                    if (people.x + people.w < 400) {
                        people.moveRight();
                    }
                    break;
                default:
                    flag = false;
                    break;
            }
            flag && flashMap();
        }
效果图（由于没有找到合适的元宝图片，所以就先用礼物图片代替元宝了^_^）







由于篇幅原因一些定义全局变量和html代码就不贴出来了，在git上可以看到完整代码，这个是接元宝游戏的第一个版本，只是简单的实现了游戏过程，往后还会完善和升级。
————————————————
版权声明：本文为CSDN博主「luqiren」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/luqiren/article/details/82783001