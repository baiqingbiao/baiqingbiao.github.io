---
layout: post
title: 详解23种设计模式之--------（9）组合模式
description: designPatterns_9
---

[回到目录](./designPatterns#directory)

#### 组合模式的定义
组合模式（Composite Pattern）非常常见的一种设计模式，又称为轻量级模式，也成为整体-部分模式（Part-Whole），
它的宗旨是通过将单个对象（叶子节点）和组合对象（树枝节点）用相同的接口进行表示。  
作用：是客户端对单个对象和组合对象保持一致的方式处理。  
属于结构型模式。

**组合和聚合**
人在一起心在一起是一个团队，只有人在一起是一个团伙，而这个团队是一个组合，团伙是一个聚合；  
CEO不可能直接发号施令给员工，而是组建一系列的部门，这个组合架构共同形成一家公司，这是组合；  
人的身体组成比如头和心脏，谁离开谁也不行或者说不再完整，这样的树形结构，是组合模式；  
一个老师可以教很多学生，一个学生又拥有很多老师，这是聚合；  
如果恋人之间有一个是绿茶或者海王，这也是一个聚合；  
组合具有相同的主线 和生命周期；

**组合模式的适用场景**
1.希望客户端可以忽略对象与单个对象的差异时；  
2.对象层次具备整体和部分，呈树形结构（如树形菜单，操作系统目录结构，公司组织架构等）

**组合模式有透明写法和安全写法两种，接下来用一个案例来实现这两种写法**

假设现在某一专业的课程体系(实现课程的添加，删除，获取名称，打印，获取价格等操作
，其中Course为具体的课程类（叶子节点），CoursePakage为某个根节点) 
** 透明写法（用就去覆盖，不然调用会抛异常，有点违背最少知道原则）  
抽象课程组件根节点（可以是接口/抽象类）：
```
public abstract class CourseComponet {
    public void addChild(CourseComponet componet){
        throw new UnsupportedOperationException("不支持添加操作");
    }
    public void delChild(CourseComponet componet){
        throw new UnsupportedOperationException("不支持删除操作");
    }
    public String getName(CourseComponet componet){
        throw new UnsupportedOperationException("不支持获取名称操作");
    }
    public void print(){
        throw new UnsupportedOperationException("不支持打印操作");
    }
    public Double getPrice(CourseComponet componet){
        throw new UnsupportedOperationException("不支持获取价格操作");
    }

}

```
叶子节点（具体的课程）：
```
@Data
public class Course extends CourseComponet {
    private String name;
    private double price;

    public Course(String name, double price) {
        this.name = name;
        this.price = price;
    }

    @Override
    public String getName(CourseComponet componet) {
        return this.name;
    }

    @Override
    public Double getPrice(CourseComponet componet) {
        return this.price;
    }

    @Override
    public void print() {
        System.out.println(name + "(￥ "+price+")");
    }
}
```
课程包（如果不用组合模式（private List<Course> list = new ArrayList<Course>();），现在用CourseComponet，
不管有多少层级，后面也都是它）：
```
public class CoursePakage extends CourseComponet {
    private List<CourseComponet> list = new ArrayList<CourseComponet>();

    private String name;
    private Integer level;

    public CoursePakage(String name, Integer level) {
        this.name = name;
        this.level = level;
    }

    @Override
    public void addChild(CourseComponet componet) {
        list.add(componet);
    }

    @Override
    public void delChild(CourseComponet componet) {
        list.remove(componet);
    }

    @Override
    public String getName(CourseComponet componet) {
        return this.getName(componet);
    }

    @Override
    public void print() {
        System.out.println(this.name);
        for (CourseComponet c : list
             ) {
            if(this.level != null){
                for (int i = 0; i < this.level; i ++){
                    System.out.print("   ");
                }
                for (int i = 0; i < this.level; i++){
                    if(i == 0){
                        System.out.print("+");
                    }
                    System.out.print("-");
                }

            }
            c.print();
        }
    }
}
```
客户端调用:
```
    public static void main(String[] args) {
        System.out.println("透明的组合模式");

        CourseComponet javaBase = new Course("java基础课程",8000);
        CourseComponet python = new Course("python课程",5000);

        CourseComponet coursePakage = new CoursePakage("java架构师课程",2);

        CourseComponet designPattern = new Course("设计模式课程",1500);
        CourseComponet source = new Course("源码分析",4000);
        CourseComponet softSkill = new Course("软件技能",5000);
		
		 //python.addChild(designPattern); //如果python把设计模式加进去会抛异常,所以这里不符合最少知道原则

        coursePakage.addChild(designPattern);
        coursePakage.addChild(source);
        coursePakage.addChild(softSkill);

        CourseComponet catalog = new CoursePakage("专业课程目录",1);
        catalog.addChild(javaBase);
        catalog.addChild(python);
        catalog.addChild(coursePakage);

        catalog.print();

    }

}
```
输出：
```
专业课程目录
   +-java基础课程(￥ 8000.0)
   +-python课程(￥ 5000.0)
   +-java架构师课程
      +--设计模式课程(￥ 1500.0)
      +--源码分析(￥ 4000.0)
      +--软件技能(￥ 5000.0)
```

为了解决最少知道原则问题，使用安全模式写法：  
假设现在设计一个文件系统，有目录层级，文件夹和文件，首先建立一个根节点目录：
```
public abstract class Directory {
    protected String name;
    public Directory(String name) {
        this.name = name;
    }
    public abstract void show();
}
```
在创建一个文件夹类继承：
```
public class Folder extends Directory{
    List<Directory> dirs;//创建一个存放文件的容器,

    private Integer lever;//增加一个层级关系

    public void show() {
        System.out.println(this.name);
        for (Directory c : dirs
                ) {
            if(this.lever != null){
                for (int i = 0; i < this.lever; i ++){
                    System.out.print("   ");
                }
                for (int i = 0; i < this.lever; i++){
                    if(i == 0){
                        System.out.print("+");
                    }
                    System.out.print("-");
                }

            }
            c.show();
        }
    }

    public Folder(String name,Integer lever) {
        super(name);
        this.lever = lever;
        this.dirs = new ArrayList<Directory>();
    }

    public boolean add(Directory directory){
        return this.dirs.add(directory);
    }

    public boolean del(Directory directory){
        return this.dirs.remove(directory);
    }

    public Directory get(int index){
        return this.dirs.get(index);
    }
    
    public void list(){
        for (Directory dir: dirs) {
            System.out.println(dir.name);
        }
    }
}
```
在创建一个文件类继承（叶子节点）:
```
public class File extends Directory{

    public File(String name) {
        super(name);
    }

    public void show() {
        System.out.println(this.name);
    }
}
```
测试：
```
public class ClientTest {
    public static void main(String[] args) {
        System.out.println("===============安全模式文件目录结构=============");
        Folder folder = new Folder("E：/",1);
        File file = new File("PUBG");
        File file1 = new File("CS:GO");
        File file2 = new File("食人的大鹫");
        File file3 = new File("古墓丽影");
        File file4 = new File("刺客信条");

        File file5 = new File("无名文件.lk");

        Folder folder2 = new Folder("Steam游戏目录",2);
        Folder folder3 = new Folder("PS游戏",2);
        Folder folder4 = new Folder("其他文件夹",2);

        folder.add(folder2);
        folder.add(folder3);
        folder.add(folder4);
        folder.add(file5);

        folder2.add(file);
        folder2.add(file1);
        folder2.add(file3);
        folder2.add(file4);

        folder3.add(file2);

        System.out.println("===========show()=============");
        folder.show();
        System.out.println("===========list()=============");
        folder.list();
    }
}
```
输出：
```
E：/
   +-Steam游戏目录
      +--PUBG
      +--CS:GO
      +--古墓丽影
      +--刺客信条
   +-PS游戏
      +--食人的大鹫
   +-其他文件夹
   +-无名文件.lk
===========list()=============
Steam游戏目录
PS游戏
其他文件夹
无名文件.lk
```

这里File是调用不了show方法的，所以会符合最少知道原则。  
主要的是实现了层级关系，若去哪找以前的一种方式：只能是List<List<List<>List<Directory...这样下去>>> dirs。

源码应用：Map的puttAll方法（传递一个Map），ArratList的addAll（传递一个Collection），MyBatis中的SqlNode：  
![类图](/images/designPattern/9-1.PNG)  
直接调用SqlNode的apply方法就把这些节点的方法组合到一起了。  

**透明和安全模式有啥区别，咋没get到**  

组合模式一定是树形结构吗？  
不管你是整体还是个体，都要想办法用同一种方法来操作

**组合模式的优点**
* 清楚的定义分层次的复杂的对象，表示对象的全部或部分层次；
* 让客户端忽略了层次的差异，方便对整个层次结构进行控制；
* 简化客户端代码；
* 符合开闭原则

透明模式是将某一个公共接口封装到某一个抽象节点中，系统所有节点都具有一致性为，当系统绝大多数层次都具备相同行为时，就采用透明模式；  
如果说各系统各层次差异性比较大，整个相对稳定修改比较少，就采用安全组合模式。

**组合模式缺点**
* 限制类型时会比较复杂（使用的都是同一种类型）
* 使设计变得更加抽象（是一种更高维度的设计）
