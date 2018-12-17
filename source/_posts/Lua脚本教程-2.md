---
title: Lua脚本教程(2)--运算符,循环与流程控制
catalog: true
date: 2018-12-17 10:42:43
subtitle:
header-img: "Demo.png"
tags:
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
