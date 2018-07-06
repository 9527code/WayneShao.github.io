---
title: 【新手学Java】使用beanUtils控制javabean
tags:
  - Java
  - beanUtils
  - javabean
abbrlink: 33316
date: 2016-10-03 23:28:21
---
使用beanUtils控制javabean
<!-- more -->

使用BeanUtils设置/读取属性的值以及默认支持的自动转化:
```java
@Test
//使用BeanUtils设置/读取属性的值以及自动转化
public void test1() throws IllegalAccessException, InvocationTargetException, NoSuchMethodException{
    Person p=new Person();
    //使用BeanUtils设置属性的值
    BeanUtils.setProperty(p, "username", "李四");
    //使用BeanUtils读取属性的值        
    System.out.println(BeanUtils.getProperty(p, "username"););
    //类型不同依然可以自动转化,BeanUtils默认支持八种基本类型的转换
    BeanUtils.setProperty(p,"age", "123");
    System.out.println(p.getAge());
    
}
```
注册已有的转化器来完成复杂类型的自动转化:
```java
@Test
//注册已有的转化器来完成复杂类型的自动转化
public void test3() throws IllegalAccessException, InvocationTargetException{
    Person p=new Person();
    String birthday="1995-05-05";
    
    //注册Apache提供的时间转换器
    ConvertUtils.register(new DateLocaleConverter(), Date.class);
    
    BeanUtils.setProperty(p, "birthday", birthday);
    
    System.out.println(p.getBirthday());
}
```
 Apache已有的时间转化器中不能很好地过滤空字符串，若待转换字符串为空则会抛出异常；而现实业务非常复杂，Apache无法提供给我们所有的类型转化方法，需要时我们可以注册自己需要的转换器完成业务需求。
 
 注册自己的转换器完成时间转化：
 ```java
@Test
//注册自己的转换器完成时间转化
public void test2() throws IllegalAccessException, InvocationTargetException{
    Person p=new Person();
    String birthday="1995-05-05";
    
    //为了日期可以赋值到bean的属性,我们给benUtils注册日期转换器
    ConvertUtils.register(new Converter(){
        @SuppressWarnings({ "unchecked", "rawtypes" })
        public Object convert(Class type,Object value){
            if(value==null){
                return null;
            }
            if(!(value instanceof String)){
                throw new ConversionException("只支持String类型的转换");
            }
            String str=(String) value;
            if(str.trim().equals("")){
                return null;
            }
            SimpleDateFormat dateformate=new SimpleDateFormat("yyyy-MM-dd");
            try {
                return dateformate.parse(str);
            } catch (ParseException e) {
                throw new RuntimeException(e);
            }                
        }
    }, Date.class);
    
    BeanUtils.setProperty(p, "birthday", birthday);
    
    System.out.println(p.getBirthday());
}
```
直接使用map对象填充类:
```java
@Test
//直接使用map对象填充类
public void test4() throws Exception{
    HashMap<String, String> map=new HashMap<String,String>();
    map.put("username","李四");
    map.put("password","lisi");
    map.put("age","26");
    map.put("birthday","1990-05-05");
    
    ConvertUtils.register(new DateLocaleConverter() , Date.class);
    
    Person p=new Person();
    BeanUtils.populate(p, map);
    
    System.out.println(p.getUsername());
    System.out.println(p.getPassword());
    System.out.println(p.getAge());
    System.out.println(p.getBirthday());
    
}