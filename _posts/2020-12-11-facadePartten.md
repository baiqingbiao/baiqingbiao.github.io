---
layout: post
title: 详解23种设计模式之--------（6）门面模式
description: designPatterns_6
category: blog
---

#### 门面模式的定义
  * 门面模式（Facade Partten），又叫外观模式，提供了一个统一的接口，用来访问子系统中的一群接口；
  * 特征：门面模式定义了一个高层接口，让子系统更容易使用；
  * 属于结构型设计模式

**门面模式大家应该经常都在用，有时可能自己都不知道，自作聪明感觉自己写了一个非常好的方法，学了设计模式之后，才发现早就已经有人提出这个方法论了！**

#### 门面模式适用场景  

	* 子系统越来越复杂，增加门面模式来提供简单接口；
	* 构建多层系统结构，利用门面对象作为每层的入口，简化层间调用；
	
	
#### 生活中的门面模式  

	* 前台接待员
	* 包工头
	* 赛利亚  
	
以包工头为例，假设现在要建一座大楼，那么你肯定要找到水泥工、钢筋工、木工、水电工、搬砖工、粉刷匠等等，特别麻烦，那么门面模式的意思就是你不需要一一去找
这些人，你只需要去找一个包工头，让他的团队或者他再去找人来修建这座楼，就可以了。那么这个包工头就是门面模式中的高层接口；（感觉有点像代理模式）
**非常简单，看个例子**
假设现在要调用三个子系统的方法：

```
public class SubsystemA {
    public void doA(){
        System.out.println("doA");
    }
}

public class SubsystemB {
    public void doB(){
        System.out.println("doB");
    }
}

public class SubsystemC {
    public void doC(){
        System.out.println("doC");
    }
}
```
facade 类：

```
public class Facade {

    SubsystemA subsystemA = new SubsystemA();
    SubsystemB subsystemB = new SubsystemB();
    SubsystemC subsystemC = new SubsystemC();

    public void doA(){
        this.subsystemA.doA();
    }
    public void doB(){
        this.subsystemB.doB();
    }
    public void doC(){
        this.subsystemC.doC();
    }
}
```
调用：
```
public class Test {
    public static void main(String[] args) {
        Facade facade= new Facade();
        facade.doA();
        facade.doB();
        facade.doC();
    }
}
```
这种可以说无处不见，看起来很像Controller，没错，它本身也是个Facade！  
现在来使用门面模式优化一个场景：

假设现在要使用积分兑换一件商品，要有这么几个过程：1.积分是否足够；2.积分扣减；3.物流信息  
那么原有的模式是这样的：
```
import lombok.Data;

@Data
public class GiftInfo {
    private String name;

    public GiftInfo(String name) {
        this.name = name;
    }
}
```

```
public class CheckIntegral {
    public boolean checkJf(int a){
        System.out.println("积分足够");
        return true;
    }
}
```

```
public class IntegralDec {
    public boolean dec(int a){
        System.out.println("购买完成，积分扣减成功");
        return true;
    }
}
```

```
public class LogisticsInfo {
    public String logInfo(){
        String log = "物流已发出";
        return log;
    }
}

```

```
public class Test {
    public static void main(String[] args) {
        GiftInfo giftInfo = new GiftInfo("《热爱生活》");
        CheckIntegral checkIntegral = new CheckIntegral();
        IntegralDec integralDec = new IntegralDec();
        LogisticsInfo logisticsInfo = new LogisticsInfo();
        if(checkIntegral.checkJf(999)){
            if(integralDec.dec(100)){
                logisticsInfo.logInfo();
            }
        }
    }
}
```
可以看到原先是这种瀑布流的模式。如果用门面模式效果会如何，来试一下。  
新建一个门面模式类：  
```
public class FacadeService {
    private CheckIntegral checkIntegral = new CheckIntegral();
    private IntegralDec integralDec = new IntegralDec();
    private LogisticsInfo logisticsInfo = new LogisticsInfo();
    public void exchange(GiftInfo giftInfo){
        if(checkIntegral.checkJf(999)){
            if(integralDec.dec(100)){
                System.out.println(logisticsInfo.logInfo());
            }
        }
    }
}
```
改造测试类：
```
public class Test {
    public static void main(String[] args) {
        GiftInfo giftInfo = new GiftInfo("《热爱生活》");
        FacadeService facadeService = new FacadeService();
        facadeService.exchange(giftInfo);
    }
}
```
执行结果：
```
积分足够
购买完成，积分扣减成功
物流已发出
```
没错，代码是一样的~就是移了下位置，但区别就是这就是设计模式，所以以后不要觉得自己没有用过设计模式，其实你一直在用！  
（参考jdk源码-JDBCUtils里的方法，调用的很多方法都不是自己的）

**门面模式的优点**  
 * 简化了调用过程，无需深入了解子系统，以防给子系统带来风险；
 * 减少了系统依赖，松散耦合；
 * 更好的划分访问层次，提高了安全性；
 * 遵循迪米特法则，即最少知道原则；
**门面模式的优点**
 * 当增加子系统和扩展子系统行为时，可能容易带来未知风险；
 * 不符合开闭原则；
 * 某些情况下可能违背单一职责原则； 
 
 
**门面模式与代理模式**
* 门面模式就是一种代理模式（静态代理）
* 门面模式重点：封装
* 静态代理模式：增强
* 不做增强的静态代理就是门面模式
包括后面的委派模式（不属于GOF 23），也是一种静态代理，代理模式是结构型模式，委派模式是行为型模式。

**门面模式与单例模式**
* 门面模式做成单例，工具包