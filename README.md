Javascript Tetris
=================

An HTML5 Tetris Game

 * [play the game](http://codeincomplete.com/projects/tetris/)
 * read a [blog article](http://codeincomplete.com/posts/2011/10/10/javascript_tetris/)
 * view the [source](https://github.com/jakesgordon/javascript-tetris)

>> _*SUPPORTED BROWSERS*: Chrome, Firefox, Safari, Opera and IE9+_

FUTURE
======

 * menu
 * animation and fx
 * levels
 * high scores
 * touch support
 * music and sound fx


License
=======

[MIT](http://en.wikipedia.org/wiki/MIT_License) license.

-----------------

# Javascript俄罗斯方块

## HTML5俄罗斯方块游戏

* 玩游戏
* 阅读博客文章
* 查看源
>> _*支持的浏览器*: Chrome, Firefox, Safari, Opera and IE9+

### 游戏在桌面浏览器上玩耍效果最佳。抱歉，没有移动支持。

在这里玩耍：https://xiaozhu2021.github.io/javascript-tetris/
-----------

## 未来

* 菜单

* 动画和效果

* 等级

* 高分数

* 触摸支持

* 音乐和声音效果

## 执照

MIT许可证。


---------------
# blog

Javascript俄罗斯方块
2011年10月10日，星期一
 

我以前的游戏蛇，花了一段时间才出货。了解完成游戏的意义是一个很好的经验，但是实际上，这些只是我与一些基本游戏机制一起玩并学习一些游戏编程的辅助项目。如果我在每个小型项目上都花那么多时间，那么我将在2035年继续这样做……

…考虑到这一点，我需要确保我的下一场比赛是一个简短的周末项目，所以我选择了一个简单的项目，并且我没有花任何时间来完善它……

俄罗斯方块
现在玩游戏
查看源代码
详细了解其工作原理
如果没有修饰，它会有点丑陋，但是功能齐全，我可以向您展示如何自己实现它……

实施细节

俄罗斯方块是一个非常容易实现的游戏，只是带有一些内联css / javascript的简单html文件。

唯一棘手的部分是处理7个Temino的旋转。

您可以尝试在运行时进行数学旋转，但是您会很快发现一些片段围绕中心块（j，l，s，t和z）旋转，而另一些围绕块之间的点（i和o）旋转）。


您可以对两种不同的行为进行特殊说明，或者可以接受以下事实：俄罗斯方块经过硬编码，始终具有7个4旋转的片段，并且只需预先对所有28个模式进行硬编码。

如果假设所有块在概念上都布置在4x4网格上（每个单元是否被占用），则可以避免使用其他特殊情况的代码。有趣的是4x4 = 16，“是否已占用” = 1或0。

听起来我们可以将每个模式表示为一个简单的16位整数，以确切定义我们希望每个片段旋转的方式：



var i = { blocks: [0x0F00, 0x2222, 0x00F0, 0x4444], color: 'cyan'   };
var j = { blocks: [0x44C0, 0x8E00, 0x6440, 0x0E20], color: 'blue'   };
var l = { blocks: [0x4460, 0x0E80, 0xC440, 0x2E00], color: 'orange' };
var o = { blocks: [0xCC00, 0xCC00, 0xCC00, 0xCC00], color: 'yellow' };
var s = { blocks: [0x06C0, 0x8C40, 0x6C00, 0x4620], color: 'green'  };
var t = { blocks: [0x0E40, 0x4C40, 0x4E00, 0x4640], color: 'purple' };
var z = { blocks: [0x0C60, 0x4C80, 0xC600, 0x2640], color: 'red'    };
然后，我们可以提供一个给定的辅助方法：

上面的一件
旋转方向（0-3）
俄罗斯方块网格上的位置
…将遍历该块将占据的俄罗斯方块网格中的所有单元：

function eachblock(type, x, y, dir, fn) {
  var bit, result, row = 0, col = 0, blocks = type.blocks[dir];
  for(bit = 0x8000 ; bit > 0 ; bit = bit >> 1) {
    if (blocks & bit) {
      fn(x + col, y + row);
    }
    if (++col === 4) {
      col = 0;
      ++row;
    }
  }
};
有效件定位
当左右滑动一块或将其放下时，我们需要注意边界检查。我们可以在eachblock帮助器的基础上提供一种occupied方法，如果将一块放置在俄罗斯方块网格上某个位置（具有特定旋转方向）所需的任何块都被占用或超出范围，则该方法返回true：

function occupied(type, x, y, dir) {
  var result = false
  eachblock(type, x, y, dir, function(x, y) {
    if ((x < 0) || (x >= nx) || (y < 0) || (y >= ny) || getBlock(x,y))
      result = true;
  });
  return result;
};

function unoccupied(type, x, y, dir) {
  return !occupied(type, x, y, dir);
};
注意：假定getBlock返回true或false表示俄罗斯方块网格上的该位置是否已被占用。

随机化下一块
选择下一个随机片段是一个有趣的难题。如果我们以纯粹随机的方式选择，例如：

var pieces = [i,j,l,o,s,t,z];
var next = pieces[Math.round(Math.random(0, pieces.length-1))];
…我们发现游戏非常令人沮丧，因为我们经常得到相同类型的长序列，有时在相当长的一段时间内都没有得到想要的作品。

在俄罗斯方块中挑选下一块的标准方法似乎是想象一个袋子，每个袋子有4个实例，我们从袋子中随机取出一个物品，直到它变空，然后冲洗并重复。

这样可以确保每28个片段中至少出现4次，也确保同一片段最多只能重复出现4次…从技术上讲，它最多可以重复8次，因为我们可以一个袋子末尾有一个4的链，然后在下一个袋子的开头有一个4的链，但是这种可能性很小。

这使得游戏更具可玩性，因此我们实现如下randomPiece方法：

var pieces = [];
function randomPiece() {
  if (pieces.length == 0)
    pieces = [i,i,i,i,j,j,j,j,l,l,l,l,o,o,o,o,s,s,s,s,t,t,t,t,z,z,z,z];
  var type = pieces.splice(random(0, pieces.length-1), 1)[0]; // remove a single piece
  return { type: type, dir: DIR.UP, x: 2, y: 0 };
};
一旦我们有了数据结构和辅助方法，游戏的其余部分将变得非常简单。

游戏常量和变量
我们声明一些不变的常量：

var KEY     = { ESC: 27, SPACE: 32, LEFT: 37, UP: 38, RIGHT: 39, DOWN: 40 },
    DIR     = { UP: 0, RIGHT: 1, DOWN: 2, LEFT: 3, MIN: 0, MAX: 3 },
    stats   = new Stats(),
    canvas  = get('canvas'),
    ctx     = canvas.getContext('2d'),
    ucanvas = get('upcoming'),
    uctx    = ucanvas.getContext('2d'),
    speed   = { start: 0.6, decrement: 0.005, min: 0.1 }, // seconds until current piece drops 1 row
    nx      = 10, // width of tetris court (in blocks)
    ny      = 20, // height of tetris court (in blocks)
    nu      = 5;  // width/height of upcoming preview (in blocks)
以及可能会reset()针对每个游戏而改变的一些变量：

var dx, dy,        // pixel size of a single tetris block
    blocks,        // 2 dimensional array (nx*ny) representing tetris court - either empty block or occupied by a 'piece'
    actions,       // queue of user actions (inputs)
    playing,       // true|false - game is in progress
    dt,            // time since starting this game
    current,       // the current piece
    next,          // the next piece
    score,         // the current score
    rows,          // number of completed rows in the current game
    step;          // how long before current piece drops by 1 row
在更复杂的游戏中，这些都应该封装在一个或多个类中，但是为了使俄罗斯方块更简单，我们使用了简单的全局变量。但这并不意味着我们希望他们在整个代码中进行修改，因此为大多数可变游戏状态编写getter和setter方法仍然有意义。

function setScore(n)            { score = n; invalidateScore(); };
function addScore(n)            { score = score + n; };
function setRows(n)             { rows = n; step = Math.max(speed.min, speed.start - (speed.decrement*rows)); invalidateRows(); };
function addRows(n)             { setRows(rows + n); };
function getBlock(x,y)          { return (blocks && blocks[x] ? blocks[x][y] : null); };
function setBlock(x,y,type)     { blocks[x] = blocks[x] || []; blocks[x][y] = type; invalidate(); };
function setCurrentPiece(piece) { current = piece || randomPiece(); invalidate();     };
function setNextPiece(piece)    { next    = piece || randomPiece(); invalidateNext(); };
这还使我们能够采用一种受控的方式来知道何时更改了值，以便我们可以invalidate访问UI并知道该部分需要重新呈现。这将使我们能够优化渲染，并且只有在draw事物发生变化时才能进行优化。

游戏循环
核心游戏循环是乒乓球，突围 和蛇循环的简化版本。使用requestAnimationFrame （或polyfill），我们只需要update根据时间间隔和draw结果来确定游戏状态：
```
var last = now = timestamp();
function frame() {
  now = timestamp();
  update((now - last) / 1000.0);
  draw();
  last = now;
  requestAnimationFrame(frame, canvas);
}
frame(); // start the first frame
处理键盘输入
我们的键盘处理程序非常简单，我们实际上不做任何按键操作（除了开始/放弃游戏外）。取而代之的是，我们仅将操作记录在队列中以在处理过程中进行处理update。

function keydown(ev) {
  if (playing) {
    switch(ev.keyCode) {
      case KEY.LEFT:   actions.push(DIR.LEFT);  break;
      case KEY.RIGHT:  actions.push(DIR.RIGHT); break;
      case KEY.UP:     actions.push(DIR.UP);    break;
      case KEY.DOWN:   actions.push(DIR.DOWN);  break;
      case KEY.ESC:    lose();                  break;
    }
  }
  else if (ev.keyCode == KEY.SPACE) {
    play();
  }
};

```
玩游戏
定义了数据结构，设置了常量和变量，提供了获取器和设置器，启动了游戏循环并处理了键盘输入之后，我们现在可以看看实现俄罗斯方块游戏机制的逻辑：

该update循环由处理下一个用户操作组成，并且如果累积的时间大于某个变量（基于完成的行数），则将当前片段减少1行：
```
function update(idt) {
  if (playing) {
    handle(actions.shift());
    dt = dt + idt;
    if (dt > step) {
      dt = dt - step;
      drop();
    } 
  }
};
```
处理用户输入的操作包括向左或向右移动当前片段，旋转它或将其再下移1行：
```
function handle(action) {
  switch(action) {
    case DIR.LEFT:  move(DIR.LEFT);  break;
    case DIR.RIGHT: move(DIR.RIGHT); break;
    case DIR.UP:    rotate();        break;
    case DIR.DOWN:  drop();          break;
  }
};
```
move和rotate简单地改变当前片x，y或dir可变的，但是它们必须检查，以确保新的位置/方向未被占用：
```
function move(dir) {
  var x = current.x, y = current.y;
  switch(dir) {
    case DIR.RIGHT: x = x + 1; break;
    case DIR.LEFT:  x = x - 1; break;
    case DIR.DOWN:  y = y + 1; break;
  }
  if (unoccupied(current.type, x, y, current.dir)) {
    current.x = x;
    current.y = y;
    invalidate();
    return true;
  }
  else {
    return false;
  }
};

function rotate(dir) {
  var newdir = (current.dir == DIR.MAX ? DIR.MIN : current.dir + 1);
  if (unoccupied(current.type, current.x, current.y, newdir)) {
    current.dir = newdir;
    invalidate();
  }
};
```
该drop方法会将当前片段向下移动1行，但是如果那不可能，那么当前片段将尽可能地低，并将其分解为单独的块。在这一点上，我们增加了分数，检查所有已完成的行并设置新的曲目。如果新棋子也处于占用位置，则游戏结束：
```
function drop() {
  if (!move(DIR.DOWN)) {
    addScore(10);
    dropPiece();
    removeLines();
    setCurrentPiece(next);
    setNextPiece(randomPiece());
    if (occupied(current.type, current.x, current.y, current.dir)) {
      lose();
    }
  }
};

function dropPiece() {
  eachblock(current.type, current.x, current.y, current.dir, function(x, y) {
    setBlock(x, y, current.type);
  });
};
```
渲染图
渲染成为<canvas>API和DOM helper方法的一种相当简单的用法html：

  function html(id, html) { document.getElementById(id).innerHTML = html; }
由于俄罗斯方块以相当“矮胖”的方式移动，我们可以认识到我们不需要以60fps的速度在每一帧上重画所有内容。我们仅在元素更改时才可以重绘元素。对于此简单的实现，我们将UI分为4部分，并且仅当我们检测到以下更改时才重新渲染这些部分：

比分
完成的行数
下一块预览显示
游戏场
最后一部分，游戏场，是一个相当广泛的类别。从技术上讲，我们可以跟踪网格中的每个单独的块，然后仅重绘已更改的块，但这可能会过大。重绘整个网格仅需几毫秒即可完成，并且确保仅在发生更改时才进行绘制，这意味着我们每秒只能承受几次这种小碰撞。
```
var invalid = {};

function invalidate()         { invalid.court  = true; }
function invalidateNext()     { invalid.next   = true; }
function invalidateScore()    { invalid.score  = true; }
function invalidateRows()     { invalid.rows   = true; }

function draw() {
  ctx.save();
  ctx.lineWidth = 1;
  ctx.translate(0.5, 0.5); // for crisp 1px black lines
  drawCourt();
  drawNext();
  drawScore();
  drawRows();
  ctx.restore();
};

function drawCourt() {
  if (invalid.court) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    if (playing)
      drawPiece(ctx, current.type, current.x, current.y, current.dir);
    var x, y, block;
    for(y = 0 ; y < ny ; y++) {
      for (x = 0 ; x < nx ; x++) {
        if (block = getBlock(x,y))
          drawBlock(ctx, x, y, block.color);
      }
    }
    ctx.strokeRect(0, 0, nx*dx - 1, ny*dy - 1); // court boundary
    invalid.court = false;
  }
};

function drawNext() {
  if (invalid.next) {
    var padding = (nu - next.type.size) / 2; // half-arsed attempt at centering next piece display
    uctx.save();
    uctx.translate(0.5, 0.5);
    uctx.clearRect(0, 0, nu*dx, nu*dy);
    drawPiece(uctx, next.type, padding, padding, next.dir);
    uctx.strokeStyle = 'black';
    uctx.strokeRect(0, 0, nu*dx - 1, nu*dy - 1);
    uctx.restore();
    invalid.next = false;
  }
};

function drawScore() {
  if (invalid.score) {
    html('score', ("00000" + Math.floor(score)).slice(-5));
    invalid.score = false;
  }
};

function drawRows() {
  if (invalid.rows) {
    html('rows', rows);
    invalid.rows = false;
  }
};

function drawPiece(ctx, type, x, y, dir) {
  eachblock(type, x, y, dir, function(x, y) {
    drawBlock(ctx, x, y, type.color);
  });
};

function drawBlock(ctx, x, y, color) {
  ctx.fillStyle = color;
  ctx.fillRect(x*dx, y*dy, dx, dy);
  ctx.strokeRect(x*dx, y*dy, dx, dy)
};
```
改进空间
就像我说的，这只是俄罗斯方块游戏的原始机制。如果要完善此游戏，则需要添加以下内容：

菜单
等级
高分数
动画和效果
音乐和声音效果
触摸支持
玩家对玩家
玩家对战AI
（等等）

游戏在桌面浏览器上播放效果最佳。抱歉，没有移动支持。

Chrome，Firefox，IE9 +，Safari，Opera
