---
layout: post
title: 详解23种设计模式之--------（5）代理模式
description: designPatterns_5
category: blog
---

#### 代理模式的定义
  * 代理模式（Proxy Partten），是指为其它用户提供一种代理，以控制对这个对象的访问；
  * 代理对象在客户端和目标对象之间起到中介作用；
  * 属于结构型设计模式
#### 生活中的代理模式
  * 房产中介；
  * 快递小哥；
  * 黄牛党等等；  
常见的代理模式：SpringAOP；

我们知道代理模式又分为静态代理和动态代理，那么他们又有什么区别和联系呢？  

##### 静态代理（显示声明被代理对象）：
接下来通过一个例子详细了解一下静态代理模式：众所周知，大千世界，芸芸众生，石头能生出猴子，公鸡也会修炼成仙；而最令人匪夷所思的是，唐僧竟然不喜欢美女；
然鹅必不可少的，万物都需要繁衍生息，草木之间叫做授粉，牲畜之间称为交配，人与人之间必须要先建立感情，建立感情的前提是找到另一半；
不得不提到程序员，大家都知道，程序员加班最多，工作枯燥，身边的资源又少，所以很多人年龄逐渐增长头发日渐稀少，逐渐变成了剩男剩女，最后不得不去相亲，但是相完亲之后
又去忙工作了，所以他们的父母不得不承担起给子女找对象的重任。。  
我们现在就将这个现象用静态代理来演示一下：首先创建一个 人 的顶层抽象类：
```
public interface IPerson {
    void findlove();
}
```
然后，假设有找对象需求的这个人叫苏大强：
```
public class SuDaqiang implements IPerson {
    public void findlove() {
        System.out.println("苏大强要求：肤白貌美大长腿");
    }
}
```
身负重任的老爸：
```
public class Sulaoqiang implements IPerson {
    private SuDaqiang suDaqiang;

    public Sulaoqiang(SuDaqiang suDaqiang ){
        this.suDaqiang = suDaqiang;
    }
    public void findlove() {
        System.out.println("老爸开始物色");
        suDaqiang.findlove();
        System.out.println("已找到菜根花小宝贝");
    }
}
```
好了那么创建完之后，编写测试类：
```
public class Test {
    public static void main(String[] args) {
        Sulaoqiang sulaoqiang = new Sulaoqiang(new SuDaqiang());
        sulaoqiang.findlove();
    }
}
```
执行结果：
```
老爸开始物色
苏大强要求：肤白貌美大长腿
已找到菜根花小宝贝
```

那么这里看到，老爸实现了苏大强的需求，并且在这前后还做了一些其他操作；但是这里有一个问题，
老爸的构造函数传的参数是苏大强，那么假设现在需求变了，不只有苏大强了，还有苏小弱，
那么按照这种逻辑，这里苏小若的父亲是肯定会帮苏小弱找对象，但是苏大强的老爸不会去帮苏小弱。
如果10000人都有需求，怎么办？这时候就形成了一个产业链，这个产业链的Boss就是月老；那么月老就会专门去为这些找对象的人服务，
后来慢慢的越来越专业，去月球住了，就形成了动态代理。  
再后来，月老就只干这一件事，符合软件设计的单一职责原则，后来一亿人有需求，月老分分钟就能搞定。扩展性非常强，符合开闭原则。


##### 动态代理（动态声明被代理对象）：

那么现在如何实现**动态代理**呢？目前JDK和CGLIB已经帮我们实现了；  
相似点：利用底层逻辑通过字节码重组动态生成一个代理类，这个类只在内存中实现，与目标类在同一继承体系，
这样在调用这个代理类的时候，会找到生成这个代理类的类的相同的方法，然会通过反射的机制，找到这个类的方法。
在调用之前，做一些事情，在调用之后，做一些事情。

  * jdkProxy
  
修改增加jdk代理类：
 
```
public class JdkYuelao implements InvocationHandler {
    private IPerson target;
    public IPerson getInstance(IPerson target){
        this.target = target;
        Class<?> clazz = target.getClass();//拿到target的class

        return (IPerson)Proxy.newProxyInstance(clazz.getClassLoader(),clazz.getInterfaces(),this);
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        before();
        Object result = method.invoke(this.target,args);
        after();
        return result;
    }
    private void after() {
        System.out.println("牵线成功");
    }
    private void before() {
        System.out.println("我是月老，已收集你的信息");
    }
}
```

改造测试类：

```
public class Test {
    public static void main(String[] args) {
        JdkYuelao jdkYuelao = new JdkYuelao();
        IPerson sudaqiang = jdkYuelao.getInstance(new SuDaqiang());
        sudaqiang.findlove();

        IPerson suxiaoruo = jdkYuelao.getInstance(new SuXiaoruo());
        suxiaoruo.findlove();

    }
}
```

执行俩人的结果：

```
我是月老，已收集你的信息
苏大强要求：肤白貌美大长腿
牵线成功
我是月老，已收集你的信息
苏小弱要求：富婆
牵线成功
```

**可以看出，动态代理这种方式更为优雅的变法方式，不管是谁都可以进行代理，而相比静态代理这种硬编码的形式显得更加的灵活。**

**动态代理的方式：不管你要代理的类、目标是哪一个，都可以动态获取到，而静态代理总是走硬编码**


2. CGLIB代理方式
ssm就是基于cglib代理方式； 

旧版本添加maven依赖；
```
        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>2.2.2</version>
        </dependency>
``` 
代理类：
```
public class CglibYuelao implements MethodInterceptor {

    public Object getInstance(Class<?> clazz){
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(clazz);
        enhancer.setCallback(this);
        return  enhancer.create();
    }

    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        before();
        Object result = methodProxy.invokeSuper(o,objects);//调用父类的，会生成子类
        after();
        return result;
    }

    private void after() {
        System.out.println("牵线成功");
    }

    private void before() {
        System.out.println("我是月老，已收集你的信息");
    }
}
```

有需求的类(不需要实现任何接口)：

```
public class SuDaqiang {
    public void findlove() {
        System.out.println("苏大强要求：肤白貌美大长腿");
    }
}
```

调用：

```
我是月老，已收集你的信息
苏大强要求：肤白貌美大长腿
牵线成功
```

cglib的优点就是不需要用户有必要的前提条件，也就是说只要是一个class就好了；