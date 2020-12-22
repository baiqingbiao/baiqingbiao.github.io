---
layout: post
title: 详解23种设计模式之--------（11）桥接模式
description: designPatterns_11
category: blog
---

#### 桥接模式的定义
桥接模式（Bridge Pattern）也成为桥梁模式、接口（interface）模式
或柄体（handle and body）模式，是将抽象部分与它的具体实现部分分离，使他们都可以独立的变化。

通过组合的方式建立两个类之间的关系，而不是继承（来替代多种继承）。

属于结构型模式

**生活中的场景**
* 链接两个空间维度的桥梁
* 链接虚拟网络（虚拟机）和物理网络（通过桥接模式链接，虚拟网卡与物理网卡通过交换机（网络适配器）来接通）

**桥接模式的适用场景**
1.在抽象和具体实现之间需要增加更多的灵活性的场景；  
2.一个类存在两个（或两个以上）独立变化的维度，而这两个（或以上）维度都需要独立进行扩展；
3.不希望使用继承，或因为多层继承导致系统类的个数剧增；

假设现在有三个维度的course、note、video，有java和python同时实现他：
```
public interface ICourse {
}
```
```
public interface INote {
}
```
```
public interface IVideo {
}
```
```
public class JavaCourse implements ICourse{
}
```
```
public class JavaNote implements INote{
}
```
```
public class JavaVideo implements IVideo{
}
```
```
public class PythonCourse implements ICourse {
}
```
```
public class PythonNote implements INote {
}
```
```
public class PythonVideo implements IVideo {
}
```
看一下类图是这样的，三个独立的模块：
![类图](/images/designPattern/11-1.PNG)  

这时候用桥接模式来连接一下，这个桥梁的作用就是把他们组合到一起：  
增加一个连接器，让它来实现ICourse接口，并且拿到INote与IVideo；
```
@Data
public class AbstractCourse implements ICourse{
    private INote note;
    private IVideo video;
}
```
然后让JavaCourse和PythonCourse来继承：
```
public class JavaCourse extends AbstractCourse{
}
```
```
public class PythonCourse extends AbstractCourse {
}
```
那么现在这个连接器就把三个独立的模块连接了起来。

![类图](/images/designPattern/11-2.PNG)  


再看一个例子，假设平时发短信，需要把这些消息保存起来，那么这些消息又分为普通消息和加急消息，
而这个短消息和邮件都有这两种，那么可以认为他们是相互独立的，看一下如何将他们关联起来。  
Imessage接口：
```
public interface IMessage {
    void send(String content,String toUser);
}
```
邮件：
```
public class MailMessage implements IMessage {
    public void send(String content, String toUser) {
        System.out.println("发送邮件给" + toUser);
    }
}
```
短信：
```
public class SmsMessage implements IMessage {
    public void send(String content, String toUser) {
        System.out.println("发送短信给" + toUser);
    }
}
```
普通的：
```
public class Nomal {
}
```
加急的：
```
public class Urgency {
    void sendMessage(String message,String toUser){
        message = "【加急】" + message;
    }
}
```
按照之前的方式，就是让Urgency分别继承SmsMessage和MailMessage,然后分别对消息做出处理；  
但是现在用桥接模式是这样的：  
新建抽象消息类，将Imessage通过构造方法传递进来，创建发送方法调用自己：
```
public abstract class AbstractMessage {
    private IMessage message;

    public AbstractMessage(IMessage message) {
        this.message = message;
    }

    public void send(String content, String toUser) {
        this.message.sendMessage(content,toUser);
    }
}
```
加急的方法去继承：
```
public class Urgency extends AbstractMessage{
    public Urgency(IMessage message) {
        super(message);
    }

    public void send(String message, String toUser){
        message = "【加急】" + message;
        super.send(message,toUser);
    }
}
```
普通的：
```
public class Nomal extends AbstractMessage{
    public Nomal(IMessage message) {
        super(message);
    }
}
```
测试：
```
public class Test {
    public static void main(String[] args) {
        IMessage iMessage = new MailMessage();
        AbstractMessage abstractMessage = new Nomal(iMessage);
        abstractMessage.send("加班申请","孙总");

        iMessage = new SmsMessage();
        abstractMessage = new Urgency(iMessage);
        abstractMessage.send("加班申请","孙总");
    }
}
```
执行：
```
使用邮件发送加班申请给孙总
使用短信发送【加急】加班申请给孙总
```
之前的类图，两个独立模块:
![类图](/images/designPattern/11-3.PNG) 
加上链接之后：
![类图](/images/designPattern/11-4.PNG) 


（DriverManager）


**桥接模式的优点**
* 分离抽象部分及其具体实现部分；
* 提高了系统的扩展性；
* 符合合成复用原则；
* 符合开闭原则

**桥接模式缺点**
* 增加了系统的理解与设计难度
* 需要正确识别系统中两个独立变化的维度
