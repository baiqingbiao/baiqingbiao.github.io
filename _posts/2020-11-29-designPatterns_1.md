---
layout: post
title: 详解23种设计模式之--------（1）工厂模式
description: designPatterns_1
category: blog
---


### 软件设计模式作为软件架构的内功心法，想必也是一门必修基础知识，了解与学习也要趁早，今天起就来学习一下。
#### 首先来看第一节，工厂模式，其中工厂模式又分为三种：
##### 1. 简单工厂模式（Simple Design Pattern）
  指由一个工厂对象决定创建出哪一种产品的实例，属于创建型模式；此处声明一下，简单工厂模式不属于GOF 23种设计模式，此处只是为了过度一下，学习其中的思想。
  废话不多说，我们将通过一个例子来了解一下。<br>
  现在假设我们要创建一门课程：
  ```
  public class Course {
     void record(){
       System.out.println("录制课程");
     }
}
  ```
  原先我们是这样手动创建的：
  ```
  public class Test {
     public static void main(String[] args){
       Course course = new Course();
       course.record();
     }
}
  ```
  那现在假设我们又要去创建一门课程，比如Java课程，可能我们会想直接将Course拷贝一份，然后再去手动创建JavaCourse的实例，但这时候就违背了面向抽象编程的设计原则；这个时候就需要把课程抽象出来，变成接口：
  
 ```
  public interface ICourse {
     void record();
}
  ```
  此时就出现了工厂，这时候就让JavaCourse去实现：
  ```
  public class JavaCourse implements ICourse {

    @Override
    public void record() {
        System.out.println("录制java course");
    }
}
```
同样的，后续再有其他的课程，就去实现这个课程Course，调用的时候，就不再是自给自足的模式：
```
public class Test {
     public static void main(String[] args){
       ICourse iCourse = new JavaCourse();
       course.record();
     }
}
```
这还不能完全叫做工厂，因为JavaCourse还是手动去创建的，工厂的概念就是将创建过程包装起来，具体怎么创建用户不用去关心，而是由工厂去管理，这也符合单一职责原则。<br>
那这个时候我们就去创建一个工厂类去统一去管理课程：
```
public class CourseFactory{
    public ICourse create(String courseName){//方法返回类型要比父类更小，更加严谨
        if("java".equals(courseName)){
            return new JavaCourse();
        }else if("python".equals(courseName)){
            return new PythonCourse();
        }else{
            return null;
        }
    }
}
```
这个时候再去创建就直接给工厂类的创建方法传递一个标识，就可以创建需要的实例；
```
public class Test {
     public static void main(String[] args){
       ICourse iCourse = new CourseFactory().create("java");
       course.record();
     }
}
```
后面再怎么加，创建逻辑也不会再变，这个时候思路就会变得很清晰。<br>
当然，这里传“java”不太合适，我们可以做个升级，这里用到java的一个反射技术，传递一个class name：
```
public class Test {
     public static void main(String[] args){
       ICourse iCourse = new CourseFactory().create("com.design.pattern1.JavaCource");//包名+类名
       course.record();
     }
}
```
  同时把工厂升级： 
  ```
public class CourseFactory{
    public ICourse create(String className){//方法返回类型要比父类更小，更加严谨
        try{
            if(null != className && !"".equals(className)) {
                return (ICourse)className.forName(className).newInstance();
            }
        }catch(Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
但是Spring里不是这么干的，他传的是Class对象，他不是传的类名，这时候在升一下级：
  ```
public class CourseFactory{
    public ICourse create(Class<? extends ICourse> clas){//规定是接口子类，否则不给创建
        try{
            if(null != clss) {
                return clas.newInstance();//这时就不用再进行强转(ICourse)
            }
        }catch(Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
创建的时候：
```
public class Test {
     public static void main(String[] args){
       ICourse course = new CourseFactory().create(JavaCourse.class);//写错会被检测到
       course.record();
     }
}
```
这就是简单工厂的一个例子，包括java源码中的Calendar.getInstance()、LoggerFactory()等诸多底层也是这么实现的。<br>
简单工厂模式的适用场景：<br>
1. 工厂类创建的对象较少<br>
2. 客户端只需要传入工厂类的参数，对于如何创建对象的逻辑不需要关心。<br>
------

