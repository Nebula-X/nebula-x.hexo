---
title: 微服务API网关框架(3)--Lua基本语法
catalog: true
date: 2018-12-20 10:47:23
subtitle:
header-img: "Demo.png"
tags:
- 微服务API网关
catagories:
- 微服务API网关
---

# Lua基本语法

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

##### 字符串用法拓展
    1. 常规的转义符号
        \a  响铃(BEL)
        \b  退格(BS) ，将当前位置移到前一列
        \f  换页(FF)，将当前位置移到下页开头
        \n  换行(LF) ，将当前位置移到下一行开头
        \r  回车(CR) ，将当前位置移到本行开头
        \t  水平制表(HT) （跳到下一个TAB位置）
        \v  垂直制表(VT)
        \\  代表一个反斜线字符''\'
        \'  代表一个单引号（撇号）字符
        \"  代表一个双引号字符
        \0  空字符(NULL)
        \ddd    1到3位八进制数所代表的任意字符
        \xhh    1到2位十六进制所代表的任意字符
    
    2. 字符串操作
        (1)string.upper(argument):字符串全部转为大写字母  
        (2)string.lower(argument):字符串全部转为小写字母
        (3)string.gsub(mainString,findString,replaceString,num):在字符串中替换,mainString为要替换的字符串， findString 为被替换的字符，replaceString 要替换的字符，num 替换次数（可以忽略，则全部替换）
            > string.gsub("aaaa","a","z",3);
            zzza    3
        (4)string.find (str, substr, [init, [end]]):在一个指定的目标字符串中搜索指定的内容(第三个参数为索引),返回其具体位置。不存在则返回 nil
            > string.find("Hello Lua user", "Lua", 1) 
            7    9
        (5)string.reverse(arg):字符串反转    
        (6)string.format(...):返回一个类似printf的格式化字符串
            > string.format("the value is:%d",4)
            the value is:4
        (7)string.char(arg) 和 string.byte(arg[,int]): char 将整型数字转成字符并连接， byte 转换字符为整数值(可以指定某个字符，默认第一个字符)。
            > string.char(97,98,99,100)
            abcd
            > string.byte("ABCD",4)
            68
            > string.byte("ABCD")
            65
        (8)string.len(arg):计算字符串长度。
        (9)string.rep(string, n):返回字符串string的n个拷贝
        (10)".." 符号: 链接两个字符串
        (11)string.gmatch(str, pattern):回一个迭代器函数，每一次调用这个函数，返回一个在字符串 str 找到的下一个符合 pattern 描述的子串。如果参数 pattern 描述的字符串没有找到，迭代函数返回nil。
            > for word in string.gmatch("Hello Lua user", "%a+") do print(word) end
            Hello
            Lua
            user
        (12)string.match(str, pattern, init):string.match()只寻找源字串str中的第一个配对. 参数init可选, 指定搜寻过程的起点, 默认为1。 
                                            在成功配对时, 函数将返回配对表达式中的所有捕获结果; 如果没有设置捕获标记, 则返回整个配对字符串. 当没有成功的配对时, 返回nil。
             > = string.match("I have 2 questions for you.", "%d+ %a+")
             2 questions
             > = string.format("%d, %q", string.match("I have 2 questions for you.", "(%d+) (%a+)"))
             2, "questions"  
    3.字符串格式化         
         Lua 提供了 string.format() 函数来生成具有特定格式的字符串, 函数的第一个参数是格式 , 之后是对应格式中每个代号的各种数据。
            %c - 接受一个数字, 并将其转化为ASCII码表中对应的字符
            %d, %i - 接受一个数字并将其转化为有符号的整数格式
            %o - 接受一个数字并将其转化为八进制数格式
            %u - 接受一个数字并将其转化为无符号整数格式
            %x - 接受一个数字并将其转化为十六进制数格式, 使用小写字母
            %X - 接受一个数字并将其转化为十六进制数格式, 使用大写字母
            %e - 接受一个数字并将其转化为科学记数法格式, 使用小写字母e
            %E - 接受一个数字并将其转化为科学记数法格式, 使用大写字母E
            %f - 接受一个数字并将其转化为浮点数格式
            %g(%G) - 接受一个数字并将其转化为%e(%E, 对应%G)及%f中较短的一种格式
            %q - 接受一个字符串并将其转化为可安全被Lua编译器读入的格式
            %s - 接受一个字符串并按照给定的参数格式化该字符串
         为进一步细化格式, 可以在%号后添加参数. 参数将以如下的顺序读入:
            (1) 符号: 一个+号表示其后的数字转义符将让正数显示正号. 默认情况下只有负数显示符号.
            (2) 占位符: 一个0, 在后面指定了字串宽度时占位用. 不填时的默认占位符是空格.
            (3) 对齐标识: 在指定了字串宽度时, 默认为右对齐, 增加-号可以改为左对齐.
            (4) 宽度数值
            (5) 小数位数/字串裁切: 在宽度数值后增加的小数部分n, 若后接f(浮点数转义符, 如%6.3f)则设定该浮点数的小数只保留n位, 若后接s(字符串转义符, 如%5.3s)则设定该字符串只显示前n位
         例子:
            string1 = "Lua"
            string2 = "Tutorial"
            number1 = 10
            number2 = 20
            -- 基本字符串格式化
            print(string.format("基本格式化 %s %s",string1,string2))
            -- 日期格式化
            date = 2; month = 1; year = 2014
            print(string.format("日期格式化 %02d/%02d/%03d", date, month, year))
            -- 十进制格式化
            print(string.format("%.4f",1/3))
    4. 字符与整数相互转换  
         -- 字符转换
         -- 转换第一个字符
         print(string.byte("Lua"))
         -- 转换第三个字符
         print(string.byte("Lua",3))
         -- 转换末尾第一个字符
         print(string.byte("Lua",-1))
         -- 第二个字符
         print(string.byte("Lua",2))
         -- 转换末尾第二个字符
         print(string.byte("Lua",-2))
         -- 整数 ASCII 码转换为字符
         print(string.char(97))   

##### 字符串--正则表达式
     Lua 中的匹配模式直接用常规的字符串来描述。 它用于模式匹配函数 string.find, string.gmatch, string.gsub, string.match。
     你还可以在模式串中使用字符类。
     字符类指可以匹配一个特定字符集合内任何字符的模式项。比如，字符类%d匹配任意数字。所以你可以使用模式串 '%d%d/%d%d/%d%d%d%d' 搜索 dd/mm/yyyy 格式的日期：
     下面的表列出了Lua支持的所有字符类：
     单个字符(除 ^$()%.[]*+-? 外): 与该字符自身配对
     .(点): 与任何字符配对
     %a: 与任何字母配对
     %c: 与任何控制符配对(例如\n)
     %d: 与任何数字配对
     %l: 与任何小写字母配对
     %p: 与任何标点(punctuation)配对
     %s: 与空白字符配对
     %u: 与任何大写字母配对
     %w: 与任何字母/数字配对
     %x: 与任何十六进制数配对
     %z: 与任何代表0的字符配对
     %x(此处x是非字母非数字字符): 与字符x配对. 主要用来处理表达式中有功能的字符(^$()%.[]*+-?)的配对问题, 例如%%与%配对
     [数个字符类]: 与任何[]中包含的字符类配对. 例如[%w_]与任何字母/数字, 或下划线符号(_)配对
     [^数个字符类]: 与任何不包含在[]中的字符类配对. 例如[^%s]与任何非空白字符配对        
     
     当上述的字符类用大写书写时, 表示与非此字符类的任何字符配对. 例如, %S表示与任何非空白字符配对.例如，'%A'非字母的字符:
     > print(string.gsub("hello, up-down!", "%A", "."))
     hello..up.down.    4
     数字4不是字符串结果的一部分，他是gsub返回的第二个结果，代表发生替换的次数。
     在模式匹配中有一些特殊字符，他们有特殊的意义，Lua中的特殊字符如下：
     ( ) . % + - * ? [ ^ $
     '%' 用作特殊字符的转义字符，因此 '%.' 匹配点；'%%' 匹配字符 '%'。转义字符 '%'不仅可以用来转义特殊字符，还可以用于所有的非字母的字符。
      
      模式条目可以是：
          单个字符类匹配该类别中任意单个字符；
          单个字符类跟一个 '*'， 将匹配零或多个该类的字符。 这个条目总是匹配尽可能长的串；
          单个字符类跟一个 '+'， 将匹配一或更多个该类的字符。 这个条目总是匹配尽可能长的串；
          单个字符类跟一个 '-'， 将匹配零或更多个该类的字符。 和 '*' 不同， 这个条目总是匹配尽可能短的串；
          单个字符类跟一个 '?'， 将匹配零或一个该类的字符。 只要有可能，它会匹配一个；
          %n， 这里的 n 可以从 1 到 9； 这个条目匹配一个等于 n 号捕获物（后面有描述）的子串。
          %bxy， 这里的 x 和 y 是两个明确的字符； 这个条目匹配以 x 开始 y 结束， 且其中 x 和 y 保持 平衡 的字符串。 意思是，如果从左到右读这个字符串，对每次读到一个 x 就 +1 ，读到一个 y 就 -1， 最终结束处的那个 y 是第一个记数到 0 的 y。 举个例子，条目 %b() 可以匹配到括号平衡的表达式。
          %f[set]， 指 边境模式； 这个条目会匹配到一个位于 set 内某个字符之前的一个空串， 且这个位置的前一个字符不属于 set 。 集合 set 的含义如前面所述。 匹配出的那个空串之开始和结束点的计算就看成该处有个字符 '\0' 一样。
      模式：
        模式 指一个模式条目的序列。 在模式最前面加上符号 '^' 将锚定从字符串的开始处做匹配。 在模式最后面加上符号 '$' 将使匹配过程锚定到字符串的结尾。 如果 '^' 和 '$' 出现在其它位置，它们均没有特殊含义，只表示自身。
      捕获：
        模式可以在内部用小括号括起一个子模式； 这些子模式被称为 捕获物。 当匹配成功时，由 捕获物 匹配到的字符串中的子串被保存起来用于未来的用途。 捕获物以它们左括号的次序来编号。 例如，对于模式 "(a*(.)%w(%s*))" ， 字符串中匹配到 "a*(.)%w(%s*)" 的部分保存在第一个捕获物中 （因此是编号 1 ）； 由 "." 匹配到的字符是 2 号捕获物， 匹配到 "%s*" 的那部分是 3 号。
      作为一个特例，空的捕获 () 将捕获到当前字符串的位置（它是一个数字）。 例如，如果将模式 "()aa()" 作用到字符串 "flaaap" 上，将产生两个捕获物： 3 和 5 。
            
#### table(表)   
    在 Lua 里，table 的创获取数据的类型建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表
    也可以在表里添加一些数据，直接初始化表:
    1. 初始化表与移除引用
        -- 创建一个空的 table
        local tbl1 = {}         
        -- 直接初始表
        local tbl2 = {"apple", "pear", "orange", "grape"}
        -- 初始化表
        mytable = {} 
        -- 指定值
        mytable[1]= "Lua"
        -- 移除引用
        mytable = nil
        -- lua 垃圾回收会释放内存
    
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

##### table -- 操作
    1.table.concat (table [, sep [, start [, end]]]):concat是concatenate(连锁, 连接)的缩写. table.concat()函数列出参数中指定table的数组部分从start位置到end位置的所有元素, 元素间以指定的分隔符(sep)隔开。
    2.table.insert (table, [pos,] value):在table的数组部分指定位置(pos)插入值为value的一个元素. pos参数可选, 默认为数组部分末尾.
    3.table.remove (table [, pos]):返回table数组部分位于pos位置的元素. 其后的元素会被前移. pos参数可选, 默认为table长度, 即从最后一个元素删起。
    4.table.sort (table [, comp]):对给定的table进行升序排序

##### table -- table间连接,table中的插入,移除,排序
    使用 concat() 方法来连接两个 table
        fruits = {"banana","orange","apple"}
        -- 返回 table 连接后的字符串
        print("连接后的字符串 ",table.concat(fruits)
        -- 指定连接字符
        print("连接后的字符串 ",table.concat(fruits,", "))
        -- 指定索引来连接 table
        print("连接后的字符串 ",table.concat(fruits,", ", 2,3))
        
        fruits = {"banana","orange","apple"}
        -- 在末尾插入
        table.insert(fruits,"mango")
        print("索引为 4 的元素为 ",fruits[4])
        
        -- 在索引为 2 的键处插入
        table.insert(fruits,2,"grapes")
        print("索引为 2 的元素为 ",fruits[2])
        
        print("最后一个元素为 ",fruits[5])
        table.remove(fruits)
        print("移除后最后一个元素为 ",fruits[5])    
        
        --排序
        fruits = {"banana","orange","apple","grapes"}
        print("排序前")
        for k,v in ipairs(fruits) do
            print(k,v)
        end
        
        table.sort(fruits)
        print("排序后")
        for k,v in ipairs(fruits) do
            print(k,v)
        end

##### table -- 数组
    1. 一维数组
        array = {"Lua", "Tutorial"}
        
        for i= 0, 2 do
           print(array[i])
        end
    2. 多维数组
         -- 初始化数组
         array = {}
         for i=1,3 do
            array[i] = {}
               for j=1,3 do
                  array[i][j] = i*j
               end
         end
         
         -- 访问数组
         for i=1,3 do
            for j=1,3 do
               print(array[i][j])
            end
         end   
        
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
    3. 多返回值
        Lua函数中，在return后列出要返回的值的列表即可返回多值
            function maximum (a)
                local mi = 1             -- 最大值索引
                local m = a[mi]          -- 最大值
                for i,val in ipairs(a) do
                   if val > m then
                       mi = i
                       m = val
                   end
                end
                return m, mi
            end
            
            print(maximum({8,10,23,12,5}))     

##### 函数定义
    optional_function_scope function function_name( argument1, argument2, argument3..., argumentn)
        function_body
        return result_params_comma_separated
    end
    
    optional_function_scope: 该参数是可选的制定函数是全局函数还是局部函数，未设置该参数默认为全局函数，如果你需要设置函数为局部函数需要使用关键字 local。
    function_name: 指定函数名称。
    argument1, argument2, argument3..., argumentn: 函数参数，多个参数以逗号隔开，函数也可以不带参数。
    function_body: 函数体，函数中需要执行的代码语句块。
    result_params_comma_separated: 函数返回值，Lua语言函数可以返回多个值，每个值以逗号隔开。
    
##### 函数的可变参数
       1. 在函数参数列表中使用三点 ... 表示函数有可变的参数
            function add(...)  
            local s = 0  
              for i, v in ipairs{...} do   --> {...} 表示一个由所有变长参数构成的数组  
                s = s + v  
              end  
              return s  
            end  
            print(add(3,4,5,6,7))  --->25
       2. select("#",...) 来获取可变参数的数量
                function average(...)
                   result = 0
                   local arg={...}
                   for i,v in ipairs(arg) do
                      result = result + v
                   end
                   print("总共传入 " .. select("#",...) .. " 个数")
                   return result/select("#",...)
                end
                
                print("平均值为",average(10,5,3,4,5,6))
            
            在遍历变长参数的时候只需要使用 {…}，然而变长参数可能会包含一些 nil，那么就可以用 select 函数来访问变长参数了：select('#', …) 或者 select(n, …)
            select('#', …) 返回可变参数的长度
            select(n, …) 用于访问 n 到 select('#',…) 的参数
            调用select时，必须传入一个固定实参selector(选择开关)和一系列变长参数。如果selector为数字n,那么select返回它的第n个可变实参，否则只能为字符串"#",这样select会返回变长参数的总数
                do  
                    function foo(...)  
                        for i = 1, select('#', ...) do  -->获取参数总数
                            local arg = select(i, ...); -->读取参数
                            print("arg", arg);  
                        end  
                    end  
                  
                    foo(1, 2, 3, 4);  
                end
        
#### thread(线程)
    在 Lua 里，最主要的线程是协同程序（coroutine）。
    协程跟线程（thread）差不多，拥有自己独立的栈、局部变量和指令指针，可以跟其他协同程序共享全局变量和其他大部分东西。
    线程跟协程的区别：线程可以同时多个运行，而协程任意时刻只能运行一个，并且处于运行状态的协程只有被挂起（suspend）时才会暂停。
    
#### userdata(自定义类型)     
    userdata 是一种用户自定义数据，用于表示一种由应用程序或 C/C++ 语言库所创建的类型，可以将任意 C/C++ 的任意数据类型的数据（通常是 struct 和 指针）存储到 Lua 变量中调用。
