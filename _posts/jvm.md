---
layout: post
title: JVM
description: 都说世界上没有最好的编程语言，我认为java就是
category: blog
---

[热门语言排行](https://www.tiobe.com/tiobe-index/)

重大事件：  
* 1995年5月23日，Java语言诞生；
* 1998年2月，JDK1.1被下载20,000,000次。
   
[jvm文档-jdk8](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-1.html#jvms-1.2)  

[类加载](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html)

转换.class文件利于阅读:java -p -v \输入.class > 输出.txt

类加载:把类加载到jvm中去,如果jvm想要把数据加载进来,那么jvm一定是把数据加载到内存中,那么想要访问这些数据肯定需要一个访问入口或媒介,才能从内存中拿到数据,而数据肯定是从.class文件中获得中获得的,毋庸置疑,数据访问的入口肯定是一个对象,因为java是面向对象的

![1](/images/jvm/1-1.png)  

而其被称作虚拟机,必定与物理计算机数据加载过程一致.

![2](/images/jvm/1-2.png)

类加载过程:  
![3](/images/jvm/1-3.png)  

![4](/images/jvm/1-4.png) 

**装载**:
要想加载字节码文件到内存当中,必须要先找到这个字节码文件,而且加载这个文件必须是一个流文件,所以第一步字节码文件要先变成一个字节流,然后可以通过他的名字找; 
装载.Class文件>>字节流,通过全限定名获取二进制字节流,(java中有这样的一个代码模块,来通过全限定名寻找这个二进制流,而且这个动作是放在虚拟机外部实现的(应用程序要决定如何让获取需要实现的类),实现这个代码的模块叫类加载器)

将数据放入内存,将字节流代表的静态储存结构放入方法区,变成一个运行储存结构,因为要run起来了.  
同时在内存当中生成数据访问入口(生成class对象代表数据访问入口).   
实际上在装载阶段完成之后,方法区里面就有类信息/静态变量/常量了,但是即使编译过后的热点代码不在这个阶段进入方法区.包括堆中只有代表这个类的一个Class对象.  
装载阶段:最重要的一步,只能在这个阶段才能对它进行操控.  
1.对类加载器进行操作  
2.自定义类加载器操作/做字节码增强操作(Javaagent)  

加载字节码文件的方式:  
   1. 从本地系统中直接加载;  
   2. 通过网络下载.class文件;  
   3. 从归档文件中加载.class文件(jar包,war包);  
   4. 从专有数据库中提取.class文件(jsp);  
   5. 将java源文件动态编译为.class文件,也就是运行时计算而成(动态代理);  
   6. 从加密文件中获取;  
   
**链接**:  
   1.验证:1验证字节码文件必须符合要求,2不能危害虚拟机的安全  
* 文件格式验证:主要目的是保证输入的字节流能够正确的解析并储存在方法区之内(所以发生在进入方法区之前).
   * 如是否以16进制的cafebaby开头
   * 版本号是否正确
* 元数据验证:进行语义校验,java语法等,不符合规范的不能进入方法区,或者进去了要识别出来
   * 是否有父类
   * 是否继承了final类,继承类抽象类,没实现抽象方法等
* 字节码验证:校验对虚拟机的安全危害,并不一定语法错误
   * 运行检查
   * 栈数据类型和操作码参数是否吻合(超出栈空间)
* 符号引用验证:最后一步验证,发生在解析阶段
   * 常量池中描述类是否存在
   * 访问的方法或者字段是否存在且有足够的权限(保证可以执行解析操作)  
   2.准备:为类变量分配内存(到方法区)并为类变量**设置默认初始值**如int类型为0,引用类型为null;但是不会为实例变量分配(不带static修饰),实例变量会随对象一起分配到java堆.  
```
private static int a = 1;  
```
准备阶段 a = 0;  
```
private final static int a = 1;  
```
准备阶段 a = 1;  
初始化为ConstantValue所指向的值(编译后的初始化值),可以理解为在编译器就将结果放入了调用它的常量池当中;  
ConstantValue:通知虚拟机自动为final static属性赋值,而且只有final static修饰的时候才会触发ConstantValue;  
局限于基本数据类型和String,因为在常量池中只能引用到基本数据类型以及String.  
![符号引用](/images/jvm/符号引用.PNG)

解析：类中的符号引用(文件中的记录值)转换为直接引用(真正在内存中开辟并记录地址),把文件中的信息在物理内存中落地,
除了动态 invockdynamic(jdk1.1后首次引入,jdk7引入,jdk8投入使用) 以外,会对任意的解析结果做缓存,也就是仅保留第一次解析的结果;

**初始化**:Classinit,**给变量设置程序员想要赋予的值**,指令类变量,静态代码块;
JVM按需加载,先进行装载,链接;若有父类,先初始化父类等;  
问题:初始化什么时候被触发?
答:主动使用到类的时候.

**使用**:
* **主动引用类**的时候，会对类进行初始化
   1. 创建类的实例，也就是new的方式;
   2. 访问某个类或接口的静态变量，或者对给静态变量赋值;
   3. 调用类的静态方法;
   4. 反射(如Class.forName("com.test.Test"));
   5. 初始化某个类的子类,其父类也会被初始化;
   6. Java虚拟机启动时被标明为启动类的类('JvmCaseApplication'),直接使用'java.exe'来运行某个主类
* 不经意当中，可能会用到其他的类，但是不会对类进行初始化，这种叫做**被动引用**
   1. 通过子类调用父类的静态变量，不会引起子类的初始化；
   2. 使用类的final常量不会引起类的初始化；
   3. 使用类的数组，不会引起类的初始化；  
   
```
public class Demo {
    public static void main(String[] args) {
        //String ta = SubInitClass.a;//通过子类引用父类的静态字段，只会引起父类的初始化，不会引起子类的初始化
        //String tb = InitClass.b;//使用类的final常量不会引起类的初始化
        //SubInitClass[] subInitClasses = new SubInitClass[10];//定义数组不会引起类的初始化
        //SubInitClass.a = "hi";// 给静态变量赋值,会初始化类
        //InitClass.method();//调用静态方法,会初始化类
        //SubInitClass subInitClass = new SubInitClass();//初始化子类,父类也会被初始化

        /*String ta = SubInitClass.a;
        ta.getClass().getClassLoader();//反射会初始化类*/

}
}

class InitClass{
    static{
        System.out.println("初始化InitClass");
    }
    public static String a = "hello";
    public final static String b = "b";
    public static int c = 1;
    public static void method(){}
}
class SubInitClass extends InitClass{
    static{
        System.out.println("初始化SubInitClass");
    }
}
```

**卸载**:
   1. 该类所有的实例都已经被回收，也就是java堆中不存在任何该类的实例；
   2. 该类的ClassLoader已经被回收;
   3. 该类的java.lang.Class(装载阶段生成)对象没有在任何地方引用,无法在任何地方通过反射调用该方法;
   
   ![类加载器](/images/jvm/加载器.PNG)
   
   什么是类加载器？
   1. java中有这样的一个代码模块,来通过全限定名寻找这个二进制流,而且这个动作是放在虚拟机外部实现的(应用程序要决定如何让获取需要实现的类)
   2. 确定类在虚拟机中的唯一性；
   
* 启动类加载器(Bootstrap ClassLoader)：负责将存放在\lib目录中的，或被-Xbootclasspath参数指定的路径中的，  
并且是虚拟机识别的类库加载到虚拟机中。如rt.jar。名字不符合即使放在目录中也不被加载。  
如果需要把加载请求委派给引导类加载器，直接使用null代替即可。  
**本身是C层面的，目前java中看不到**

* 扩展类加载器(Extension ClassLoader)：由sum.misc.Launcher$ExtClassLoader实现，负责加载<Java_Home>\lib\ext目录中的，  
或者被java.ext.dir系统变量所指定的路径中的所有类库。开发者可以直接使用扩展类加载器。

* 应用程序类加载器(Application ClassLoader)：由sun.misc.Launcher$App-ClassLoader实现。  
是ClassLoader中的getSystemClassLoader()方法的返回值，所以也称为**系统类加载器**。  
负责加载用户路径(ClassPath)上所指定的类库，如果应用程序中没有自定义过自己的类加载器，这个就是默认的加载器，开发人员可以直接使用这个类加载器。


```
public class Demo1 {
    public static void main(String[] args) {
        //App ClassLoader
        System.out.println(new Worker().getClass().getClassLoader());
        //Ext ClassLoader
        System.out.println(new Worker().getClass().getClassLoader().getParent());
        //Bootstrap ClassLoader
        System.out.println(new Worker().getClass().getClassLoader().getParent().getParent());
        System.out.println(new String().getClass().getClassLoader());

//        sun.misc.Launcher$AppClassLoader@18b4aac2
//        sun.misc.Launcher$ExtClassLoader@42a57993
//        null
//        null
    }
}
```

**类加载机制**：
   1. 全盘负责(当前类加载器机制)：当一个类加载器负责加载某个Class的时候，他所依赖和引入的所有的Class都由当前的这个类加载器负责载入,除非new一个Bootstarap.loadClass去显示的进行加载;  
   但是请注意,这个步骤仅仅仅仅只是调用了一个loadClass方法,但是真正加载字节码文件生成Class对象的过程不由它负责,还要往上找(父类委托);
   2. 父类委托(双亲委派机制):永远只找到最顶层的类加载器,只有当父类加载器找不到字节码文件时才会从自己的类路径中找到并装载目标类,
   首先判断顶层类是否被加载了,如果被加载了那么剩下的就不用再加载了,因为只需要加载级别高的一个;  
   **Java推荐机制**,可以通过继承他的ClassLoader去实现自己的类加载器;
   3. 缓存机制:左右被加载的Class都在内存中进行缓存,当程序需要用某个Class的时候,就去缓存区先寻找是否有Class对象,没有才会去读类的二进制文件进行加载成Class对象;
   修改了Class后必须重启JVM,对于一个类加载器的实例来说,相同的全名永远只加载一次;  
   jdk8用的直接内存(原空间),类变量只被初始化一次;  
   **ClassLoader.loadClass()**,类缓存核心方法:findLoadClass()方法,如果都没有,就调用findClass方法,所以重写类加载器的时候只需要重写**findClass()**方法,并不需要整体重写;  
   resolve():**链接操作**,对文件格式以及字节码进行验证操作,为static开辟初始空间,符号引用转为直接引用,访问控制与方法覆盖;
   
   问题:双亲委派


#### 1.运行时数据区
#### 2.类加载机制
#### 3.字节码执行机制
#### 4.GC
#### 5.JVM内存监控与故障定位
#### 6.JVM调优