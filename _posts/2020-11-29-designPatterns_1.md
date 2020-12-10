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
<br>


##### 2. 工厂方法模式（Factory Method Pattern）
  · GOF 第一种设计模式：是指定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类，工厂方法让类的实例化推迟到子类中进行，属于创建型模式<br>
  · 与简单工厂类似，用户只需要关心产品由哪个工厂实现，而不用关心产品实现的细节；<br>
  · 后续如果新加入产品，工厂创建的逻辑不会变，符合开闭原则（对修改关闭，对扩展开放）<br>
  · 根据单一职责原则（专类干专事）划分开来
  通过一个例子来看,首先要设计一个课程工厂顶层抽象：
  ```
  public interface ICourseFactory{
      ICourse create();
  }
  ```
  然后设计一个JavaCourseFactory、PythonCourseFactory工厂去实现它：
  ```
  public class JavaCourseFactory implements ICourseFactory{
      public ICourse create(){  
          return new JavaCourse();
      }
  }
  ```
  ```
  public class PythonCourseFactory implements ICourseFactory{
      public ICourse create(){  
          return new PythonCourse();
      }
  }
  ```
  编写测试类
  ```
  public class Test {
      public static void main(String[] args){
          ICourseFactory factory = new PythonCourseFactory();
          ICourse course = factory.create();
          course.record();
      }
  }
  ```
工厂方法模式适用场景：
· 创建对象需要大量的重复代码；  
· 客户端（应用层）不依赖于产品类实例如何被创建、实现等细节；  
· 一个类通过其子类来指定创建那个对象；  
工厂方法模式的缺点：  
· 类的个数容易过多，增加了代码结构的复杂度；  
· 增加了系统的抽象性和理解难度；

##### 3. 抽象工厂模式（Abastract Factory Pattern）
· 提供一个一系列相关或相互依赖的接口，无需指定他们具体的类。<br>
· 属于创建型设计模式。<br>
[点这里看菜鸟的介绍](https://www.runoob.com/design-pattern/abstract-factory-pattern.html)<br>
假如现在要实现这几个接口负责的功能：
```
public interface ICourse {
    void record();
}
```
```
public interface INote {
    void edit();
}
```
```
public interface IVideo {
    void play();
}
```
然后编写Java和Python的实现类：
```
public class JavaCourse implements ICourse {

    public void record() {
        System.out.println("录制java course");
    }
}
```
```
public class JavaNote implements INote{
    @Override
    public void edit() {

    }
}

```
```
public class JavaVideo implements IVideo {
    @Override
    public void play() {

    }
}
```
```
public class PythonCourse implements ICourse {

    public void record() {
        System.out.println("录制python course");
    }
}
```
```
public class PythonNote implements INote{
    @Override
    public void edit() {

    }
}
```
```
public class PythonVideo implements IVideo {
    @Override
    public void play() {

    }
}
```
然后建立这个课程（产品）族的抽象：
```
public abstract class CourseFactory {//同一个产品族
    public void init(){
        System.out.println("初始化基础数据");
    }

    protected abstract ICourse createCourse();

    protected abstract INote createNote();

    protected abstract IVideo createVideo();
}
```
这时候再去创建课程类继承它的课程族群:
```
public class JavaCourseFactory extends CourseFactory {
    @Override
    protected ICourse createCourse() {
        super.init();
        return new JavaCourse();
    }

    @Override
    protected INote createNote() {
        super.init();
        return new JavaNote();
    }

    @Override
    protected IVideo createVideo() {
        super.init();
        return new JavaVideo();
    }
}
```
```
public class PythonCourseFactory extends CourseFactory {
    @Override
    protected ICourse createCourse() {
        super.init();
        return new PythonCourse();
    }

    @Override
    protected INote createNote() {
        super.init();
        return new PythonNote();
    }

    @Override
    protected IVideo createVideo() {
        super.init();
        return new PythonVideo();
    }
}
```
编写测试类：
```
public class Test {
    public static void main(String[] args) {
        CourseFactory factory = new JavaCourseFactory();
        factory.createCourse().record();
        factory.createNote().edit();
        factory.createVideo().play();
    }
}
```
#### 总结：
+ 以上就是抽象工厂的一个综合，虽然他的产品结构变得更加复杂，但其符合类的单一职责原则；<br>
+ 完整的描述了Java和Python两个产品等级的关系；<br>
+ 扩展费力，每一层都要修改<br>
#### 抽象工厂模式的缺点：<br>
+ 规定了所有可能被创建的集合，产品族中扩展新的产品困难，需要修改抽象工厂的接口；<br>
+ 增加了系统的抽象性和理解难度。<br>
#### 抽象工厂模式的优点：<br>
- 具体产品在应用层代码隔离，无需关心创建细节；<br>
- 将一个系列的产品族统一到一起创建。<br>
#### 抽象工厂模式的适用场景：<br>
* 客户端（应用层）不依赖于产品类实例如何被创建、实现等细节。<br>
* 强调一系列相关的产品对象（属于同一产品族）一起使用创建对象需要大量重复代码；<br>
* 提供一个产品类的库，所有的产品以同样的接口出现从而使客户端不依赖于具体的实现。<br>
* 产品族，一系列的相关的产品，整合到一起有关联性；
* 产品等级，同一个继承体系；
