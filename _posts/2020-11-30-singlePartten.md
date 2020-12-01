---
layout: post
title: 详解23种设计模式之--------（2）单例模式
description: designPatterns_2
category: blog
---

### 单例模式
#### 定义
  * 确保一个类在任何情况下都只有一个实例，并提供一个全局访问点；
  * 隐藏其所有的构造方法；
  * 属于创建型模式
#### 单例模式的适用场景
  * 确保任何情况下都只有一个实例
#### 单例模式的常见写法
  * 饿汉式单例
  * 懒汉式单例
  * 注册时单例
  * ThreadLocal单例
##### 饿汉式单例  
```
 /**
  *优点：性能高、执行效率高、没有任何锁
  *缺点：某些情况下，可能会造成内存浪费
  */
 public class HungrySingleton{
    private static HungrySingleton hungrySingleton = new HungrySingleton();
    private HungrySingleton(){}
    public static HungrySingleton getInstance(){return hungrySingleton}
 }
```
 另一种写法：  
```
  /**
  *先静态后动态
  *先上，后下
  *先属性，后方法
  */
  public class HungrySingleton{
    private static HungrySingleton hungrySingleton;
    //装个B，和上面一样
    static{
       hungrySingleton = new HungrySingleton()
    }
    private HungrySingleton(){}
    public static HungrySingleton getInstance(){return hungrySingleton}
 }
```  
    造成内存浪费的缺点怎那么解决：
##### 懒汉式单例  
```
  /**
  *优点：节省内存
  *缺点：可能会线程不安全（多个线程同时判断实例为空，同时new，这时候就创建了多个实例，可使用多线程测试）
  */
 public class LazySimpleSingleton{
    private static LazySimpleSingleton instance;//初始化不赋值/不占内存空间
    private LazySimpleSingleton();
    
    public static LazySimpleSingleton getInstance(){
        if(instance == null){
            instance = new LazySimpleSingleton();
        }
        return instance;
    }
 }
```
