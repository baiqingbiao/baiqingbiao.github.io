---
layout: post
title: 详解23种设计模式之--------（12）委派模式
description: designPatterns_12
---

[回到目录](./designPatterns#directory)

#### 委派模式的定义
* 委派模式（Delegate Pattern）也成为委托模式，他的基本作用就是负责任务的调度和任务分配，将任务的分配和
执行分离开来，可以看作是一种特殊情况下的静态代理的全权代理；
* 不属于GOF 23种设计模式之一
* 属于行为型模式


**生活中的场景**
* 老板给员工下达命令，让专门在这个领域的人员完成任务。
* 授权委托书

**委派模式的适用场景**
1.委派对象本身不知道如何处理一个任务（或一个请求），把请求交给其他对象来处理；  
2.实现程序的解耦  

委派模式不需要知道由谁来做（a或b），只需要找到委派对象拿到（a或b）的执行结果，从而实现程序的解耦；

通过代码实现一个具体的业务场景。以员工和老板场景为例，中间有部门主管或项目经理（Leader），老板下达命令给leader
，leader通过分析谁可以做来决定交给谁做：  
首先创建员工抽象类，让leader和员工都去实现：
```
public interface IEmployee {
    void doing(String task);
}
```
```
public class EmployeeA implements IEmployee {
    protected String goodat = "编程";
    public void doing(String task) {
        System.out.println("我是员工A，我擅长"+goodat+"，现在开始做"+task+"工作。");
    }
}
```
```
public class EmployeeB implements IEmployee{
    protected String goodat = "平面设计";
    public void doing(String task) {
        System.out.println("我是员工B，我擅长"+goodat+"，现在开始做"+task+"工作。");
    }
}
```
leader下的员工技能都放起来；
```
public class Leader implements IEmployee{
    private Map<String,IEmployee> employeeMap = new HashMap<String, IEmployee>();

    public Leader() {
        employeeMap.put("爬虫",new EmployeeA());
        employeeMap.put("海报",new EmployeeB());
    }

    public void doing(String task) {
        if(employeeMap.containsKey(task)){
            employeeMap.get(task).doing(task);
        }else {
            System.out.println("不好意思做不了，另请高明。");
        }
    }
}
```
老板直接下达命令，这里传递一个leader：

```
public class Boss {
    public void command(String task,Leader leader){
        leader.doing(task);
    }
}
```
执行结果
```
我是员工A，我擅长编程，现在开始做爬虫工作。
我是员工B，我擅长平面设计，现在开始做海报工作。
不好意思做不了，另请高明。
```
![类图](/images/designPattern/12-1.PNG)  

在源码中的应用：
* JVM类加载机制（双亲委派机制：一个类会把请求委托给自己的父类执行，
如果父类执行返回成功，自己就不再执行了，如果执行不了，则自己执行） （ClassLoader.loadClass()）
* Method(java.lang).invoke
* SpringIoc(BeanDefinition)
* DispatcherServlet

试一下DispatcherServlet是怎么实现的，
```
public class MemberController {
    public void getMemberById(String mid){

    }
}
```
看一下如何实现调度：
```
public class DispatcherServlet extends HttpServlet {

    private Map<String,Method> handlerMapping = new HashMap<String,Method>();//初始化调用init放入方法
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doDispatcher(req,resp);
    }

    private void doDispatcher(HttpServletRequest req, HttpServletResponse resp) {
        String url = req.getRequestURI();
        Method method = handlerMapping.get(url);//获得方法
        //method.invoke();//反射获取类来调度
    }

    @Override
    public void init() throws ServletException {
        try {
            handlerMapping.put("/web/getMemberById.json",MemberController.class.getMethod("getMemberById",new Class[]{String.class}));
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }
    }
}

```


**委派模式的优点**
* 通过任务委派能够将一个大型的任务细化，然后通过统一管理这些子任务的完成情况实现任务的跟进，能够加快任务执行效率。
**委派模式缺点**
* 任务委派方式需要需要根据任务的复杂程度进行不同的改变，在任务比较复杂得情况下可能需要进行多重委派，容易造成紊乱。

**委派模式和代理模式**
* 委派模式是行为型模式，代理模式是结构型模式
* 委派模式注重的是任务派遣，注重结果
* 代理模式注重的是代码增强，注重过程
* 委派模式是一种特殊的静态代理，相当于全权代理

**委派模式和组合模式**
