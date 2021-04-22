---
layout: post
title: 详解23种设计模式
description: designPatterns
category: blog
---

## <a id="directory">目录</a>

[1. 工厂模式](#directory1)

[2. 单例模式](#directory2)

[3. 原型模式](#directory3)

[4. 建造者模式](/123123/jvm)








## <a id="directory1">工厂模式</a>

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

[回到目录](#directory)

## <a id="directory2">单例模式</a>

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
	
[回到目录](#directory)

## <a id="directory3">原型模式</a>

#### 原型模式的定义：
  * 原型模式（ProtoType Pattern）是指原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象；
  * 调用者不需要知道任何创建细节，不调用构造函数；
  * 属于创建型模式；  
关键信息：不是通过new，而是通过一个方法去创建实例，这个方法会拷贝并保留原先的值。

例：顶层抽象（带有clone方法）
```
public interface IProtoTypePattern<T> {
    T clone();
}

```
实体类：
```
public class ConcreteProtoyype implements IProtoTypePattern{
    private int age;
    private String name;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public ConcreteProtoyype clone() {
        ConcreteProtoyype concreteProtoyype = new ConcreteProtoyype();
        concreteProtoyype.setAge(this.age);
        concreteProtoyype.setName(this.name);
        return concreteProtoyype;
    }

    @Override
    public String toString() {
        return "ConcreteProtoyype{" +
                "age=" + age +
                ", name='" + name + '\'' +
                '}';
    }
}
```
客户端调用：
```
public class Client {
    public static void main(String[] args) {
        //创建原型对象
        ConcreteProtoyype concreteProtoyype = new ConcreteProtoyype();
        concreteProtoyype.setAge(18);
        concreteProtoyype.setName("huyawen");
        System.out.println(concreteProtoyype);

        //拷贝原型对象
        ConcreteProtoyype concreteProtoyype1  = concreteProtoyype.clone();
        System.out.println(concreteProtoyype1);
    }
}
```
执行结果：
```
ConcreteProtoyype{age=18, name='huyawen'}
ConcreteProtoyype{age=18, name='huyawen'}
```

##### 原型模式的适用场景：
 * 类初始化消耗资源过多；
 * new 产生一个对象的过程非常繁琐的过程（数据准备，访问权限等）；
 * 构造函数比较复杂；
 * 循环体中创建多个对象时；
 
 其中原型模式又分为浅克隆和深克隆：
##### 1. 浅克隆：实现Cloneable接口：
 
   
```
@Data
public class ConcreteProtoyype implements Cloneable{
    private int age;
    private String name;
    private List<String> hobbies;

    @Override
    public ConcreteProtoyype clone() {
        try {
            return (ConcreteProtoyype)super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }

    @Override
    public String toString() {
        return "ConcreteProtoyype{" +
                "age=" + age +
                ", name='" + name + '\'' +
                ", hobbies=" + hobbies +
                '}';
    }
}
```
客户端调用：
```
public class Client {
    public static void main(String[] args) {
        //创建原型对象
        ConcreteProtoyype concreteProtoyype = new ConcreteProtoyype();
        concreteProtoyype.setAge(18);
        concreteProtoyype.setName("huyawen");
        List<String> hobbies = new ArrayList<String>();
        hobbies.add("吃");
        hobbies.add("睡");
        hobbies.add("拉屎");
        concreteProtoyype.setHobbies(hobbies);
        System.out.println(concreteProtoyype);

        //拷贝原型对象
        ConcreteProtoyype concreteProtoyype1  = concreteProtoyype.clone();
        concreteProtoyype1.getHobbies().add("学习");
        System.out.println(concreteProtoyype1);
    }
}
```
这时候的输出：
```
ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎]}
ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎, 学习]}

```
未发现异常，拷贝原型对象之后再看一下原型对象有没有发生变化：
```
public class Client {
    public static void main(String[] args) {
        //创建原型对象
        ConcreteProtoyype concreteProtoyype = new ConcreteProtoyype();
        concreteProtoyype.setAge(18);
        concreteProtoyype.setName("huyawen");
        List<String> hobbies = new ArrayList<String>();
        hobbies.add("吃");
        hobbies.add("睡");
        hobbies.add("拉屎");
        concreteProtoyype.setHobbies(hobbies);

        //拷贝原型对象
        ConcreteProtoyype concreteProtoyype1  = concreteProtoyype.clone();
        concreteProtoyype1.getHobbies().add("学习");
        System.out.println("原型对象："+concreteProtoyype);
        System.out.println("克隆对象："+concreteProtoyype1);
        System.out.println(concreteProtoyype == concreteProtoyype1);

        System.out.println("原型对象的爱好：" + concreteProtoyype.getHobbies());
        System.out.println("克隆对象的爱好：" + concreteProtoyype1.getHobbies());
        System.out.println(concreteProtoyype.getHobbies() == concreteProtoyype1.getHobbies());
    }
}
```
执行结果：
```
原型对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎, 学习]}
克隆对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎, 学习]}
false
原型对象的爱好：[吃, 睡, 拉屎, 学习]
克隆对象的爱好：[吃, 睡, 拉屎, 学习]
true
```
这时候发现原型对象发生了变化，但是仔细观察，克隆对象和原型对象并不是一个对象，那么为什么会随着变化呢?  
接着看对象的爱好地址是相同的，这时候就说明浅克隆的一个问题：对基本数据类型或String类型等类型可以使用，对引用型数据类型拷贝的是对象地址，而不是具体的元素。
这时候就要应用到了：  
##### 2. 深克隆
  通过序列化和反序列化实现的方式，遵循里氏替换原则；  
  增加方法（记得实现Serializable接口，否则会抛出序列化异常）：
```
    public ConcreteProtoyype deepClone() {
        try{
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(bos);
            oos.writeObject(this);

            ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bis);
            return (ConcreteProtoyype)ois.readObject();
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }
    }
```
客户端直接调用：
```
public class Client {
    public static void main(String[] args) {
        //创建原型对象
        ConcreteProtoyype concreteProtoyype = new ConcreteProtoyype();
        concreteProtoyype.setAge(18);
        concreteProtoyype.setName("huyawen");
        List<String> hobbies = new ArrayList<String>();
        hobbies.add("吃");
        hobbies.add("睡");
        hobbies.add("拉屎");
        concreteProtoyype.setHobbies(hobbies);

        //拷贝原型对象
        ConcreteProtoyype concreteProtoyype1  = concreteProtoyype.deepClone();
        concreteProtoyype1.getHobbies().add("学习");
        System.out.println("原型对象："+concreteProtoyype);
        System.out.println("克隆对象："+concreteProtoyype1);
        System.out.println(concreteProtoyype == concreteProtoyype1);

        System.out.println("原型对象的爱好：" + concreteProtoyype.getHobbies());
        System.out.println("克隆对象的爱好：" + concreteProtoyype1.getHobbies());
        System.out.println(concreteProtoyype.getHobbies() == concreteProtoyype1.getHobbies());
    }
}
```
打印结果：
```
原型对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎]}
克隆对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎, 学习]}
false
原型对象的爱好：[吃, 睡, 拉屎]
克隆对象的爱好：[吃, 睡, 拉屎, 学习]
false
```
可以看到这次符合了期望值，克隆对象的引用类型发生了变化，而原型对象还是原来那个。  
更好的实现方式：Json字符串反序列化方式。  
引发的问题：  
    1. 性能问题、占用I/O、安全性；
    2. 破坏单例：
例子：创建饿汉式单例：
```
public static ConcreteProtoyype instance = new ConcreteProtoyype();
    private ConcreteProtoyype(){}

    public static ConcreteProtoyype getInstance(){
        return instance;
    }
```
调用：
```
public class Client {
    public static void main(String[] args) {
        //创建原型对象
        ConcreteProtoyype concreteProtoyype = ConcreteProtoyype.getInstance();
        concreteProtoyype.setAge(18);
        concreteProtoyype.setName("huyawen");
        List<String> hobbies = new ArrayList<String>();
        hobbies.add("吃");
        hobbies.add("睡");
        hobbies.add("拉屎");
        concreteProtoyype.setHobbies(hobbies);

        //拷贝原型对象
        ConcreteProtoyype concreteProtoyype1  = concreteProtoyype.deepClone();
        concreteProtoyype1.getHobbies().add("学习");
        System.out.println("原型对象："+concreteProtoyype);
        System.out.println("克隆对象："+concreteProtoyype1);
        System.out.println(concreteProtoyype == concreteProtoyype1);

        System.out.println("原型对象的爱好：" + concreteProtoyype.getHobbies());
        System.out.println("克隆对象的爱好：" + concreteProtoyype1.getHobbies());
        System.out.println(concreteProtoyype.getHobbies() == concreteProtoyype1.getHobbies());
    }
}
```
打印：
```
原型对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎]}
克隆对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎, 学习]}
false
原型对象的爱好：[吃, 睡, 拉屎]
克隆对象的爱好：[吃, 睡, 拉屎, 学习]
false
```
发现这里两个对象不同，所以单例被破坏了。。。那么如何解决？  
  * 不实现Cloneable接口；（注意：**单例模式不能实现Cloneable接口**）
    
```
  @Override
    public ConcreteProtoyype clone() {
        return instance;
    }
```
 此时又变回了原型模式，没错，**原型和单例本身就是冲突的！！**，这种理论客观上就不存在。。
 作为一名架构师，要考虑代码的风险，要规避任何可能发生的情况，要防患于未来！
 有兴趣可以看一下JDK源码中的  
ArrayListClone方式：
```
     public Object clone() {
        try {
            ArrayList var1 = (ArrayList)super.clone();
            var1.elementData = Arrays.copyOf(this.elementData, this.size);
            var1.modCount = 0;
            return var1;
        } catch (CloneNotSupportedException var2) {
            throw new InternalError(var2);
        }
    }
```
```
     public static <T> T[] copyOf(T[] original, int newLength) {
        return (T[]) copyOf(original, newLength, original.getClass());
    }
```
```
     public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```
 
HashMap：
```
     public Object clone() {
        HashMap var1;
        try {
            var1 = (HashMap)super.clone();
        } catch (CloneNotSupportedException var3) {
            throw new InternalError(var3);
        }

        var1.reinitialize();
        var1.putMapEntries(this, false);
        return var1;
    }
```
```
     void reinitialize() {
        this.table = null;
        this.entrySet = null;
        this.keySet = null;
        this.values = null;
        this.modCount = 0;
        this.threshold = 0;
        this.size = 0;
    }
 ```
 ```
     final void putMapEntries(Map<? extends K, ? extends V> var1, boolean var2) {
        int var3 = var1.size();
        if (var3 > 0) {
            if (this.table == null) {
                float var4 = (float)var3 / this.loadFactor + 1.0F;
                int var5 = var4 < 1.07374182E9F ? (int)var4 : 1073741824;
                if (var5 > this.threshold) {
                    this.threshold = tableSizeFor(var5);
                }
            } else if (var3 > this.threshold) {
                this.resize();
            }

            Iterator var8 = var1.entrySet().iterator();

            while(var8.hasNext()) {
                Entry var9 = (Entry)var8.next();
                Object var6 = var9.getKey();
                Object var7 = var9.getValue();
                this.putVal(hash(var6), var6, var7, false, var2);
            }
        }
    }
```
 用这种方式改造一下我们的代码：
```
     public ConcreteProtoyype deepCloneHobbies() {
        try{
            ConcreteProtoyype result = (ConcreteProtoyype)super.clone();
            result.hobbies = (List) ((ArrayList)result.hobbies).clone();
            return result;
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }
    }
```
 客户端调用：
```
 ConcreteProtoyype concreteProtoyype1  = concreteProtoyype.deepCloneHobbies();
```
 结果：
 
```
原型对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎]}
克隆对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎, 学习]}
false
原型对象的爱好：[吃, 睡, 拉屎]
克隆对象的爱好：[吃, 睡, 拉屎, 学习]
false
```

### 总结

   只要是Cloneable接口下的都是浅克隆：ArrayList看似为深克隆，假设ArrayList里有引用类型呢？  
    怎么做才能实现深克隆：  
           序列化  
           转Json  
    如果不是架构师没必要了解原型模式，因为用不到；（apach.lang.util包；jdk实现了浅克隆；spring）  
#### 原型模式优点  

    性能优良，java自带的原型模式，是基于内存的二进制流的拷贝，比直接new一个对象性能更高；  
    可以使用深克隆方式保存对象的状态，使用原型模式将原对象复制一份并保存，简化了创建过程  
#### 原型模式的缺点

     必须配备克隆（或拷贝）方法  
     当对已有类进行改造的时候，需要修改代码，违反了开闭原则；  
     深拷贝，浅拷贝需要运用得当。  
	 
[回到目录](#directory)
