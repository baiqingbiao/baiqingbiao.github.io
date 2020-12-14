---
layout: post
title: 详解23种设计模式之--------（7）装饰器模式
description: designPatterns_7
category: blog
---


**装饰器模式的定义**
* 装饰器模式（Decorator Pattern）也叫做包装模式（Wrapper Pattern），是指在不改变原有对象的基础上，将功能附加到对象上，提供了比继承更有
弹性的替代方案（扩展原有对象的功能）
* 属于结构型设计模式

**生活中的装饰器模式**
* 煎饼侠（加鸡蛋...）
* 蛋糕（各种）

**装饰器模式的适用场景**
* 用于扩展一个类的功能或给一个类添加附加职责；
* 动态的给一个对象增加功能，这些功能可以动态的撤销

**抽象案例（必须属于同一继承体系）**  
首先创建一个顶层抽象类：
```
public abstract class Component {
    public abstract void operation();
}
```
然后是原始实现（可扩展）：
```
public class ConcreteComponent extends Component{//不加料（原始）
    public void operation() {
        System.out.println("处理业务逻辑");
    }
}
```
装饰器：
```
public abstract class Decorate extends Component {
    protected Component component;

    public Decorate(Component component) {
        this.component = component;
    }

    public void operation() {
        //转发请求给组件对象，可以在转发前后执行一些附加动作
        component.operation();
    }
}
```
增加功能A的类：
```
public class ConcreteDecoratorA extends Decorate {

    public ConcreteDecoratorA(Component component) {
        super(component);
    }
    private void operationFirst(){
    }
    private void operationLast(){
    }

    public void operation() {
        //调用父类的方法，在调用的前后执行一些附加动作
        operationFirst();//添加的功能
        super.operation();//调用父类的方法
        operationLast();//添加的功能
    }
}
```
增加功能B的类：
```
public class ConcreteDecoratorB extends Decorate {

    public ConcreteDecoratorB(Component component) {
        super(component);
    }
    private void operationFirst(){
    }
    private void operationLast(){
    }

    public void operation() {
        //调用父类的方法，在调用的前后执行一些附加动作
        operationFirst();//添加的功能
        super.operation();//调用父类的方法
        operationLast();//添加的功能
    }
}
```
客户端调用：
```
public class Client {
    public static void main(String[] args) {
        Component component = new ConcreteComponent();//创建需要被加工的原始对象（需要被装饰的对象）
        Decorate decorateA = new ConcreteDecoratorA(component);//给对象透明的增加功能A，并调用
        decorateA.operation();
        Decorate decorateB = new ConcreteDecoratorB(component);//给对象透明的增加功能A，并调用
        decorateB.operation();
        Decorate decorate2 = new ConcreteDecoratorB(decorateA);//装饰器也可以装饰具体的装饰对象
        decorate2.operation();
    }
}
```

md看起来还是太抽象了，接着用煎饼侠看一下：  
假设现在我要买一个煎饼，现在需要实现添加鸡蛋、香肠、芥末等等来计算出价格：  
由于装饰器模式必须属于继承体系，现在创建煎饼抽象类：  
```
public abstract  class BatterCake {
    public abstract String getMsg();
    public abstract int getPrice();
}
```
创建基础煎饼：
```
public class BaseCake extends BatterCake{
    public String getMsg() {
        return "一个煎饼";
    }

    public int getPrice() {
        return 5;
    }
}
```
装饰器：
```
public abstract class Decorator extends  BatterCake{
    protected BatterCake batterCake;

    public Decorator(BatterCake batterCake) {
        this.batterCake = batterCake;
    }

    public String getMsg() {
        return batterCake.getMsg();
    }

    public int getPrice() {
        return batterCake.getPrice();
    }
}

```
添加鸡蛋：
```
public class AddEgg extends Decorator {

    public AddEgg(BatterCake batterCake) {
        super(batterCake);
    }

    public String getMsg() {
        return super.getMsg() + "+2个鸡蛋";
    }

    public int getPrice() {
        return super.getPrice()+ 2;
    }
}
```
添加香肠：
```
public class AddSauage extends Decorator {

    public AddSauage(BatterCake batterCake) {
        super(batterCake);
    }

    public String getMsg() {
        return super.getMsg() + "+1根香肠";
    }

    public int getPrice() {
        return super.getPrice()+ 2;
    }
}
```
测试：
```
public class ClientCake {
    public static void main(String[] args) {
        BatterCake batterCake = new BaseCake();
        Decorator decorator = new AddEgg(batterCake);
        decorator = new AddSauage(decorator);
        //...随便加
        System.out.println(decorator.getMsg() + "，价格:"+decorator.getPrice() +"块");
    }
}
```
执行结果：
```
一个煎饼+2个鸡蛋+1根香肠，价格:9块
```

**装饰器模式的特点就是一层套一层，可以重复叠加与扩展对象功能**

优化系统日志为Json格式：  
原先的格式：  