---
title: 微服务API网关框架(5)--Lua 模块与包,元表(Metatable)，协程，文件I/O，错误处理,面向对象,数据库访问与时间操作
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
    
    
    从lua5.1开始，Lua 加入了标准的模块管理机制，Lua 的模块是由变量、函数等已知元素组成的 table，
    
    因此创建一个模块很简单，就是创建一个 table，然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。
    
    一）模块定义
    
    模块的文件名 和 模块定义引用名称要一致
    
    -- 文件名为 model.lua
    -- 定义一个名为 model 的模块
    model = {}
     
    -- 定义一个常量
    model.constant = "这是一个常量"
     
    -- 定义一个函数
    function model.func1()
        print("这是一个公有函数")
    end
     
    local function func2()
        print("这是一个私有函数！")
    end
     
    function model.func3()
        func2()
    end
     
    return model
    
    二）require 函数
    Lua提供了一个名为require的函数用来加载模块。要加载一个模块，只需要简单地调用就可以了。例如：
    require("<模块名>")  或者  require "<模块名>"
    执行 require 后会返回一个由模块常量或函数组成的 table，并且还会定义一个包含该 table 的全局变量。
    
    -- test_model.lua 文件
    -- model 模块为上文提到 model.lua
    
    require("model")
    print(model.constant)  
    model.func3()
    
    -------------------------
    另一种写法，给加载的模块定义一个别名变量，方便调用  
    local m = require("model")   
    print(m.constant)
    m.func3()
    
    以上代码执行结果为：
        这是一个常量
        这是一个私有函数！
    
    
    如：模块定义的model，为local修饰为局部变量，那只能采用local m = require("model") 引用
    
    三）require 加载机制
    
    我们使用require命令时，系统需要知道引入哪个路径下的model.lua文件。
    
    require 用于搜索 Lua 文件的路径是存放在全局变量 package.path 中，
    
    当 Lua 启动后，会以环境变量 LUA_PATH 的值来初始这个环境变量。
    如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。
    
    lua文件的路径存放在全局变量package.path中，默认的package.path的值为 print(package.path)
    ./?.lua;/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/?.lua;/usr/local/share/lua/5.1/?.lua;/usr/local/share/lua/5.1/?/init.lua;/usr/local/openresty/luajit/share/lua/5.1/?.lua;/usr/local/openresty/luajit/share/lua/5.1/?/init.lua
    
    我们运行require("model");相当于把model替换上面的?号，lua就会在那些目录下面寻找model.lua如果找不到就报错。
    所以我们就知道为什么会报错了。
    
    那我们如何解决，我这里介绍常用的解决方案，编辑环境变量LUA_PATH
    在当前用户根目录下打开 .profile 文件（没有则创建，打开 .bashrc 文件也可以），
    例如把 "~/lua/" 路径加入 LUA_PATH 环境变量里：
    
    #LUA_PATH
    export LUA_PATH="/usr/local/lua/?.lua;;"
    
    文件路径以 ";" 号分隔，最后的 2 个 ";;" 表示新加的路径后面加上原来的默认路径。
    
    接着，更新环境变量参数，使之立即生效。
    
    source ~/.profile
    
    这时假设 package.path 的值是：
    
    /usr/local/lua/?.lua;./?.lua;/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/?.lua;/usr/local/share/lua/5.1/?.lua;/usr/local/share/lua/5.1/?/init.lua;/usr/local/openresty/luajit/share/lua/5.1/?.lua;/usr/local/openresty/luajit/share/lua/5.1/?/init.lua
    
    那么调用 require("model") 时就会尝试打开以下文件目录去搜索目标。
    
## 元表(Metatable)
    http://www.runoob.com/lua/lua-metatables.html
    https://www.cnblogs.com/blueberryzzz/p/8947446.html
    
    举个例子，在 Lua table 中我们可以访问对应的key来得到value值，但是却无法对两个 table 进行操作。
    
    那如何计算两个table的相加操作a+b？
    
    local t1 = {1,2,3}
    local t2 = {4,5,6}
    
    local t3 = t1 + t2   ---->  {1,2,3,4,5,6}
    
    类似java的一些操作重载
    
    这种类似的需求，lua 提供了元表(Metatable)和元方法，允许我们改变table的行为，每个行为关联了对应的元方法。
    
    1）setmetatable(table,metatable): 对指定table设置元表(metatable)，
    如果元表(metatable)中存在__metatable键值，setmetatable会失败 。
    
    2）getmetatable(table): 返回对象的元表(metatable)。
    
    mytable = {}                          -- 普通表 
    mymetatable = {}                      -- 元表
    setmetatable(mytable,mymetatable)     -- 把 mymetatable 设为 mytable 的元表 
    
    等价于：
    mytable = setmetatable({},{})
    
    返回对象元表：
    getmetatable(mytable)                 -- 返回mymetatable
    
    
    元方法的命名都是以 __ 两个下划线开头。
    
    一）__index 元方法
    对表读取索引一个元方法
    
    这是 metatable 最常用的键。
    
    当你通过键来访问 table 的时候，如果这个键没有值，那么Lua就会寻找该table的metatable（假定有metatable）
    中的__index元方法。如果__index是一个表，Lua会在表格中查找相应的键。
    
    local t = {}              -- 普通表t为空 
    local other = {foo = 2}   -- 元表 中有foo值
    setmetatable(t,{__index = other})     -- 把 other 设为 t 的元表__index
    print(t.foo);  ---输出 2
    print(t.bar);  ---输出nil
    
    t.foo为什么会输出2，就是因为我们重写了__index索引的重载，lua在执行中如果t中没有foo，
    就会在他的元表中__index中去找，__index等于other，就输出2。
    
    ----------------
    
    如果我们把t设置一下foo的值为3，在看看结果
    local t = {foo = 3 }                  -- 普通表t为空 
    local other = {foo = 2}               -- 元表 中有foo值
    setmetatable(t,{__index = other})     -- 把 other 设为 t 的元表__index
    print(t.foo);  ---输出 3
    print(t.bar);  ---输出nil
    
    ---------------------------------------------
    
    如果__index赋值为一个函数的话，Lua就会调用那个函数，table和键会作为参数传递给函数。
    __index 元方法查看表中元素是否存在，如果不存在，返回结果为 nil；如果存在则由 __index 返回结果。
    
    local t = {key1 = "value1" }            
    local function metatable(mytable,key)
       if key == "key2" then
          return "metatablevalue"
       else
          return nil
       end
    end
    setmetatable(t,{__index = metatable})     
    print(t.key1); 
    print(t.key2);  
    print(t.key3);  
    
    分析：
    
    print(t.key1);  ---这个输出value1 ，是因为t表中有此key
    print(t.key2);  ---这个输出metatablevalue，是因为t表中没有此key，就会调用t的元表中的元方法__index，
                    ---这是__index是个函数，就会执行这个函数，传t表和key值这两个参数到此函数，
                    ---函数体中判断有此key2 就输出metatablevalue；
    
    print(t.key3);  ---这个输出nil，是因为t表没有，元方法__index函数中 对key3返回nil值
    
    --------------------------------------
    
    总结：lua对表索引取值的步骤
    
    Lua查找一个表元素时的规则，其实就是如下3个步骤:
    1.在表中查找，如果找到，返回该元素，找不到则继续步骤2
    2.判断该表是否有元表，如果没有元表，返回nil，有元表则继续步骤3。
    3.判断元表有没有__index元方法，如果__index方法为nil，则返回nil；
      如果__index方法是一个表，则重复1、2、3；
      如果__index方法是一个函数，则执行该函数，得到返回值。
    
    
    
    二）__newindex 元方法
    
    __newindex 元方法用来对表更新，__index则用来对表访问 。
    
    当你给表进行索引进行赋值，但此索引不存在；lua会查找是否有元表，有元表就会查找__newindex 元方法是否存在：
    如果存在则调用__newindex的值进行执行操作，但不会对原表进行赋值操作。
    
    以下实例演示了 __newindex 元方法的应用：
    
    mymetatable = {}
    mytable = setmetatable({key1 = "value1"}, { __newindex = mymetatable })
    
    print("mytable.key1=",mytable.key1)
    mytable.newkey = "新值2"
    
    print("mytable.newkey=",mytable.newkey)
    print("mymetatable.newkey=",mymetatable.newkey)
    
    mytable.key1 = "新值1"
    print("mytable.key1=",mytable.key1)
    print("mymetatable.key1=",mymetatable.key1)
    
    以上实例中表设置了元方法 __newindex
    在对新索引键（newkey）赋值时（mytable.newkey = "新值2"），会调用元方法，而对mytable原表不进行赋值。
    而对mytable已存在的索引键（key1），则会进行赋值，而不调用元方法 __newindex。
    
    ----------------------------
    
    如果我们要对原来的table进行赋值，那我们就可以用rawset；；__newindex函数会传三个参数，
    mytable = setmetatable({key1 = "value1"}, { 
        __newindex = function(t,k,v)    ---第一个参数为table，第二个参数为key，第三个参数为value
    		rawset(t,k,v);
    	end
    })
    mytable.key1 = "new value"
    mytable.key2 = 4
    print("mytable.key1=",mytable.key1)
    print("mytable.key2=",mytable.key2)
    
    key2原本是不再mytable表中的，通过元方法__newindex中函数使用了rawset，就可以对原table进行赋值。
    
    三）为表添加操作符“+”
    
    我们这里定义“+”这元方法，把它定义为两个table相连
    如 
    t1={1,2,3}  
    t2={4,5,6}
    t1 + t2 相加的结果，我们想得到的是 {1,2,3,4,5,6}
    那我们如何写元表？
    “+”对应的元方法为__add
    
    local function add(mytable,newtable)
    	local num = table.maxn(newtable)
    	for i = 1, num do
          table.insert(mytable,newtable[i])
      end
      return mytable
    end
    
    local t1 = {1,2,3}
    local t2 = {4,5,6}
    
    setmetatable(t1,{__add = add})
    
    t1 = t1 + t2
    
    for k,v in ipairs(t1) do
    	print("key=",k," value=",v)
    end
    
    这样我们就实现了两个table相加
    
    
    以下是我们的操作符对应关系
    模式                    描述
    __add                 对应的运算符 '+'.
    __sub                 对应的运算符 '-'.
    __mul                 对应的运算符 '*'.
    __div                 对应的运算符 '/'.
    __mod                 对应的运算符 '%'.
    __unm                 对应的运算符 '-'.
    __concat              对应的运算符 '..'.
    __eq                  对应的运算符 '=='.
    __lt                  对应的运算符 '<'.
    __le                  对应的运算符 '<='.
    
    
    四）__call元方法
    __call元方法在 Lua 调用一个值时调用。以下实例演示了计算两个表中所有值相加的和：
    
    类似的 t();类似函数调用
    
    local function call(mytable,newtable)
    	local sum = 0
    	local i
        for i = 1, table.maxn(mytable) do
            sum = sum + mytable[i]
        end
        for i = 1, table.maxn(newtable) do
            sum = sum + newtable[i]
        end
        return sum
    end
    local t1 = {1,2,3}
    local t2 = {4,5,6}
    setmetatable(t1,{__call = call})
    
    local sum = t1(t2)     
    print(sum)
    
    
    五）__tostring 元方法
    __tostring 元方法用于修改表的输出行为。以下实例我们自定义了表的输出内容，把表中所有的元素相加输出：
    local t1 = {1,2,3}
    
    setmetatable(t1,{
    	__tostring = function(mytable)
    		local sum = 0
    		for k, v in pairs(mytable) do
    			sum = sum + v
    		end
    		return "all value sum =" .. sum
    	
    	end
    })
    print(t1)    ----print方法会调用table的tostring元方法   
    
    到此我们的元表 和 元方法 就讲完了，这个是需要大家自己动手去测试体验的。要有领悟能力
    
    
    
    六）点号与冒号操作符的区别
    
    local str = "abcde"
    print("case 1:", str:sub(1, 2))
    print("case 2:", str.sub(str, 1, 2))
    
    执行结果
    case 1: ab
    case 2: ab
    
    冒号操作会带入一个 self 参数，用来代表 自己。而点号操作，只是 内容 的展开。
    在函数定义时，使用冒号将默认接收一个 self 参数，而使用点号则需要显式传入 self 参数。
    obj = { x = 20 }
    function obj:fun1()
        print(self.x)
    end
    等价于
    obj = { x = 20 }
    function obj.fun1(self)
        print(self.x)
    end
    注意：冒号的操作，只有当变量是类对象时才需要。


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
    面向对象的特征: 封装 继承 多态（lua没有多态）,抽象(这个属性个别语言有)
    
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
        
        
    
    面向对象编程（Object Oriented Programming，OOP）是一种非常流行的计算机编程架构。
    java，c++，.net等都支持面向对象
    
    面向对象特征
    1） 封装：指能够把一个实体的信息、功能、响应都装入一个单独的对象中的特性。
    2） 继承：继承的方法允许在不改动原程序的基础上对其进行扩充，这样使得原功能得以保存，
        而新功能也得以扩展。这有利于减少重复编码，提高软件的开发效率。
    3） 多态：同一操作作用于不同的对象，可以有不同的解释，产生不同的执行结果。在运行时，
        可以通过指向基类的指针，来调用实现派生类中的方法。
    4） 抽象：抽象(Abstraction)是简化复杂的现实问题的途径，它可以为具体问题找到最恰当的类定义，
        并且可以在最恰当的继承级别解释问题。
    
    一）Lua 中面向对象
    对象由属性和方法组成，lua中的面向对象是用table来描述对象的属性，function表示方法；
    LUA中的类可以通过table + function模拟出来。
    
    一个简单实例
    以下简单的类代表矩形类，包含了二个属性：length 和 width；getArea方法用获取面积大小
    
    新建rect.lua脚本
    
    local rect = {length = 0, width = 0}
    
    -- 派生类的方法 new
    function rect:new (length,width)
      local o = {
         --设定各个项的值
         length = length or 0,
         width = width or 0
      }
      setmetatable(o, {__index = self})
      return o
    end
    
    -- 派生类的方法 getArea
    function rect:getArea ()
      return self.length * self.width
    end
    
    return rect
    
    -----------------test.lua------------------------
    
    引用模块
    local rect = require("rect")
    
    创建对象
    创建对象是为类的实例分配内存的过程。每个类都有属于自己的内存并共享公共数据。
    
    local rect1 = rect:new(10,20)
    local rect2 = rect:new(5,6)
    
    访问属性  ----》 rect1.length
    访问成员函数  ----》rect1:getArea()
    
    print("长度1：",rect1.length);
    print("面积1：",rect1:getArea());  --- 点号 和 冒号 调用前面课程已经介绍
    print("长度2：",rect2.length);
    print("面积2：",rect2:getArea());
    
    
    二）Lua继承
    
    -------------------shape基类---------------------------
    
    local shape = {name = ""}
    
    -- 创建实体对象方法 new
    function shape:new (name)
      local o = {
        name = name or "shape"
      }
      setmetatable(o, {__index = self})
      return o
    end
    
    -- 获取周长的方法 getPerimeter
    function shape:getPerimeter ()
      print("getPerimeter in shape");
      return 0
    end
    
    -- 获取面积的方法 getArea
    function shape:getArea ()
      print("getArea in shape");
      return 0
    end
    
    function shape:getVum ()
      print("getVum in shape");
      return 0
    end
    
    return shape
    
    ----------------------triangle继承类----------------------------
    
    local shape = require("shape")
    
    local triangle = {}
    
    -- 派生类的方法 new
    function triangle:new (name,a1,a2,a3)
      local obj = shape:new(name);
      
      --1）当方法在子类中查询不到时，再去父类中去查找。
      local super_mt = getmetatable(obj);
      setmetatable(self, super_mt);
      
      --2）把父类的元表 赋值super对象
      obj.super = setmetatable({}, super_mt)
      
      --3）属性赋值
      obj.a1 = a1 or 0;
      obj.a2 = a2 or 0;
      obj.a3 = a3 or 0;
      
      setmetatable(obj, { __index = self })
      
      return obj;
    end
    
    -- 派生类的方法 getPerimeter
    function triangle:getPerimeter()
      print("getPerimeter in triangle");
      return (self.a1 + self.a2 + self.a3);
    end
    
    -- 派生类的方法 getHalfPerimeter
    function triangle:getHalfPerimeter()
      print("getHalfPerimeter in triangle");
      return (self.a1 + self.a2 + self.a3) / 2
    end
    
    return triangle
    
    ------------------test-----------------------
    
    local rect = require("rect")
    --local shape = require("shape")
    local triangle = require("triangle")
     
    local rect1 = rect:new(10,20)
    local rect2 = rect:new(5,6)
    
    --local shape1 = shape:new();
    
    local triangle1 = triangle:new("t1",1,2,3)
    local triangle2 = triangle:new("t2",6,7,8)
    
    print("长度1：",rect1.length);
    print("面积1：",rect1:getArea());
    print("===============");
    print("长度2：",rect2.length);
    print("面积2：",rect2:getArea());
    print("===============");
    --print("shape1 getPerimeter:",shape1:getPerimeter())
    print("===============");
    ----覆盖了shape的方法
    print("t1 getPerimeter:",triangle1:getPerimeter())
    print("t1 getHalfPerimeter:",triangle1:getHalfPerimeter())
    
    print("t2 getPerimeter:",triangle2:getPerimeter())
    print("t2 getHalfPerimeter:",triangle2:getHalfPerimeter())
    
    ---- 从shape继承的getVum方法
    print("t1 getVum:",triangle1:getVum())
    print("t2 getVum:",triangle2:getVum())
    print("===============");
    print("t2 super getPerimeter:",triangle2.super:getPerimeter())     

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

## 时间操作
    在 Lua 中，函数 time、date 和 difftime 提供了所有的日期和时间功能。
    在 OpenResty 的世界里，不推荐使用这里的标准时间函数，
    因为这些函数通常会引发不止一个昂贵的系统调用，同时无法为 LuaJIT JIT 编译，对性能造成较大影响。
    推荐使用 ngx_lua 模块提供的带缓存的时间接口，
    如 ngx.today, ngx.time, ngx.utctime, ngx.localtime, ngx.now, ngx.http_time，以及 ngx.cookie_time 等。
    
    一）os.time ([table])
    它会返回当前的时间和日期的时间戳（精确到秒），如赋值table，表示此table指定日期的时间戳
    
    字段名称 		取值范围
    year 			四位数字
    month 			1--12
    day 			1--31
    hour 			0--23
    min 			0--59
    sec 			0--59
    isdst 			boolean（true表示夏令时）
    
    对于 time 函数，如果参数为 table，那么 table 中必须含有 year、month、day 字段。
    其他字缺省时段默认为中午（12:00:00）。
    print(os.time())    
    a = { year = 2018, month = 1, day = 30, hour = 0, min = 0, sec = 0 }
    print(os.time(a))   
    
    时间戳的是以计算机最小时间和指定时间之间相差的秒数，计算机最小时间为1970-1-1 00:00:00（美国时区），
    针对中国时区就是1970-1-1 08:00:00
    a = { year = 1970, month = 1, day = 1, hour = 8, min = 0, sec = 1 }
    print(os.time(a)) 
    输出的就是1秒
    
    二）os.difftime (t2, t1)
    返回 t1 到 t2 的时间差，单位为秒。
    
    local day1 = { year = 2018, month = 1, day = 30 }
    local t1 = os.time(day1)
    local day2 = { year = 2018, month = 1, day = 31 }
    local t2 = os.time(day2)
    print(os.difftime(t2, t1))
    
    --->output：86400
    
    三）os.date ([format [, time]])
    把一个表示日期和时间的数值，转换成更高级的表现形式。
    格式字符 			含义
    %a 					一星期中天数的简写（例如：Wed）
    %A 					一星期中天数的全称（例如：Wednesday）
    %b 					月份的简写（例如：Sep）
    %B 					月份的全称（例如：September）
    %c 					日期和时间（例如：07/30/15 16:57:24）
    %d 					一个月中的第几天[01 ~ 31]
    %H 					24小时制中的小时数[00 ~ 23]
    %I 					12小时制中的小时数[01 ~ 12]
    %j 					一年中的第几天[001 ~ 366]
    %M 					分钟数[00 ~ 59]
    %m 					月份数[01 ~ 12]
    %p 					“上午（am）”或“下午（pm）”
    %S 					秒数[00 ~ 59]
    %w 					一星期中的第几天[0 ~ 6 = 星期天 ~ 星期六]
    %x 					日期（例如：07/30/15）
    %X 					时间（例如：16:57:24）
    %y 					两位数的年份[00 ~ 99]
    %Y 					完整的年份（例如：2015）
    %% 					字符'%'
    
    print(os.date("today is %A, in %B"))
    print(os.date("now is %x %X"))
    print(os.date("%Y-%m-%d %H:%M:%S"))
    
    -->output
    today is Thursday, in July
    now is 07/30/15 17:39:22
    2018-03-29 22:36:05
    
    ---------------------------
    
    t = os.date("*t", os.time());
    for i, v in pairs(t) do
          print(i, v);
    end
    
    yday    120  --一年中的第几天，一月一日为1
    month   4
    sec     9
    min     9
    hour    16
    day     30
    year    2018
    isdst   false  --是否夏令时
    wday    2   --一周第几天  星期日为1
