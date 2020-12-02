---
layout: post
title: 详解23种设计模式之--------（3）原型模式
description: designPatterns_3
category: blog
---


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
D:\igix_soft\jdk8u\jdk8u232-b09\bin\java.exe "-javaagent:D:\igix_soft\idea2018\IntelliJ IDEA 2018.1.4\lib\idea_rt.jar=57303:D:\igix_soft\idea2018\IntelliJ IDEA 2018.1.4\bin" -Dfile.encoding=UTF-8 -classpath D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\charsets.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\ext\access-bridge-64.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\ext\cldrdata.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\ext\dnsns.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\ext\jaccess.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\ext\localedata.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\ext\nashorn.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\ext\sunec.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\ext\sunjce_provider.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\ext\sunmscapi.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\ext\sunpkcs11.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\ext\zipfs.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\jce.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\jsse.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\management-agent.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\resources.jar;D:\igix_soft\jdk8u\jdk8u232-b09\jre\lib\rt.jar;D:\pakage\网上商城\20201127\out\production\20201127 protoTypePattern.Client
ConcreteProtoyype{age=18, name='huyawen'}
ConcreteProtoyype{age=18, name='huyawen'}
```
