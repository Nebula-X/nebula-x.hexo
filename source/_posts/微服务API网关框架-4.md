---
title: 微服务API网关框架(4)--运算符,循环,迭代器与流程控制
catalog: true
date: 2018-12-20 11:40:17
subtitle:
header-img: "Demo.png"
tags:
- 微服务API网关
catagories:
- 微服务API网关
---

## 运算符
    Lua提供了以下几种运算符类型:
        算术运算符
        关系运算符
        逻辑运算符
        其他运算符 (..,#)
    
### 算术运算符
    操作符         描述          实例
    +              加法          A + B 输出结果 30
    -              减法          A - B 输出结果 -10
    *              乘法          A * B 输出结果 200
    /              除法          B / A w输出结果 2
    %              取余          B % A 输出结果 0
    ^              乘幂          A^2 输出结果 100
    -              负号          -A 输出结果v -10

### 关系运算符
    操作符          描述	                                                                        实例
    ==              等于，检测两个值是否相等，相等返回 true，否则返回 false	                    (A == B) 为 false。
    ~=              不等于，检测两个值是否相等，相等返回 false，否则返回 true	                    (A ~= B) 为 true。
    >               大于，如果左边的值大于右边的值，返回 true，否则返回 false	                    (A > B) 为 false。
    <               小于，如果左边的值大于右边的值，返回 false，否则返回 true	                    (A < B) 为 true。
    >=              大于等于，如果左边的值大于等于右边的值，返回 true，否则返回 false	            (A >= B) 返回 false。
    <=              小于等于， 如果左边的值小于等于右边的值，返回 true，否则返回 false	        (A <= B) 返回 true。     

### 逻辑运算符
    操作符	        描述                                                                 实例
    and             逻辑与操作符。 若 A 为 false，则返回 A，否则返回 B。	                 (A and B) 为 false。
    or              逻辑或操作符。 若 A 为 true，则返回 A，否则返回 B。	                 (A or B) 为 true。
    not             逻辑非操作符。与逻辑运算结果相反，如果条件为 true，逻辑非为 false。    not(A and B) 为 true。

### 运算符优先级
    从高到低的顺序：
        ^
        not    - (unary)
        *      /
        +      -
        ..
        <      >      <=     >=     ~=     ==
        and
        or
        
## 循环
    1. while 循环	在条件为 true 时，让程序重复地执行某些语句。执行语句前会先检查条件是否为 true。
        while(condition)
        do
           statements
        end
        var 从 exp1 变化到 exp2，每次变化以 exp3 为步长递增 var，并执行一次 "执行体"。exp3 是可选的，如果不指定，默认为1。
        
        例子:
            for i=1,f(x) do
                print(i)
            end
             
            for i=10,1,-1 do
                print(i)
            end
    2. for 循环	重复执行指定语句，重复次数可在 for 语句中控制。
        (1)数值for循环
            for var=exp1,exp2,exp3 do  
                <执行体>  
            end  
        (2)泛型for循环   
            a = {"one", "two", "three"}
            for i, v in ipairs(a) do
                print(i, v)
            end 
    3. repeat...until	重复执行循环，直到 指定的条件为真时为止
        repeat
           statements
        until( condition )
        
        例子：
            --[ 变量定义 --]
            a = 10
            --[ 执行循环 --]
            repeat
               print("a的值为:", a)
               a = a + 1
            until( a > 15 )  
    4. 循环嵌套	可以在循环内嵌套一个或多个循环语句（while do ... end;for ... do ... end;repeat ... until;）
    5. 无限循环
        while( true )
        do
           print("循环将永远执行下去")
        end
    6. break 用于退出当前循环或语句
        例子：
            --[ 定义变量 --]
            a = 10
            --[ while 循环 --]
            while( a < 20 )
            do
               print("a 的值为:", a)
               a=a+1
               if( a > 15)
               then
                  --[ 使用 break 语句终止循环 --]
                  break
               end
            end    
            
## 迭代器
    1. 泛型 for 迭代器
        泛型 for 在自己内部保存迭代函数，实际上它保存三个值：迭代函数、状态常量、控制变量。
        泛型 for 迭代器提供了集合的 key/value 对，语法格式如下：
        for k, v in pairs(t) do
            print(k, v)
        end
        
        array = {"Lua", "Tutorial"}
        
        for key,value in ipairs(array) 
        do
           print(key, value)
        end
        
    2. 无状态的迭代器
        无状态的迭代器是指不保留任何状态的迭代器，因此在循环中我们可以利用无状态迭代器避免创建闭包花费额外的代价。
        每一次迭代，迭代函数都是用两个变量（状态常量和控制变量）的值作为参数被调用，一个无状态的迭代器只利用这两个值可以获取下一个元素。
        这种无状态迭代器的典型的简单的例子是ipairs，它遍历数组的每一个元素。
        以下实例我们使用了一个简单的函数来实现迭代器，实现 数字 n 的平方：
        function square(iteratorMaxCount,currentNumber)
           if currentNumber<iteratorMaxCount
           then
              currentNumber = currentNumber+1
           return currentNumber, currentNumber*currentNumber
           end
        end
        
        for i,n in square,3,0
        do
           print(i,n)
        end   
        
        简单例子:
            function iter (a, i)
                i = i + 1
                local v = a[i]
                if v then
                   return i, v
                end
            end
             
            function ipairs (a)
                return iter, a, 0
            end   
     
    3.多状态的迭代器
         很多情况下，迭代器需要保存多个状态信息而不是简单的状态常量和控制变量，最简单的方法是使用闭包，还有一种方法就是将所有的状态信息封装到table内，将table作为迭代器的状态常量，因为这种情况下可以将所有的信息存放在table内，所以迭代函数通常不需要第二个参数。
         array = {"Lua", "Tutorial"}
         
         function elementIterator (collection)
            local index = 0
            local count = #collection
            -- 闭包函数
            return function ()
               index = index + 1
               if index <= count
               then
                  --  返回迭代器的当前元素
                  return collection[index]
               end
            end
         end
         
         for element in elementIterator(array)
         do
            print(element)
         end
                      
## 流程控制
    1.if 语句	if 语句 由一个布尔表达式作为条件判断，其后紧跟其他语句组成。
        if(布尔表达式)
        then
           --[ 在布尔表达式为 true 时执行的语句 --]
        end
    2.if...else 语句	if 语句 可以与 else 语句搭配使用, 在 if 条件表达式为 false 时执行 else 语句代码。
        格式一：
            if(布尔表达式)
            then
               --[ 布尔表达式为 true 时执行该语句块 --]
            else
               --[ 布尔表达式为 false 时执行该语句块 --]
            end
        
        格式二：
            if( 布尔表达式 1)
            then
               --[ 在布尔表达式 1 为 true 时执行该语句块 --]
            
            elseif( 布尔表达式 2)
            then
               --[ 在布尔表达式 2 为 true 时执行该语句块 --]
            
            elseif( 布尔表达式 3)
            then
               --[ 在布尔表达式 3 为 true 时执行该语句块 --]
            else 
               --[ 如果以上布尔表达式都不为 true 则执行该语句块 --]
            end
    3.if 嵌套语句	你可以在if 或 else if中使用一个或多个 if 或 else if 语句 。
        if( 布尔表达式 1)
        then
           --[ 布尔表达式 1 为 true 时执行该语句块 --]
           if(布尔表达式 2)
           then
              --[ 布尔表达式 2 为 true 时执行该语句块 --]
           end
        end
