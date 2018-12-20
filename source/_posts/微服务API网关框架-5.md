---
title: 微服务API网关框架(5)--Lua 模块与包,元表(Metatable)，协程，文件I/O，错误处理,面向对象与数据库访问
catalog: true
date: 2018-12-20 11:40:23
subtitle:
header-img: "Demo.png"
tags:
- 微服务API网关
catagories:
- 微服务API网关
---

## 模块与包
    Lua 加入了标准的模块管理机制，可以把一些公用的代码放在一个文件里，以 API 接口的形式在其他地方调用，有利于代码的重用和降低代码耦合度。
    Lua 的模块是由变量、函数等已知元素组成的 table，因此创建一个模块很简单，就是创建一个 table，然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。
    demo:
       -- 文件名为 module.lua
       -- 定义一个名为 module 的模块
       module = {}
        
       -- 定义一个常量
       module.constant = "这是一个常量"
        
       -- 定义一个函数
       function module.func1()
           io.write("这是一个公有函数！\n")
       end
        
       local function func2()
           print("这是一个私有函数！")
       end
        
       function module.func3()
           func2()
       end
        
       return module
    
    模块引用
        require 函数
        Lua提供了一个名为require的函数用来加载模块。要加载一个模块，只需要简单地调用就可以了。
        
        require("<模块名>")
        或者
        require "<模块名>"
        
        -- test_module.lua 文件
        -- module 模块为上文提到到 module.lua
        require("module")
         
        print(module.constant)
         
        module.func3()
        
    加载机制
        加载模块路径
        对于自定义的模块，模块文件不是放在哪个文件目录都行，函数 require 有它自己的文件路径加载策略，它会尝试从 Lua 文件或 C 程序库中加载模块。
        require 用于搜索 Lua 文件的路径是存放在全局变量 package.path 中，当 Lua 启动后，会以环境变量 LUA_PATH 的值来初始这个环境变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。
        当然，如果没有 LUA_PATH 这个环境变量，也可以自定义设置，在当前用户根目录下打开 .profile 文件（没有则创建，打开 .bashrc 文件也可以），例如把 "~/lua/" 路径加入 LUA_PATH 环境变量里：
            #LUA_PATH
            export LUA_PATH="~/lua/?.lua;;"   
        文件路径以 ";" 号分隔，最后的 2 个 ";;" 表示新加的路径后面加上原来的默认路径。

    C语言包（非重点）-- 参考：http://www.runoob.com/lua/lua-modules-packages.html
       Lua和C是很容易结合的，使用C为Lua写包。 
       与Lua中写包不同，C包在使用以前必须首先加载并连接，在大多数系统中最容易的实现方式是通过动态连接库机制。
       Lua在一个叫loadlib的函数内提供了所有的动态连接的功能。这个函数有两个参数:库的绝对路径和初始化函数。
    
## 元表(Metatable)
    http://www.runoob.com/lua/lua-metatables.html
    

## 协程
    http://www.runoob.com/lua/lua-coroutine.html

## 文件I/O
    http://www.runoob.com/lua/lua-file-io.html

## 错误处理
    错误种类
        1. 语法错误
            语法错误通常是由于对程序的组件（如运算符、表达式）使用不当引起的
        2. 运行错误
            运行错误是程序可以正常执行，但是会输出报错信息

    错误处理
       使用两个函数：assert 和 error 来处理错误
       assert函数:
           local function add(a,b)
              assert(type(a) == "number", "a 不是一个数字")
              assert(type(b) == "number", "b 不是一个数字")
              return a+b
           end
           add(10)
           
           执行以上程序会出现如下错误：
           lua: test.lua:3: b 不是一个数字
           stack traceback:
               [C]: in function 'assert'
               test.lua:3: in local 'add'
               test.lua:6: in main chunk
               [C]: in ?         
        
       现象:实例中assert首先检查第一个参数，若没问题，assert不做任何事情；否则，assert以第二个参数作为错误信息抛出     

       error函数:
         格式: error (message [, level])
            功能：终止正在执行的函数，并返回message的内容作为错误信息(error函数永远都不会返回)
            通常情况下，error会附加一些错误位置的信息到message头部。
            Level参数指示获得错误的位置:
            Level=1[默认]：为调用error位置(文件+行号)
            Level=2：指出哪个调用error的函数的函数
            Level=0:不添加错误位置信息      

    pcall 和 xpcall、debug(非重点内容)
      Lua中处理错误，可以使用函数pcall（protected call）来包装需要执行的代码。
      pcall接收一个函数和要传递给后者的参数，并执行，执行结果：有错误、无错误；返回值true或者或false, errorinfo
          if pcall(function_name,...) then
          -- 没有错误
          else
          -- 一些错误
          end
        简单例子:
            > =pcall(function(i) print(i) end, 33)
            33
            true
            > =pcall(function(i) print(i) error('error..') end, 33)
            33
            false        stdin:1: error..        
        
        pcall以一种"保护模式"来调用第一个参数，因此pcall可以捕获函数执行中的任何错误。
        通常在错误发生时，希望落得更多的调试信息，而不只是发生错误的位置。但pcall返回时，它已经销毁了调用桟的部分内容。
        Lua提供了xpcall函数，xpcall接收第二个参数——一个错误处理函数，当错误发生时，Lua会在调用桟展开（unwind）前调用错误处理函数，于是就可以在这个函数中使用debug库来获取关于错误的额外信息了。
        debug库提供了两个通用的错误处理函数，返回的调试内容会多一点:
        debug.debug：提供一个Lua提示符，让用户来检查错误的原因
        debug.traceback：根据调用桟来构建一个扩展的错误消息

## 面向对象
    面向对象的特征: 封装 继承 多态（lua没有多态）
    
    封装
        Lua中的表不仅在某种意义上是一种对象。像对象一样，表也有状态（成员变量）；也有与对象的值独立的本性，特别是拥有两个不同值的对象（table）代表两个不同的对象；一个对象在不同的时候也可以有不同的值，但他始终是一个对象；与对象类似，表的生命周期与其由什么创建、在哪创建没有关系。对象有他们的成员函数，表也有：
           Account = {balance = 0}
           function Account.withdraw (v)
               Account.balance = Account.balance - v
           end 
           
        简单完整实例
           -- Meta class
           Shape = {area = 0}
           
           -- 基础类方法 new -- 创建对象
           function Shape:new (o,side)
             o = o or {}
             setmetatable(o, self)
             self.__index = self
             side = side or 0
             self.area = side*side;
             return o
           end
           
           -- 基础类方法 printArea
           function Shape:printArea ()
             print("面积为 ",self.area) -- 访问属性
           end
           
           -- 创建对象
           myshape = Shape:new(nil,10)
           
           myshape:printArea()   -- 访问成员函数
    
    继承
        继承是指一个对象直接使用另一对象的属性和方法
             -- Meta class
            Shape = {area = 0}
            -- 基础类方法 new
            function Shape:new (o,side)
              o = o or {}
              setmetatable(o, self)
              self.__index = self
              side = side or 0
              self.area = side*side;
              return o
            end
            -- 基础类方法 printArea
            function Shape:printArea ()
              print("面积为 ",self.area)
            end
            
            -- 创建对象
            myshape = Shape:new(nil,10)
            myshape:printArea()
            
            Square = Shape:new()
            -- 派生类方法 new
            function Square:new (o,side)
              o = o or Shape:new(o,side)
              setmetatable(o, self)
              self.__index = self
              return o
            end
            
            -- 派生类方法 printArea
            function Square:printArea ()
              print("正方形面积为 ",self.area)
            end
            
            -- 创建对象
            mysquare = Square:new(nil,10)
            mysquare:printArea()
            
            Rectangle = Shape:new()
            -- 派生类方法 new
            function Rectangle:new (o,length,breadth)
              o = o or Shape:new(o)
              setmetatable(o, self)
              self.__index = self
              self.area = length * breadth
              return o
            end
            
            -- 派生类方法 printArea
            function Rectangle:printArea ()
              print("矩形面积为 ",self.area)
            end
            
            -- 创建对象
            myrectangle = Rectangle:new(nil,10,20)
            myrectangle:printArea()
            
    重写
        -- 派生类方法 printArea
        function Square:printArea ()
          print("正方形面积 ",self.area)
        end        

## 数据库访问
    Lua 连接MySql 数据库
    
    require "luasql.mysql"
    
    --创建环境对象
    env = luasql.mysql()
    
    --连接数据库
    conn = env:connect("数据库名","用户名","密码","IP地址",端口)
    
    --设置数据库的编码格式
    conn:execute"SET NAMES UTF8"
    
    --执行数据库操作
    cur = conn:execute("select * from role")
    
    row = cur:fetch({},"a")
    
    --文件对象的创建
    file = io.open("role.txt","w+");
    
    while row do
        var = string.format("%d %s\n", row.id, row.name)
    
        print(var)
    
        file:write(var)
    
        row = cur:fetch(row,"a")
    end
    
    
    file:close()  --关闭文件对象
    conn:close()  --关闭数据库连接
    env:close()   --关闭数据库环境    
