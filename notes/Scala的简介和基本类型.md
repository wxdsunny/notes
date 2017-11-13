---
title: Scala的简介和基本类型
tags: 基本类型,控制结构,scala
grammar_cjkRuby: true
---


scala是spark的原生语言
## scala的基本类型和操作
### 1.强类型,弱类型语言

> - 强类型语言:
> 	- 定义对象或变量时,需要指定其归属类型,一旦一个变量类型确定,它所归属的类型不可再变
> - 弱类型语言:
> 	- 定义变量时不用指定变量类型,在程序运行中可以改变变量的归属类型
> - scala定义变量
> 	- var str = "abcd"
> 		-  这样写法不是没有指定str类型,而是没有显示的指定str类型,它隐式的从变量值中自动判断
> 	- 显示写法:
> 		- var str:String = "abc"

### 2.声明变量有两种修饰符

> - var : 变量可重新赋值
> - val : (常量)不可被重新赋值
> - 在编程过程中能使用val的地方不要使用var

### 3.基础数据类型

> - JAVA基础数据类型,它对应的变量,不是对象,不能通过"."运算符来访问对象的方法
> - scala对应的java的基础数据的类型,对应的变量是对象,可以通过"."运算符来来调用对象的方法

### 4.String类型的自变量

> - val s1 = "abcd"
> - val s2 = "ab\"cd"
> - val s3 = """ab".ecvf/.cd""" 
> - 字符串模版嵌套
> - println (s"name ： $name ， age ： $age") 
> - println (s"name ： Sname ， age ： $ {age) aa") 
> - println (s"""name ： $name   age ： $age over"""  )
>  - 前面三个分号,后面三个分号

### 5.基础数据类型之间的转换方法

> - 对象.to类型
> 	- 123.toDouble
> 	- "123".toInt
> 	- 454.343.toString等

### 6.scala中的运算符

> - scala中运算符不是语法,而是函数(对象)
> - a+b 等同于 a.+(b)
> 	- 前者是后者的简写
> 	- 当一个对象通过点调用其方法的时候,如果该方法只有一个参数,name点号可以省略,小括号可以省略,对象,方法,参数之间用空格隔开即可 	https://www.scala-lang.org/api/current/scala/Int.html
> 
> - "=="在scala等同于equal方法
>  val a = new String("abc")  val b = new
> String("abc")  a == b   a.equal(b)

### 7.标识符 符合java规范

> - 类标识符,驼峰式命名首字母大写
> - 变量 方法标识符,驼峰式命名,首字母小写
> - 包标识符,全小写,层级使用点分割
> - val在scala中虽然定义的是常量,但是,一般都用变量的规则来命名标识符

### 8.scala和java的注释和语句块

> - scala注释规则和java一样
> - 语句块:
> 	- java 中的语句块全部是过程,没有返回值,只有方法语句块中用return才能有返回值
> 		- 主要用来划分作用域
> 	- scala中大部分的语句块都是有返回值的,而且不需要return
> 		- scala中的语句块除了划分作用域之外还可以带返回值

## scala的控制结构
### 10.if...else..
#### 与java中的if else区别
> scala中的if else语法是有返回值的,因此可以在变量的赋值上使用if else 语法,另外scala中没有三目运算表达式

#### 代码:
``` java
object IfTest1 {
    //接受分数,判断给出分数是优良中差
    def main(args: Array[String]): Unit = {
      val score = args(0).toInt
      //类java用法
      if(score>90){
        println("优秀")
      }else if(score>80){
        println("良好")
      }else if(score>60){
        println("中等")
      }else{
        println("差")
      }
      //java三目运算符 String result = score>60"及格":"不及格"
      //scala的用法
      //val result = if(score>60) "及格" else "不及格"
      val result = if(score > 90){
        "优秀"
      }else if(score > 80){
        "良好"
      }else if(score >60){
        "中等"
      }else{
        "差"
      }
      println(result)
    }
}
```

### 11.while循环

>  语句块中没有返回值  
>  
>  代码:
``` java
object WhileTest {
  def main(args: Array[String]): Unit = {
    var times = args(0).toInt
    while(times > 0){
      println(s"第${times}次打印")
      times = times -1
    }
    var times2 = args(0).toInt
    do{
      println(s"doWhile第${times2}次打印")
      times2 -= 1
    }while(times2>0)
    //构建死循环
    while(true){
        //循环体,轮询
    }
  }
}
```

### 12.for循环

> - for也是scala中少数没有返回值的语句块之一
> - 但是scala中也提供了一种方式(yield)让其具有返回能力

代码:

``` java
object ForTest {
  def main(args: Array[String]): Unit = {
    val times = args(0).toInt
    //循环遍历集合对象
   for(i <- 1 to times){
      println(s"1print:${i}")
    }
    //循环守卫添加
    for(i <- 1 to times if i%2==0) println(s"2print:$i")
    for(i <- 1 to times if i%2==0 && i>5) println(s"3print:$i")
    for(i<- 1 to times){
      for(j<- 1 to times){
        println(s"i:$i,j:$j")
      }
    }
   //scala的嵌套循环
    for(i <- 1 to times;j <- 1 to times){
      println(s"i:$i,j:$j")
    }
      //嵌套限定条件
     for(i <- 1 to times;j <- 1 to times if i%2==1 && j%2==0){
       println(s"i:$i,j:$j")
     }
    //i;长,j:宽 打印面积大于25的组合
     for(i<- 1 to times;j<- 1 to times if i*j>25){
       println(s"长:$i,宽:$j,面积:${i*j}")
     }*/
     //对于条件复杂的for循环,可以把小括号写成大括号
     for{
       i <- 1 to times
       j <- 1 to times
       x = i * j
       if x >25
     } println(s"长:$i,宽:$j,面积:${x}")*/
    for(i <- 1 to 9){
     for(j <- 1 to i){
       print(s"$i * $j = ${i*j}\t")
       if(i==j){
         println()
       }
     }
    }
    for(i <- 1 to 9;j <- 1 to i){
      print(s"$i * $j = ${i*j}\t")
      if(i==j) println()
    }
    for(i <- 1 to 9;j <- 1 to i) print(s"$i * $j = ${i*j}${if(i==j) "\n" else "\t"}")

    //把1-10之间的偶数以一个集合对象返回
   /* List result = new Arraylist()
    for(int i){}*/
    //让for具备返回值的能力
    //返回1-10 之间的偶数
    //yield后面的语句块的返回值就是for yield的返回值
    //yield后面的语句块一定有返回值,即便表达式上没有返回值,它会以Unit的对象"()"来作为返回值
    val result = for(i <- 1 to times) yield{
      if(i%2==0) i
    }
    println(result)
    val result2 = for(i <- 1 to times if i%2==0) yield  i
    println(result2)
    //构建一个集合对象 1-10
    //对每一个元素都加上5 返回打印新的结果
    val result3 = for(i <- 1 to 10) yield i+5
    println(result3)
  }
}

```

### 13.Unit类型

> - java 里无返回值的方法是void
> - scala中没有void,它使用Unit类型来代替
> - Unit的实例就是"()"

![Util类的实例][1]


  [1]: https://www.github.com/wxdsunny/images/raw/master/1510582467295.jpg