---
title: 【新手学Java】反射学习笔记
tags:
  - Java
  - 反射
abbrlink: 16721
date: 2016-10-03 19:11:11
categories:
  - 学习笔记
---
示例类代码
<!-- more -->

```java
@SuppressWarnings("unused")
public class Person {
    public String Name;
    private int Age;
    public Gender Gender;
    private static String Species = "人类";
    public Person(){
        Name="佚名";
        Age=-1;
    }
    public Person(String name){
        Name=name;
    }
    private Person(String name,int age){
        Name=name;
        Age=age;
    }    
    private Person(Gender g){
        Gender=g;
    }
    public void Run(){
        System.out.println(Name+" 跑!");
    }
    public void Attack(){
        System.out.println(Name+" 打!");
    }
    public void Attack(String name){
        System.out.println(Name+" 打 "+name+"!");
    }
    private void Eat(String food){
        System.out.println(Name+" 吃 "+food);
    }
    public void Introduce()
    {
        System.out.println("我叫"+Name+",我今年"+Age+"岁了。");
    }
    public static void PlayGame(String gameName){
        System.out.println("玩 "+gameName+" 游戏");
    }
    public static void main(String[] args){
        System.out.println("main");
        for(String s:args)
            System.out.println(s);
    }
}

enum Gender{
    Male,Female
}
```
反射类的无参构造函数:
```java
@Test
//反射类的无参构造函数
public void constructor1() throws Exception{
    Class clazz = Class.forName("pro.shaowei.reflect.Person");
    Constructor c=clazz.getConstructor();
    Person p = (Person) c.newInstance();
    Person p1 = (Person) clazz.newInstance();
    p.Introduce();
    p.Run();
    p1.Introduce();
    p1.Run();
}
```
反射类的有参构造函数:
```java
@Test
//反射类的有参构造函数
public void constructor2() throws Exception{
    Class clazz=Class.forName("pro.shaowei.reflect.Person");
    Constructor c=clazz.getConstructor(String.class);
    Person p=((Person) c.newInstance("张三"));
    p.Introduce();
    p.Run();
}
```
反射类的私有构造函数:
```java
@Test
//反射类的私有构造函数
public void constructor3() throws Exception{
    Class clazz=Class.forName("pro.shaowei.reflect.Person");
    //反射私有构造函数时必须从使用 getDeclaredConstructor 方法
    Constructor c=clazz.getDeclaredConstructor(String.class,int.class);
    c.setAccessible(true);//暴力反射
    Person p=((Person) c.newInstance("张三",25));
    p.Introduce();
    p.Run();
}
```
反射类的公有无参方法:
```java
@Test
//反射类的公有无参方法
public void method1() throws Exception{
    Class clazz=Class.forName("pro.shaowei.reflect.Person");
    Person p=(Person) clazz.newInstance();
    Method method=clazz.getMethod("Run");
    method.invoke(p);
}
```
反射类的公有有参方法:
```java
@Test
//反射类的公有有参方法
public void method2() throws Exception{
    Class clazz=Class.forName("pro.shaowei.reflect.Person");
    Person p=(Person) clazz.newInstance();
    Method method=clazz.getMethod("Attack",String.class);
    method.invoke(p,"李四");
}
```
反射类的私有有参方法:
```java
@Test
//反射类的私有有参方法
public void method3() throws Exception{
    Class clazz=Class.forName("pro.shaowei.reflect.Person");
    Person p=(Person) clazz.newInstance();
    Method method=clazz.getDeclaredMethod("Eat",String.class);
    method.setAccessible(true);
    method.invoke(p,"香蕉");
}
```
反射类的静态有参方法:
```java
@Test
//反射类的静态有参方法
public void method4() throws Exception{
    Class clazz=Class.forName("pro.shaowei.reflect.Person");
    Person p=(Person) clazz.newInstance();
    Method method=clazz.getDeclaredMethod("PlayGame",String.class);
    method.setAccessible(true);
    method.invoke(p,"扫雷");
}
```
反射类的main方法:
```java
@Test
//反射类的main方法
public void method5() throws Exception{
    Class clazz=Class.forName("pro.shaowei.reflect.Person");
    Person p=(Person) clazz.newInstance();
    Method method=clazz.getDeclaredMethod("main",String[].class);
    method.setAccessible(true);
    method.invoke(p,(Object)new String[]{"1","2"});
}
```
反射类公有的字段:
```java
@Test
//反射类公有的字段
public void field1() throws Exception{
    Class clazz=Class.forName("pro.shaowei.reflect.Person");
    Person p=(Person) clazz.newInstance();
    Field field=clazz.getField("Name");
    System.out.println(field.get(p));
    field.set(p, "王五");
    p.Introduce();
}

```
反射类私有的字段:
```java
@Test
//反射类私有的字段
public void field2() throws Exception{
    Class clazz=Class.forName("pro.shaowei.reflect.Person");
    Person p=(Person) clazz.newInstance();
    Field field=clazz.getDeclaredField("Age");
    field.setAccessible(true);
    System.out.println(field.get(p));
    field.set(p, 7);
    p.Introduce();
}
```
反射类私有静态的字段:
```java
@Test
//反射类私有静态的字段
public void field3() throws Exception{
    Class clazz=Class.forName("pro.shaowei.reflect.Person");
    Person p=(Person) clazz.newInstance();
    Field field=clazz.getDeclaredField("Species");
    field.setAccessible(true);
    System.out.println(field.get(p));
    field.set(p, "不死族");
    System.out.println(field.get(p));
}
```