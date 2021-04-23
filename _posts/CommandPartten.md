---
layout: post
title: 详解23种设计模式之--------（17）命令模式
description: designPatterns_17
category: blog
---

[回到目录](#directory)

#### 命令模式的定义
* 命令模式（Command Pattern）是对命令的封装，每一个命令都是一个操作，请求的一方发出请求要求执行一个操作
接受的一方接收到请求，并执行操作；
命令模式解耦了请求方与接收方，请求方只需执行命令，不用关心命令怎样被接收，怎样被操作以及是否被执行等等。
* 解耦命令请求与处理；
* 属于行为型模式 

**生活中的场景**
* 遥控器;
* 餐厅菜单（或扫码点）。

**命令模式的适用场景**
* 现实语义中具备”命令“操作（如命令菜单、shell指令等）;  
* 请求调用者和请求的接收者需要解耦，是的调用者和接收者不直接进行交互；
* 需要抽象出等待执行的行为，比如撤销（Undo）操作 和恢复（Redo）等操作；
* 需要支持命令宏（即命令组合操作（菜单列表，cmd，shell里的help））；

这里如果过看作 执行命令方（接收方）是一个抽象维度，调用方是一个具象维度（两个体系），也可以看作桥接模式（两个继承体系）；
但是命令模式关心的不是怎么去执行，而是怎么将命令的执行解耦。桥接模式是关心两方，命令模式关心中间链接或扩展。

用一个简单的业务场景看一下：  
假设现在通过操作播放器的按钮（暂停，播放，快进等）来操作视频的播放，
```
public class BiliBiliPlayer {
    public void play(){
        System.out.println("播放");
    }

    public void speed(){
        System.out.println("拖动进度条");
    }

    public void stop(){
        System.out.println("停止播放");
    }

    public void parse(){
        System.out.println("暂停");
    }
}
```
功能菜单抽象：
```
public interface IAction {
    void execute();
}
```
菜单列表（功能列表）：
```
public class ParseAction implements IAction {
    private BiliBiliPlayer biliBiliPlayer;

    public ParseAction(BiliBiliPlayer biliBiliPlayer) {
        this.biliBiliPlayer = biliBiliPlayer;
    }

    public void execute() {
        biliBiliPlayer.parse();
    }
}
```
```
public class PlayAction implements IAction {
    private BiliBiliPlayer biliBiliPlayer;

    public PlayAction(BiliBiliPlayer biliBiliPlayer) {
        this.biliBiliPlayer = biliBiliPlayer;
    }

    public void execute() {
        biliBiliPlayer.play();
    }
}
```
```
public class SpeedAction implements IAction {
    private BiliBiliPlayer biliBiliPlayer;

    public SpeedAction(BiliBiliPlayer biliBiliPlayer) {
        this.biliBiliPlayer = biliBiliPlayer;
    }

    public void execute() {
        biliBiliPlayer.speed();
    }
}
```
```
public class StopAction implements IAction {
    private BiliBiliPlayer biliBiliPlayer;

    public StopAction(BiliBiliPlayer biliBiliPlayer) {
        this.biliBiliPlayer = biliBiliPlayer;
    }

    public void execute() {
        biliBiliPlayer.stop();
    }
}
```
遥控器或手 发起控制：
```
public class Controller {
    public void execute(IAction action){
        action.execute();
    }
}
```
测试：
```
public class Test {
    public static void main(String[] args) {
        BiliBiliPlayer biliBiliPlayer = new BiliBiliPlayer();
        Controller controller = new Controller();
        controller.execute(new PlayAction(biliBiliPlayer));//实现解耦
    }
}

```

那么平时经常遇到批量执行的操作（套餐），这个时候只需要改造一下Controller：
```
public class Controller {
    List<IAction> actions = new ArrayList<IAction>();

    public void addAction(IAction action){
        actions.add(action);
    }
    public void execute(IAction action){
        action.execute();
    }
    public void execudeBatch(){
        for (IAction action:
        actions) {
            action.execute();
        }
        actions.clear();
    }
}
```
然后执行：
```
public class Test {
    public static void main(String[] args) {
        BiliBiliPlayer biliBiliPlayer = new BiliBiliPlayer();
        Controller controller = new Controller();

        controller.addAction(new PlayAction(biliBiliPlayer));
        controller.addAction(new StopAction(biliBiliPlayer));
        controller.addAction(new SpeedAction(biliBiliPlayer));

        controller.execudeBatch();
    }
}
```
就可以了。这个命令当然也要记录下来，用于回退等。日常开发并不常见。  
源码应用（Runable接口（run方法去抢cpu资源），Thread（实现了Runable接口），Junit.framework.Test）

**命令模式的优点**
* 通过引入中间件（抽象接口），解耦了命令请求与实现；
* 扩展性良好，可以很容易新增加命令；
* 支持组合命令、命令队列；
* 可以在现有命令上增加额外功能（比如日志记录，结合装饰器模式更舒服）；

**命令模式缺点**
* 具体命令类会更多。
* 增加了程序复杂度，可能会理解困难。

