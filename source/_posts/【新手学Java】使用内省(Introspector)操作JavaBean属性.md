---
title: 【新手学Java】使用内省(Introspector)操作JavaBean属性
abbrlink: 50715
date: 2016-10-03 23:15:30
tags:
  - Java
  - Introspector
  - javabean
categories:
  - 学习笔记
---
使用内省(Introspector)操作
<!-- more -->
获取类bean中的所有属性:
```java
@Test
//获取类bean中的所有属性
public void test1() throws Exception{
    BeanInfo info = Introspector.getBeanInfo(Person.class);
    PropertyDescriptor[] decriptors = info.getPropertyDescriptors();
    for(PropertyDescriptor decriptor : decriptors){
        //输出属性的名称
        System.out.println(decriptor.getName());
        //输出属性的类型
        System.out.println(decriptor.getPropertyType());
    }
        
}
```
读/写bean中某个属性:
```java
@Test
//操纵bean中某个属性
public void test2() throws Exception{
    Person p=new Person();
    
    PropertyDescriptor decriptor = new PropertyDescriptor("username",Person.class);
    
    //得到属性的写方法
    Method method=decriptor.getWriteMethod();
    method.invoke(p, "张三");

    //得到属性的读方法
    method=decriptor.getReadMethod();
    String username= (String) method.invoke(p);
    System.out.println(username);
}
```