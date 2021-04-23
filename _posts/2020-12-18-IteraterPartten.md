---
layout: post
title: 详解23种设计模式之--------（16）迭代器模式
description: designPatterns_16
category: blog
---

[回到目录](#directory)

#### 迭代器模式的定义
* 迭代器模式（Iterater Pattern）又称游标模式，它提供一种顺序访问集合/容器对象元素的方法，而又无需暴露集合内部表示；
* 本质：抽离集合对象迭代行为到迭代器中，提供一种访问接口；
* 属于行为型模式

**生活中的场景**
* 快递寄件分发;
* 刷脸检票进站（不管身份证，护照啊什么的，刷个脸就可以了）。

**迭代器模式的适用场景**
* 访问一个集合对象的内容而又无需暴露集合内部表示;  
* 为遍历不同的集合结构提供一个统一的访问接口；

一般框架中都有提供（常见如jdk等），现在来写一个看看。  
新建一个课程实体类（被迭代的对象）：
```
public class Course {
    private  String name;

    public Course(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
创建一个对Course操作的接口（增加、删除、迭代）：
```
public interface ICourseAggregate {
    void add(Course course);
    void remove(Course course);
    Iterater<Course> iterater();
}
```
接口实现：
```
public class CourseAggregateImpl implements ICourseAggregate {
    private List courseList;

    public CourseAggregateImpl(List courseList) {
        this.courseList = courseList;
    }

    public CourseAggregateImpl() {
    }

    public void add(Course course) {
        courseList.add(course);
    }

    public void remove(Course course) {
        courseList.remove(course);
    }

    public Iterater<Course> iterater() {
        return new IteraterImpl<Course>(courseList);
    }
}
```
模拟创建迭代器，实现两个方法：
```
public interface Iterater<T> {
    T next();
    boolean hasNext();
}
```
实现：
```
public class IteraterImpl<T> implements Iterater<T>{
    private List<T> list;
    private int cursor;
    private T element;

    public IteraterImpl(List<T> list) {
        this.list = list;
    }

    public T next() {
        System.out.print("当前位置:" + cursor + " ");
        element = list.get(cursor);
        cursor++;
        return element;
    }

    public boolean hasNext() {
        if(cursor > list.size() - 1){
            return false;
        }
        return true;
    }
}
```
测试：
```
public class Test {
    public static void main(String[] args) {
        Course course = new Course("java");
        Course course1 = new Course("python");
        Course course2 = new Course("oracle");

        List<Course> list = new ArrayList<Course>();
        ICourseAggregate courseAggregate = new CourseAggregateImpl(list);
        courseAggregate.add(course);
        courseAggregate.add(course1);
        courseAggregate.add(course2);

        System.out.println("======课程列表=======");
        printCourse(courseAggregate);

        courseAggregate.remove(course2);
        System.out.println("======执行删除后de课程列表=======");
        printCourse(courseAggregate);

    }

    private static void printCourse(ICourseAggregate courseAggregate) {
        Iterater<Course> iterater = courseAggregate.iterater();
        while (iterater.hasNext()){
            Course course = iterater.next();
            System.out.println(course.getName());
        }
    }
}
```
结果：
```
======课程列表=======
当前位置:0 java
当前位置:1 python
当前位置:2 oracle
======执行删除后de课程列表=======
当前位置:0 java
当前位置:1 python
```

那么可以看到这就是一个经典迭代器的模型，其实在平时用的jdk中用的原理都是一样的，jdk中的迭代器（java.util.Iterater
remove()方法没有实现（透明写法） | AbstractList实现了remove方法 | HashMap | DefaultCursor）。

**迭代器模式的优点**
* 多态迭代：为不同的聚合提供一致的遍历接口，即一个迭代接口可以访问不同的聚集对象；
* 简化集合对象窗口：迭代器模式将集合对象本身提供的元素迭代接口抽取到了迭代器中，使集合对象无需关心具体迭代行为；
* 元素迭代功能多样化：每个集合对象都可以提供一个或多个不同的迭代器，使得同种元素可以有不同的的迭代行为；
* 解耦迭代与集合：迭代器模式封装了元素迭代方法，迭代算法的变化，不会影响到集合对象的架构；

**迭代器模式缺点**

* 对于比较简单的便利（如数组或有序列表），使用迭代器遍历比较繁琐。

基础语法：E、T、K、V  ?
E: Element  如果集合，规定集合元素的类型；
T：Type     规定该类操作的具体类型
K：Key      规定key的类型
V：Value    规定Value的类型
？： ？     任意类型
<? extends Hi>
占位，不要做强制转换。