# groovy 简介
    groovy是一种运行在jvm上的动态语言，它吸收了一些其他语言的特性，在Java语言基础上增肌了许多新的功能；
    兼具脚本语言和静态语言的特性
    相比于Java而言，语法更易懂，更易上手，更易调试；
    无缝的集成了Java 的类和库
    编译后的.groovy也是以class的形式出现的

# groovy语法

+ groovy语法和Java语法类似（兼容Java 7之前的大部分特性，修改.java文件拓展名为.groovy也能够正常运行）
+ 除非另外指定，Groovy的所有内容都是public
+ Groovy允许定义简单的脚本，同时与需定义正规的class对象
+ Groovy在普通的Java对象上增加了一些独特的方法和快捷方式，是的它们更容易使用
+ Groovy闭包
+ Groovy是动态类型，可以使用def定义任何对象，类似js var，类型也可以动态变化

## 原始类型的转换
```Groovy
    int x = 3;
    println x.class // class java.lang.Integer
    double y = 3.14
    println y.class // class java.lang.Double
```
## 字符串
```Groovy
    def str1 ='单引号字符串，不会进行变量替换'
    def name = "hello"
    def str2 = "双（三）引号变量替换, $name "
    def str3 = """多行内容
    """
    def str4 = "1 add 2 equals = ${1+2}" //
    println str4 // "1 add 2 equals = 3", ${}中的运算

    // 拓展方法
    def result = name.padLeft(10)
    println result
    // 字符串截取
    println name[0..1] // he, 数组下标越界会报错


```

## 列表
```Groovy
    // 定义列表
    def list = ["Groovy" "Hello" "world"]
    // 添加元素的三种方法
    list.add("test")
    list << "jack"
    list[10] = "chen" //索引增加到10，未设置的为null

    // 集合 + -运算
    def number = [1,2,3,4]
    println number + 5 == [1,2,3,4,5] // 方法调用可以不加括号
    println number - [2,3] == [1,4]

    // 拓展的方法
    println number.join(",") == "1,2,3,4" 
```
## Map
```Groovy
    def hash = [name:"andy","age":45]
    println hash.class // class java.util.LinkedHashMap
    hash.gender = 'man'
    hash['height'] = 180
    println hash
```

## 循环语句
```Groovy
    // 类似js的遍历
    def names = ["a", "b", "c"]
    for (name in names) {
        println( name)
    }

    // 类似python
    for (i in 1..5) {
        println i
    }

    // 输出 1 2 3，downto相反
    1.upto(3){
        println it
    }
    // 1 3 5 7 step方法，不包括9
    1.step(9,2) {
        println it
    }
```

## 闭包

所有的闭包都有返回值，默认null
```Groovy
    def closure = {
        return "hello"
    }
    println closure.class //class Clouser$_run_closure1 像内部类
    println closure.call() //闭包调用

    def closure2 = { -> println "hello $it"} // {println "hello $it"}
    // it默认变量
    println closure2.call('test')

    c= {it} // c={item} 报错
    println(c('test')) // 输出test, it默认变量

    // 输出 0 1 2 3
    4.times {
        println it
    }

    // 输出 a b c d e f g
    def str = 'abcdefg'
    str.each {
        temp -> println temp // println it
    }

    def scriptClosure = {
        println "this:" + this;// 代表闭包定义处的类，找最近的class类，没有指向当前文件类
        println "owner:" + owner;// 代表闭包定义处的类或者对象
        println "delegate:" + delegate;// 代表任意对象，默认与owmer相同
    }

    // 全部输出当前文件类的名字
    scriptClosure.call()

    // 内部类的闭包
    class Person {
        def static classClosure = {
            println "this:" + this;
            println "owner:" + owner;
            println "delegate:" + delegate;
        }

        def static say(){
            def classClosure = {
                println "this:" + this;
                println "owner:" + owner;
                println "delegate:" + delegate;
            }
            classClosure.call()
        }
    }
    // 全部输出Person，找到最近的class类
    Person.classClosure.call()
    Person.say()

    //闭包中定义闭包
    def nestClosure = {
        def innerClosure = {
            println "this:" + this; // 代表闭包定义处的类
            println "owner:" + owner; // 代表闭包定义处的类或者对象
            println "delegate:" + delegate; // 代表任意对象，默认与owmer相同
        }

        innerClosure.call()
    }
    // println(nestClosure.class)
    // println nestClosure.thisObject
    // 第一行输当前文件对应的类
    // 第二三行输出内部类
    nestClosure.call()
```