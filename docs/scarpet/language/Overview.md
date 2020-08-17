# `scarpet` 编程语言的基本组件

Scarpet 又名 Carpet 脚本，或 Scarpet 脚本，是一门旨在提供在 Minecrft 中编写自定义
程序、并与世界交互的能力的编程语言。

本规范分成两部分：这一部分与任何 Minecraft 功能无关，可以独立运行；而
CarpetExpression 则用于 Minecraft 专用程序和世界操作功能。


# 简介

<pre>
script run print('Hello World!')
</pre>

或一个过于复杂的例子：

<pre>
/script run
    block_check(x1, y1, z1, x2, y2, z2, block_to_check) ->
    (
        l(minx, maxx) = sort(l(x1, x2));
        l(miny, maxy) = sort(l(y1, y2));
        l(minz, maxz) = sort(l(z1, z2));
        '自然，需要计算区域大小';
        '因为在命令模式下不支持注释';
        xsize = maxx - minx + 1;
        ysize = maxy - miny + 1;
        zsize = maxz - minz + 1;
        total_count = 0;
        loop(xsize,
            xx = minx + _ ;
            loop(ysize,
                yy = miny + _ ;
                loop(zsize,
                    zz = minz + _ ;
                    if ( block(xx,yy,zz) == block_to_check,
                        total_count += ceil(rand(1))
                    )
                )
            )
        );
        total_count
    );
    check_area_around_closest_player_for_block(block_to_check) ->
    (
        closest_player = player();
        l(posx, posy, posz) = query(closest_player, 'pos');
        total_count = block_check( posx-8,1,posz-8, posx+8,17,posz+8, block_to_check);
        print('你周围有 '+total_count+' 个 '+block_to_check+' 方块')
    )
/script invoke check_area_around_closest_player_for_block 'diamond_ore'
</pre>

或简单地

<pre>
/script run print('你周围有 '+for(rect(x,9,z,8,8,8), _ == 'diamond_ore')+' 块钻石矿')
</pre>

检查一下都有什么更高级别的 `scarpet` 函数自然好处多多。

# 程序

程序与数学表达式是类似的，就像 `"2.4*sin(45)/(2-4)"` 或者 `"sin(y)>0 & max(z, 3)>3"` 这样。
写程序和写 `2+3` 没什么区别，只是长了很多。

## 基本语言组件

程序由常量（比如 `2`，`3.14`，`pi` 或 `'foo'`——这是个字符串）、操作符（比如 `+`，`/`，`->`）、你定义的变量
（比如 `foo` 或你自己定义的其他什么东西，`_x`、`_` 之类的）组成，还有一些内置的函数。
函数有函数名和参数，形式如 `f(a,b,c)`，`f` 是函数的名字，`a, b, c` 是参数，
这些参数可以是其他任何表达式。这些就是该语言的所有部分了！
听起来非常简单，不是吗？

## 代码流

就像其他那些比较正经的编程语言那样，`scarpet` 也需要括号，基本上是为了辨别事物的开始与结束。
那些使用更复杂的结构的编程语言，比如 Java，倾向于把所有括号都用上，
圆括号代表函数调用，花括号表示代码块，方形的表示访问列表，尖括号表示泛型，诸如此类……（译注：都是C++的错）
哦不对，不用诸如此类，键盘上一共就只有四种括号……

`scarpet` 就不一样了！运行的一切都以函数为基础（虽然不像 lisp 那样是函数式语言），
只需要圆括号就能表示一切了。因此程序员需要组织好代码
以提升可读性，因为增加括号其实不对程序的性能有任何影响，
因为在执行前会先编译。看看下面 `if()` 函数的使用实例吧：

<pre>
if(x&lt;y+6,set(x,8+y,z,'air');plop(x,top('surface',x,z),z,'birch'),sin(query(player(),'yaw'))&gt;0.5,plop(0,0,0,'boulder'),particle('fire',x,y,z))
</pre>

你可以写成这样

<pre>
if(   x&lt;y+6,
           set(x,8+y,z,'air');
           plop(x,top('surface',x,z),z,'birch'),
      sin(query(player(),'yaw'))>0.5,
           plop(0,0,0,'boulder'),
      particle('fire',x,y,z)
)
</pre>

或者这样

<pre>
if
(   x&lt;y+6,
    (
        set(x,8+y,z,'air');
        plop(x,top('surface',x,z),z,'birch')
    ),
    // else if
    sin(query(player(),'yaw'))>0.5,
    (
        plop(0,0,0,'boulder')
    ),
    // else
    particle('fire',x,y,z)
)
</pre>

代码风格并不重要。它通常取决于具体情况和子组件的复杂度。
无论你加了多少空格和额外的括号，代码都会被评估为相同的表达式，
运行情况也不会出现任何变化，因此确保你的程序可读性强、干净整洁
既不会引起性能变化，也可以帮助他人更好理解。

## 函数和作用域

用户可以以类似于 `fun(args....) -> expression` 的形式定义函数，它们会被编译、保存，以便将来执行、
后续的 /script 命令调用、添加到事件，等等。
也可以将函数分配给变量，作为参数传递，
用 `call('fun', args...)` 函数调用，但大多数情况下，
以`fun(args...)` 这样的形式直接用函数名来调用就足够了。
这意味着，定义好的函数会和世界一起保存下来，留待将来使用。
有两种类型的变量，全局 - 在代码各处意义相同，
名称以 'global_' 打头，除此之外的所有变量都是局部变量，
只在每个函数内保持意义。这意味着所有函数的参数传递都是
'传递值'，而不是'传递引用'。

## 外部变量

函数可以从外部作用域'借'变量用，只需要把它们添加到函数签名包裹的内置函数 `outer` 中。
它将指定的值添加到函数调用的栈中，所以其行为与 Java 捕获 λ 表达式完全一致。
但与 Java 不同的是，捕获的变量不一定非要是 final 的。（译注：final 是 Java 中的关键字，可简单理解为不可变的值）
Scarpet 会在函数定义时附加它们的新值，其后它们的变化就不计入在内了。
大多数值是简单的复制，但可变的值，比如映射和列表，可以在函数内保留其'状态'，（译注：这里的可变，指 mutable）
有一块内存空间，行为与对象差不多。查看 `outer(var)` 一节详细了解。

## Code delivery, line indicators

注意，该部分仅适用于将你的代码粘贴到命令方块中执行。
Scarpet 建议你把代码放到应用中（拓展名为 `.sc` 的文件，放到世界文件的 "/scripts" 文件夹内，
使用命令 `/script load [app_name]` 加载为 scarpet 应用）。
从文件中加载的 scarpet 应用只应包含代码，不需要以 "/script run" 开头。

以下代码可置于世界 `/scripts` 文件夹下 `foo.sc` 应用文件中：

<pre>
run_program() -> (
  loop( 10,
    // 循环十次
    // 可以在文件脚本内写注释
    // 因为文件可以有很多行 而一条命令不行
    foo = floor(rand(10));
    check_not_zero(foo);
    print(_+' - 随便来个数：'+foo);
    print('  倒数：'+  _/foo )
  )
);
check_not_zero(foo) -> (
  if (foo==0, foo = 1)
)
</pre>

之后我们就可以在游戏内这样调用：

<pre>
/script load foo
/script in foo invoke run_program
</pre>

不过，此代码也可以作为命令输入，或在命令方块中输入。

考虑到在聊天框中能输入的命令长度是有限的，你可以把程序粘贴到命令方块中，或者从世界文件中读取。
然而，粘贴到命令方块这一过程会删去一些空格、压掉所有换行，这样程序的可读性就变差了。
如果你粘贴的程序完美无缺、永不出错，那我实在佩服，但是大多数情况下，恐怕没这么一帆风顺。
无论是在编译时（代码首次被分析）还是在后续的执行过程中（可能不小心除以零），程序都有可能崩溃。
在这些情况下，你一定渴望得到一条有意义的报错看看到底在哪出了问题，但是这样的话，你需要告诉编译器
新的一行是从哪里开始的，因为命令方块会把无论多少行都变成一行。
于是，我们引入了 `$` 这么一个换行标识符。
这样的一个不怎么好的结果是，`$` 成了程序中的唯一非法字符，因为它本应标识新行的开始，
会被换行替换掉。据我所知，Minecraft 标识符内部并没有使用 `$`，
所以这应该不会阻碍你程序的能力。

考虑在命令方块中执行的如下命令：

<pre>
/script run
run_program() -> (
  loop( 10,
    foo = floor(rand(_));
    check_not_zero(foo);
    print(_+' - 随便来个数：'+foo);
    print('  倒数：'+  _/foo )
  )
);
check_not_zero(foo) -> (
   if (foo==0, foo = 1)
)
</pre>

其本意是检查随机生成的这个数是不是零以防止除以零，Lets say that the intention was to check if the bar is zero and prevent division by zero in print, but because 
但其实 `foo` 是作为值传递的，检查是否是 0 根本不影响原来变量的值。is passed as a variable, it never changes the original foo value. Because of the inevitable division 
如果很不幸地随机生成了零，我们会得到这样的报错：

<pre>
Your math is wrong, Incorrect number format for NaN at pos 98
run_program() -> ( loop( 10, foo = floor(rand(_)); check_not_zero(foo); print(_+' - 随便来个数：'+foo);
HERE>> print(' 倒数：'+ _/foo ) ));check_not_zero(foo) -> ( if (foo==0, foo = 1))
</pre>

正如所见，我们的问题出在运算结果不是个数字（NaN）上（无穷大，自然不是个数字），
然而把我们的程序粘贴到命令中，换行就全被吞了，
所以虽然能看出错误发生在哪里、可以追踪错误来源，
但错误发生的位置（98）找起来很痛苦，如果程序明显变长，就更痛苦了（译注：考虑一下发生在 114514 位的错误吧）。
为了解决这个问题，我们可以在脚本的每一行前面加上美元符号 `$`：

<pre>
/script run
$run_program() -> (
$  loop( 10,
$    foo = floor(rand(_));
$    check_not_zero(foo);
$    print(_+' - foo: '+foo);
$    print('  reciprocal: '+  _/foo )
$  )
$);
$check_not_zero(foo) -> (
$   if (foo==0, foo = 1)
$)
</pre>

那么报错信息会是这样的

<pre>
Your math is wrong, Incorrect number format for NaN at line 7, pos 2
  print(_+' - 随便来个数：'+foo);
   HERE>> print(' 倒数：'+ _/foo )
  )
</pre>

我们可以注意到，代码块变简洁了，我们还得到了关于行号和位置的信息，
这样定位潜在问题就更简单了。

那么，我们该怎么修改程序才能达成我们原先的意图呢？
简单，让 `foo` 随函数返回值变化就好了：

<pre>
foo = check_not_zero(foo);
...
check_not_zero(foo) -> if(foo == 0, 1, foo)
</pre>

……或者把它变成全局变量，这样连传参都省了

<pre>
global_foo = floor(rand(10));
check_foo_not_zero();
...
check_foo_not_zero() -> if(global_foo == 0, global_foo = 1)
</pre>

## Scarpet 预处理

有几个预处理操作应用于你的程序源码以进行清理并为执行做准备。
其中一些通过栈追踪和函数定义报告的操作会影响你的代码，
另外一些则非常浅显。
 - 跳过 `//` 注释（文件模式下）
 - 替换 `$` 为换行（命令模式下，修改已提交的代码）
 - 移除不作为二进制操作符 `;` 使用的多余分号，以允许对分号的宽松使用（译注：即分号仅作为下一条语句分隔符）
 - 替换 `{` 为 `m(`，`[` 为 `l(`，而 `]` 和 `}` 都会变成 `)`
 
目前没有进一步优化。

## 引用

LR1 解析器、标记器和几个内置函数都是基于 EvalEx 项目构建的。
EvalEx 是一个方便的 Java 表达式评估器，可以评估简单的
数学和布尔表达式。EvalEx 基于 MIT 许可证分发。
更多信息请见：[EvalEx GitHub repository](https://github.com/uklimaschewski/EvalEx)
