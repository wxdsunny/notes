---
title: scala中类和变量定义和集合类型
tags: val,def,lazy,set,map
grammar_cjkRuby: true
---

## val def lazy 定义变量的区别
- val  类型的变量,在声明时就会把右边表达式的结果计算并赋值给val变量,一旦赋值,右边的表达式就不再计算
- def 类型的变量,在声明赋值时,右边的表达式是不会马上计算结果的
	- 在def变量每次被调用的时候,等号右边表达式都会被重新计算一次
- lazy定义变量,声明赋值时,等号右边的表达式不会马上计算结果
	- 在lazy对象被第一次调用的时候,等号右边表达式会被计算一次,并赋值给lazy对象
	- 后续的lazy对象的再次调用,右边的表达式将不会被重新计算

代码:

``` java
object DefTest {
  def sumInt(x:Int,y:Int) = {
    println(s"执行sumint方法")
    x+y
  }
  def main(args: Array[String]): Unit = {
    //val 类型的变量,在声明时就会把右边表达式的结果计算并赋值给val变量,一旦赋值,右边的表达式就不再计算
    val v = sumInt(3,5)
    println(s"打印val对象v:${v}")

    //def 类型的变量,在声明赋值时,右边的表达式是不会马上计算结果的
    //在def变量每次被调用的时候,等号右边表达式都会被重新计算一次
    def d = sumInt(3,5)
    println(s"变量d已经被定义赋值")
    println(s"变量d的值:$d")
    println(s"第二次打印d的值:$d")

    //lazy定义变量,声明赋值时,等号右边的表达式不会马上计算结果
    //在lazy对象被第一次调用的时候,等号右边表达式会被计算一次,并赋值给lazy对象
    //后续的lazy对象的再次调用,右边的表达式将不会被重新计算
    lazy val l = sumInt(3,5)
    println("变量l已经被定义赋值过")
    println(s"变量l的值是:${l}")
    println(s"第二次变量l的值是:${l}")
  }
}
```

## Set

## Map

## scala类的定义

scala的类的定义也是使用class关键字后面跟类的名字
- java类体内:
	- 属性
		- 静态属性
		- 非静态属性
	- 方法
		- 静态方法
		- 非静态方法
	- 静态代码块
- scala 类体内:
	- 属性
		- 非静态属性
	- 方法
		- 非静态方法
	- 非静态代码块
- scala中没有static关键词 
- scala中用Object(单例类)来承担静态的成员定义
- 在Class里面定义的属性和方法必须要通过实例化的对象才能调用
- 在Object


  [1]: https://www.github.com/wxdsunny/images/raw/master/1510793446419.jpg