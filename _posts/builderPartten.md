---
layout: post
title: 详解23种设计模式之--------（4）建造者模式
description: designPatterns_4
category: blog
---
[回到目录](#directory)

#### 建造者模式的定义
  * 建造者模式（Builder Partten），是将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示；
  * 特征：用户只需指定需要创建的类型就可以获得对象，建造过程及细节不用去了解；
  * 属于创建型模式
#### 建造者模式的适用场景
  * 适用于创建对象需要很多步骤，但是步骤的顺序不一定；
  * 如果一个对象有非常复杂的内部结构（很多属性）；
  * 把复杂对象的创建和使用分离；
常见的一个场景就是拼接sql，将sql按照同条件或同类别来进行一个封装；
接下来通过一个例子详细了解下一下，假设现在要做一些课程，需要写ppt，录制视频，做作业等，那么如何来通过建造者来实现呢？
首先，创建一个课程对象：
```
@Data
public class Course {
    private String name;
    private String ppt;
    private String video;
    private String note;
    private String homework;
    @Override
    public String toString() {
        return "Course{" +
                "name='" + name + '\'' +
                ", ppt='" + ppt + '\'' +
                ", video='" + video + '\'' +
                ", note='" + note + '\'' +
                ", homework='" + homework + '\'' +
                '}';
    }
}
```

然后，抽象出一个建造者出来：

```
public interface ICourseBuilder {
    Course build();
}
```

实现它的build方法，并创建一些赋值方法：

```
public class CourseBuilder implements ICourseBuilder{
    private Course course = new Course();
    public Course build() {
        return course;
    }
    public void addName(String name){
        course.setName(name);
    }
    public void addVideo(String video){
        course.setVideo(video);
    }
    public void addPpt(String ppt){
        course.setPpt(ppt);
    }
    public void addNote(String note){
        course.setNote(note);
    }
    public void addHomework(String homework){
        course.setHomework(homework);
    }
}
```

好了那么创建完之后，编写测试类：

```
public class Test {
    public static void main(String[] args) {
        CourseBuilder courseBuilder = new CourseBuilder();
        courseBuilder.addHomework("写代码");
        courseBuilder.addPpt("演讲");
        courseBuilder.addVideo("看视频");
        System.out.println( courseBuilder.build());
    }
}
```

执行结果：

```
Course{name='null', ppt='演讲', video='看视频', note='null', homework='写代码'}
```

其实呢，这种建造者模式并不常见，因为我们在连续的操作courseBuilder，如果这个变量很多，写起来就会非常麻烦，所以这里是一个优化的点。
那么优化写法就是链式写法，稍微改造一下，赋值方法完成之后返回this：

```
public class CourseBuilder implements ICourseBuilder {
    private Course course = new Course();

    public Course build() {
        return course;
    }
    public CourseBuilder addName(String name){
        course.setName(name);
        return this;
    }
    public CourseBuilder addVideo(String video){
        course.setVideo(video);
        return this;
    }
    public CourseBuilder addPpt(String ppt){
        course.setPpt(ppt);
        return this;
    }
    public CourseBuilder addNote(String note){
        course.setNote(note);
        return this;
    }
    public CourseBuilder addHomework(String homework){
        course.setHomework(homework);
        return this;
    }
}
```

改造测试类：

```
public class Test {
    public static void main(String[] args) {
        CourseBuilder courseBuilder = new CourseBuilder()
        .addHomework("写代码")
        .addPpt("演讲")
        .addVideo("看视频");
        System.out.println( courseBuilder.build());
    }
}
```

在建造者模式中，链式编程应用的非常广泛，非常巧妙。自始至终都在操作这个courseBuilder，那么就在每一步完成之后将courseBuilder返回；
