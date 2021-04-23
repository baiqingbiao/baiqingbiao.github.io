---
title: 详解23种设计模式之--------（2）单例模式
description: designPatterns_2
category: blog
---

## <a id="directory10">工厂模式</a>


### 单例模式
#### 定义
  * 确保一个类在任何情况下都只有一个实例，并提供一个全局访问点；
  * 隐藏其所有的构造方法；
  * 属于创建型模式;  
#### 单例模式的适用场景
  * 确保任何情况下都只有一个实例;  
#### 单例模式的常见写法
  * 饿汉式单例;
  * 懒汉式单例;
  * 注册时单例;
  * ThreadLocal单例;  
#### 饿汉式单例  
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
       private HungrySingleton(){
    }
    public static HungrySingleton getInstance(){
       return hungrySingleton
    }
 }
```  
    造成内存浪费的缺点怎那么解决：
#### 懒汉式单例    
```
  /**
  *优点：节省内存
  *缺点：可能会线程不安全（多个线程同时判断实例为空，同时new，这时候就创建了多个实例，可使用多线程测试）
  */
 public class LazySimpleSingleton{
    private static LazySimpleSingleton instance;//初始化不赋值,不占内存空间
    private LazySimpleSingleton();
    
    public static LazySimpleSingleton getInstance(){
        if(instance == null){
            instance = new LazySimpleSingleton();
        }
        return instance;
    }
 }
```
*这是个测试案例*  
创建一个类实现多线程，打印出当前线程获得的实例hashcode值：
```
public class SingletonMoreThread implements Runnable{
    @Override
    public void run() {
        Singleton singleton = Singleton.getInstance();
        System.out.println(Thread.currentThread().getName() + ":" + singleton);
    }
}
```
编写测试类：
```
public class SingletonTest {
    public static void main(String[] args) {
        Thread thread1 = new Thread(new SingletonMoreThread());
        Thread thread2 = new Thread(new SingletonMoreThread());
        thread1.start();
        thread2.start();
        System.out.println("end");
    }
}
```
通过调试线程，发现有三种情况：
    1. 实例相同，自始至终都是第一个实例；
    2. 实例相同，第二个把第一个覆盖掉了（此时都进入了判断，创建了两个实例，但都未打印）；
    3. 实例不相同，创建了两个；
那么这个时候，2和3就是不安全的了，如何解决这个问题呢，就要使用线程锁：


```
 public class LazySimpleSingleton{
    private static LazySimpleSingleton instance;
    private LazySimpleSingleton();
    
    public synchronized static LazySimpleSingleton getInstance(){
        if(instance == null){
            instance = new LazySimpleSingleton();
        }
        return instance;
    }
 }
```

优点：节省了内存的消耗，保证了线程安全  
缺点：性能低，受到了限制  
问题：如何解决synchronized关键字的性能问题?  

优化一下，在里面排队：
```
 public class LazySimpleSingleton{
    private static LazySimpleSingleton instance;
    private LazySimpleSingleton();
    
    public static LazySimpleSingleton getInstance(){
        synchronized(LazySimpleSingleton.class){
            if(instance == null){
                instance = new LazySimpleSingleton();
            }
        }
        return instance;
    }
 }
```
深度优化一下，实例为空在阻塞：
```
 public class LazySimpleSingleton{
    private static LazySimpleSingleton instance;
    private LazySimpleSingleton();
    
    public static LazySimpleSingleton getInstance(){
        if(instance == null){
            synchronized(LazySimpleSingleton.class){
                instance = new LazySimpleSingleton();
            }
        }
        return instance;
    }
 }
```
注意，此时做个骚操作，线程1和2同时进入实例为空的判断内部，发现还是new了两个实例，所以这种方式没用。  
所以还需要再加一个判断：
```
 public class LazySimpleSingleton{
    private static LazySimpleSingleton instance;//①
    private LazySimpleSingleton();
    
    public static LazySimpleSingleton getInstance(){
        if(instance == null){
            synchronized(LazySimpleSingleton.class){
                if(instance == null){
                    instance = new LazySimpleSingleton();//②
                    //指令重排序
                }
            }
        }
        return instance;
    }
 }
```
这就是双重检查法，问题解决了，同时也会提高性能。  
但是这里还有一个问题：指令重排序的问题：  
在线程的执行环境中，有一定的随机性，线程会不断的抢cpu时间片，同时有几个操作，  
第一个是分配实例的内存地址②，  
第二个是分配变量的内存地址①，即①变量还要指向②的地址（指针），  
实际这时候就创建了两个内存地址，而这两个地址创建的时候就会出现一个先后问题，会造成线程紊乱，所以此时要加volatile关键字：
```
 public class LazySimpleSingleton{
    private volatile static LazySimpleSingleton instance;//①
    private LazySimpleSingleton();
    
    public static LazySimpleSingleton getInstance(){
        if(instance == null){
            synchronized(LazySimpleSingleton.class){
                if(instance == null){
                    instance = new LazySimpleSingleton();//②
                    //指令重排序
                }
            }
        }
        return instance;
    }
 }
```
折腾了半天问题终于解决了，有的问题更是来回折腾，说白了，就是程序的可读性非常差，可以说不够优雅。那么，如何来进行优化呢？
可能我们心中一直在找寻一种完美的解决方案，所以就有了第三者的出现：  
利用静态内部类（与静态变量相比，静态变量是加载时就分配内存空间，而静态内部类是在你用的时候分配）
如下：  
```

/**
 * SingletonBetter.class(类加载一开始只会扫描这个类，而在调用SingletonBetter类中的getInstance方法时再去加载内部类SingletonHolder.class)
 * SingletonBetter$SingletonHolder.class
 * 利用java语法，实现延迟加载
 *
 * 优点：  1.写法优雅
 *         2.利用java本身的语法特点
 *         3.性能高，避免内存浪费
 * 缺点：能够被反射破坏
 */
public class SingletonBetter {

    private SingletonBetter(){}
    public  static SingletonBetter getInstance(){
        return SingletonHolder.instance;
    }

    private static class SingletonHolder{
        private static final SingletonBetter instance = new SingletonBetter();
    }
}
```
当然这种方式发现也有缺点，那么用该如何选择，这时候就用到了找对象法则，首先如果能接受ta的缺点，那么你就用，ta就是你的，因为其他人都只接受ta的优点。
待续。。。

这个问题官方的解决方式：  
    注册史单例，最优雅的解决方式：枚举类型的单例，已加入此判断，有兴趣自查。
