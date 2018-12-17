---
title: Lua脚本教程(1)--概念与基本语法
catalog: true
date: 2018-12-17 09:22:13
subtitle:
header-img: "Demo.png"
tags: Lua
---

# Lua教程

## 简介

### 设计目的
    其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。

### Lua特性
    轻量级: 它用标准C语言编写并以源代码形式开放，编译后仅仅一百余K，可以很方便的嵌入别的程序里。
    可扩展: Lua提供了非常易于使用的扩展接口和机制：由宿主语言(通常是C或C++)提供这些功能，Lua可以使用它们，就像是本来就内置的功能一样。
    其它特性:
    支持面向过程(procedure-oriented)编程和函数式编程(functional programming)；
    自动内存管理；只提供了一种通用类型的表（table），用它可以实现数组，哈希表，集合，对象；
    语言内置模式匹配；闭包(closure)；函数也可以看做一个值；提供多线程（协同进程，并非操作系统所支持的线程）支持；
    通过闭包和table可以很方便地支持面向对象编程所需要的一些关键机制，比如数据抽象，虚函数，继承和重载等。

## Lua安装
    http://www.runoob.com/lua/lua-environment.html

## Lua基本语法
   
### 编程模式
    1. 交互式编程 (shell)
        与python的shell使用方法一致，直接在终端下运行lua命令即可
    2. 脚本式编程
        创建xxxx.lua文件，然后使用lua xxxx.lua命令或者是luajit xxxx.lua文件即可进行运行
           
### 注释
    1.单行注释
        --
    2.多行注释
        --[[
        多行注释
        多行注释
        --]]   
        
### 标示符
    Lua 是一个区分大小写的编程语言
    Lua 标示符用于定义一个变量，函数获取其他用户定义的项。标示符以一个字母 A 到 Z 或 a 到 z 或下划线 _ 开头后加上0个或多个字母，下划线，数字(0到9)
    Lua 不允许使用特殊字符如 @, $, 和 % 来定义标示符
    最好不要使用下划线加大写字母的标示符，因为Lua的保留字也是这样的
    一般约定，以下划线开头连接一串大写字母的名字（比如 _VERSION）被保留用于 Lua 内部全局变量

### 关键词
    and    break    do    else
    elseif    end    false    for
    function    if    in    local
    nil    not    or    repeat
    return    then    true    until
    while

### 变量
    Lua 变量有三种类型：全局变量、局部变量、表中的域
    变量在使用前，必须在代码中进行声明，即创建该变量
    变量的默认值均为 nil。

    1.全局变量
        (1)在默认情况下，变量总是认为是全局的。
        (2)全局变量不需要声明，给一个变量赋值后即创建了这个全局变量，访问一个没有初始化的全局变量也不会出错，只不过得到的结果是：nil。
        (3)如果你想删除一个全局变量，只需要将变量赋值为nil。
        (4)Lua 中的变量全是全局变量，那怕是语句块或是函数里，除非用 local 显式声明为局部变量。
    2.局部变量
        局部变量的作用域为从声明位置开始到所在语句块结束。
    3.表中的域
    
#### 赋值
    1. 使用"="号进行赋值
    2. Lua可以对多个变量同时赋值，变量列表和值列表的各个元素用逗号分开，赋值语句右边的值会依次赋给左边的变量
        a,b=21,22
    3. 赋值语句Lua会先计算右边所有的值然后再执行赋值操作，所以我们可以这样进行交换变量的值
        x, y = y, x                     -- swap 'x' for 'y'
        a[i], a[j] = a[j], a[i]         -- swap 'a[i]' for 'a[j]'  
    4. 重点: 当变量个数和值的个数不一致时，Lua会一直以变量个数为基础采取以下策略
        a. 变量个数 > 值的个数             按变量个数补足nil
        b. 变量个数 < 值的个数             多余的值会被忽略
        
        a, b, c = 0, 1
        print(a,b,c)             --> 0   1   nil  
        
        a, b = a+1, b+1, b+2     -- value of b+2 is ignored
        print(a,b)               --> 1   2
        
        a, b, c = 0
        print(a,b,c)             --> 0   nil   nil
    5. 多值赋值经常用来交换变量，或将函数调用返回给变量
           a, b = f()
           f()返回两个值，第一个赋给a，第二个赋给b。
           应该尽可能的使用局部变量，有两个好处：
               1. 避免命名冲突。
               2. 访问局部变量的速度比全局变量更快。
### 数据类型
    Lua是动态类型语言，变量不要类型定义,只需要为变量赋值。 值可以存储在变量中，作为参数传递或结果返回。
    Lua中有8个基本类型分别为：nil、boolean、number、string、userdata、function、thread和table。

    数据类型	描述
    nil	            这个最简单，只有值nil属于该类，表示一个无效值（在条件表达式中相当于false）。
    boolean	        包含两个值：false和true。Lua认为false和nil为"假",其他任何值都是"真",包括0
    number	        表示双精度类型的实浮点数
    string	        字符串由一对双引号或单引号来表示
    function	    由 C 或 Lua 编写的函数
    userdata	    表示任意存储在变量中的C数据结构
    thread	        表示执行的独立线路，用于执行协同程序
    table	        Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字或者是字符串。在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。

#### 获取数据的类型
    重点: type(xxx) -- 返回的都是字符串形式的数据类型
          type(type(xxx)) == "string"
    
    print(type("Hello world"))      --> string      
    print(type(10.4*3))             --> number
    print(type(print))              --> function
    print(type(type))               --> function
    print(type(true))               --> boolean
    print(type(nil))                --> nil
    print(type(type(X)))            --> string

#### 获取当前变量的数据类型是否与设想的一致
    v = "Hello World."
    if type(v) == "string"
    then
        print("字符串类型")
    end

#### nil (空值)
    nil 类型表示一种没有任何有效值，它只有一个值 -- nil
    作用:对于全局变量和 table，nil 还有一个"删除"作用，给全局变量或者 table 表里的变量赋一个 nil 值，等同于把它们删掉
       
        例子:
            tab1 = { key1 = "val1", key2 = "val2", "val3" }
            for k, v in pairs(tab1) do
                print(k .. " - " .. v)
            end
             
            tab1.key1 = nil
            for k, v in pairs(tab1) do
                print(k .. " - " .. v)
            end

#### boolean(布尔值)
    boolean 类型只有两个可选值：true（真） 和 false（假），Lua 把 false 和 nil 看作是"假"，其他的都为"真",包括"0"也是真
    
#### number(数字)
    Lua 默认只有一种 number 类型 -- double（双精度）类型（默认类型可以修改 luaconf.h 里的定义）
    
#### string(字符串)
    1. 字符串由一对双引号或单引号来表示。
        string1 = "this is string1"
        string2 = 'this is string2'
    2. 两个方括号 "[[]]" 来表示"一块"字符串,用于多行文本 
        html = [[
        <html>
        <head></head>
        <body>
            <a href="http://www.runoob.com/">菜鸟教程</a>
        </body>
        </html>
        ]]
    3. 在对一个数字字符串上进行算术操作时，Lua 会尝试将这个数字字符串转成一个数字
        > print("2" + 6)
        8.0
        > print("2" + "6")
        8.0
        > print("2 + 6")
        2 + 6
        > print("-2e2" * "6")
        -1200.0
    4. 字符串连接符".."
         > print("a" .. 'b')
         ab
         > print(157 .. 428)
         157428
         > 
    5. 计算字符串长度
         > len = "www.runoob.com"
         > print(#len)
         14
         > print(#"www.runoob.com")
         14
 
#### table(表)   
    在 Lua 里，table 的创获取数据的类型建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表
    也可以在表里添加一些数据，直接初始化表:
    1. 初始化表
        -- 创建一个空的 table
        local tbl1 = {}         
        -- 直接初始表
        local tbl2 = {"apple", "pear", "orange", "grape"}
    
    2. -- 创建一个空的 table
       local tbl1 = {}
       -- 直接初始表
       local tbl2 = {"apple", "pear", "orange", "grape"}
    
    3. Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字或者是字符串
      table 不会固定长度大小，有新数据添加时 table 长度会自动增长，没初始的 table 都是 nil。
      重点: 不同于其他语言的数组把 0 作为数组的初始索引，在 Lua 里表的默认初始索引一般以 1 开始  
            local tbl = {"apple", "pear", "orange", "grape"}
            for key, val in pairs(tbl) do
               print("Key", key)
            end
    4. 索引,对 table 的索引使用方括号 []。Lua 也提供了"."操作
        > site = {}
        > site["key"] = "www.w3cschool.cc"
        > print(site["key"])
        www.w3cschool.cc
        > print(site.key)
        www.w3cschool.cc
#### function（函数）
    1.在 Lua 中，函数是被看作是"一类公民（First-Class Value）"
        function factorial1(n)
            if n == 0 then
                return 1
            else
                return n * factorial1(n - 1)
            end
        end
        print(factorial1(5))
        factorial2 = factorial1
        print(factorial2(5))
     
    2.function 可以以匿名函数（anonymous function）的方式通过参数传递
        -- function_test2.lua 脚本文件
        function testFun(tab,fun)
            for k ,v in pairs(tab) do
                print(fun(k,v));
            end
        end
        
        
        tab={key1="val1",key2="val2"};
        testFun(tab,
        function(key,val)--匿名函数
            return key.."="..val;
        end
        );    

#### thread(线程)
    在 Lua 里，最主要的线程是协同程序（coroutine）。
    协程跟线程（thread）差不多，拥有自己独立的栈、局部变量和指令指针，可以跟其他协同程序共享全局变量和其他大部分东西。
    线程跟协程的区别：线程可以同时多个运行，而协程任意时刻只能运行一个，并且处于运行状态的协程只有被挂起（suspend）时才会暂停。
    
#### userdata(自定义类型)     
    userdata 是一种用户自定义数据，用于表示一种由应用程序或 C/C++ 语言库所创建的类型，可以将任意 C/C++ 的任意数据类型的数据（通常是 struct 和 指针）存储到 Lua 变量中调用。
