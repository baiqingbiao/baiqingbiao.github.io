---
layout: post
title: 详解23种设计模式之--------（8）享元模式
description: designPatterns_8
category: blog
---

#### 享元模式的定义
享元模式（Flyweight Pattern）又称为轻量级模式，是对象池的一种实现。类似于线程池，线程池可以避免不停的
创建和销毁多个对象，消耗性能；提供了减少对象数量从而改善应用所需的对象结构的方式，消耗性能。

宗旨：共享细粒度对象，将多个对同一对象的访问集中起来。

属于结构型模式。

**生活中的场景**
* 各类房源共享（多个平台都有相同的房源信息）
* 全国社保联网（之前的情况是各城市相互隔离）等

**享元模式的适用场景**
* 常常应用于系统底层的开发，以便解决系统的性能问题。
* 系统有大量相似对象、需要缓冲池的场景。

假假设我现在有500个人同时查询某几个车次的票据信息，那么这个查询的重复率是非常高的
这时候就能用到享元模式：  
新建一个查询信息的接口：
```
public interface ITicket {
    public void showInfo(String bunk);
}
```
实现查询方法:
```
public class TrainTicket implements ITicket {
    String from;
    String to;
    int price;

    public TrainTicket(String from, String to) {
        this.from = from;
        this.to = to;
    }

    public void showInfo(String bunk) {
        this.price = new Random().nextInt(500);
        System.out.println(from + "->" +to +" price: " + price);
    }
}
```
票据工厂(创建缓存池)：  
```
public class TicketFactory {
    public static Map<String,ITicket> pool = new ConcurrentHashMap<String, ITicket>();
    public static ITicket queryIticket(String from,String to){
        String key = from + "->" + to;
        if(pool.containsKey(key)){
            System.out.println("缓存取值");
            return pool.get(key);
        }
        System.out.println("第一次查询,创建对象");
        TrainTicket trainTicket = new TrainTicket(from,to);
        pool.put(key,trainTicket);
        return trainTicket;
    }
}
```
调用查询：
```
    public static void main(String[] args) {
        ITicket iTicket = TicketFactory.queryIticket("虹桥","贡嘎");
        iTicket.showInfo("硬座");
        iTicket = TicketFactory.queryIticket("虹桥","贡嘎");
        iTicket.showInfo("软座");

        iTicket = TicketFactory.queryIticket("虹桥","瑞士");
        iTicket.showInfo("沙发");
    }
}
```
执行结果：
```
第一次查询,创建对象
虹桥->贡嘎 price: 293
缓存取值
虹桥->贡嘎 price: 77
第一次查询,创建对象
虹桥->瑞士 price: 391
```


![类图](https://github.com/baiqingbiao/baiqingbiao.github.io/blob/master/images/designPattern/8-1.PNG)  

享元模式重点在于他的结构，而不是创建对象；实际上与注册单例类似，但不同的是，
注册单例key为class，从key上就决定了只能创建一个对象，而享元模式key为任意对象或类型
所以可以被创建多次.

模拟连接池：

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1608046096795&di=3ac18b34307b70242b773031313250a6&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201810%2F07%2F20181007110827_wqdzq.thumb.400_0.jpg)  