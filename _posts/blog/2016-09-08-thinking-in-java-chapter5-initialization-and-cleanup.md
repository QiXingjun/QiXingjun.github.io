---
layout: post
title: 初始化和清理
categories: JAVA编程思想（第四版）
description: JAVA编程思想（第四版）读书笔记。
keywords: Java 
---

## 1. 用构造器确保初始化

在Java中，通过提供构造器，类的设计者可确保每个对象都会得到初始化。构造器采用和类相同的名字，所以“每个方法首字母小写”的编码风格并不适用于
构造器。构造器是一种特殊类型的方法，它没有返回值。

<pre class=”brush:java;gutter:true;”>
class Rock{
  //这是一个默认构造器
  Rock(){
    System.out.print("Rock");
  }
  //这是一个含参构造器
  Rock(int i){
    System.out.print("Rock"+i+" ");
  }
}
</pre>

## 2. 方法重载

为了让方法名相同而形式参数不同的构造器同时存在，必须用到方法重载。尽管方法重载是构造器所必须的，但是它也可以用于其他方法。

### 2.1 区分重载方法

区分重载的规则为：每个重载的方法都必须有一个独一无二的参数类型列表。
需要注意的是：根据方法的返回值来区分重载方法是行不通的。

### 2.2 涉及基本类型的重载

如果传入的数据类型小于方法中声明的形式参数类型，实际数据类型就会被提升。char型略有不同，如果无法找到恰好接受char参数的方法，就会把char直接提升至int型。而如果传入的实际参数较大，就得通过类型转换执行窄化转化，如果不这样做，编译器就会报错。

## 3. 默认构造器

默认构造器（又名“无参构造器”）是没有形式参数的，它的作用是创建一个“默认对象”，如果你所编写的类中没有构造器，则编译器会自动帮你创建一个默认构造器。但是，如果已经定义了一个构造器（无论是否有参数），编译器就不会再自动创建默认构造器。

## 4. this关键字

this关键字只能在方法内部使用，表示对“调用方法的那个对象”的引用。this的用法和其他对象引用并无不同。但是要注意的是，如果在方法内部调用同一个类的另一个方法，就不必使用this,直接调用即可。

### 4.1 在构造器中调用构造器

通过下面的这个例子来进行对this关键字的说明

<pre class=”brush:java;gutter:true;”>
public class Flower{
  int petalCount = 0;
  String s = "initial value"；
  Flower(int petals){
    petalCount = petals;
    System.out.println("Constructor w/ int arg only,petalCount = " + petalCount);
  }
  Flower(String ss){
    System.out.println("Constructor w/ String arg only,s = " + ss);
    s = ss;
  }
  Flower(String s,int petals){
    this(petals);
  //  this(s);      如果调用了this(petals)之后再次调用this(s),会报错
    this.s = s;
    System.out.println("String & int args");
  }
  Flower(){
    this("hi",68);
    System.out.println("default constructor (no args)");
  }  
  void printPetalCount(){
  //  this(11);     除了构造器之外，编译器禁止在其他任何方法中调用构造器
    System.out.println("petalCount = " + petalCount + "s = " + s);
  }
  public static void main(String[] args){
    Flower x = new Flower();
    x.printPetalCount();
  }
}
</pre>

通过以上代码可以知道：
1. 尽管可以用this调用一个构造器，但却不能同时调用两个。
2. 此外，必须将构造器置于最开始处，否则编译器会报错。
3. 除了构造器之外，编译器禁止在其他任何方法中调用构造器。
4. this关键字还有另外一个用法，就是在参数s的名称和数据成员s的名称相同时，使用`this.s`来代表数据成员就可以解决歧义。

### 4.2 static的含义

可以在没有创建任何对象的前提下，仅仅通过类本身来调用static方法，这是static方法的主要用途。在static方法的内部不能调用非静态方法。但是，如果代码中出现了大量的static方法，就应该重新考虑自己的设计了。

## 5. 清理：终结处理和垃圾回收

Java中的垃圾回收工作由垃圾回收器来完成，一旦垃圾回收器准备好释放对象占用的存储空间，将首先调用其`finalize()`方法，并且在下一次垃圾回收动作发生时，才会真正回收对象占用的内存。

### 5.1 finalize()的用途何在

之所以要有`finalize()`，是由于在分配内存时可能采用了类似C语言的做法，而非Java中的通常做法。使用垃圾回收器的唯一原因是为了回收程序中不再使用的内存，垃圾回收只与内存有关。
注意：无论是“垃圾回收”还是“终结”，都不保证一定会发生。如果Java虚拟机（JVM）并未面临内存耗尽的情形，它是不会浪费时间去执行垃圾回收以恢复内存的。

## 6. 成员初始化

Java尽力保证：所有的变量在使用之前都能够得到恰当的初始化，对于方法的局部变量，Java以编译时错误的形式来贯彻这种保证。

## 7. 构造器初始化


