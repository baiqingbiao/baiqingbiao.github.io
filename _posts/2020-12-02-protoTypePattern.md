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
ConcreteProtoyype{age=18, name='huyawen'}
ConcreteProtoyype{age=18, name='huyawen'}
```

##### 原型模式的适用场景：
 * 类初始化消耗资源过多；
 * new 产生一个对象的过程非常繁琐的过程（数据准备，访问权限等）；
 * 构造函数比较复杂；
 * 循环体中创建多个对象时；
 
 其中原型模式又分为浅克隆和深克隆：
##### 1. 浅克隆：实现Cloneable接口：
 
   
```
@Data
public class ConcreteProtoyype implements Cloneable{
    private int age;
    private String name;
    private List<String> hobbies;

    @Override
    public ConcreteProtoyype clone() {
        try {
            return (ConcreteProtoyype)super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }

    @Override
    public String toString() {
        return "ConcreteProtoyype{" +
                "age=" + age +
                ", name='" + name + '\'' +
                ", hobbies=" + hobbies +
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
        List<String> hobbies = new ArrayList<String>();
        hobbies.add("吃");
        hobbies.add("睡");
        hobbies.add("拉屎");
        concreteProtoyype.setHobbies(hobbies);
        System.out.println(concreteProtoyype);

        //拷贝原型对象
        ConcreteProtoyype concreteProtoyype1  = concreteProtoyype.clone();
        concreteProtoyype1.getHobbies().add("学习");
        System.out.println(concreteProtoyype1);
    }
}
```
这时候的输出：
```
ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎]}
ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎, 学习]}

```
未发现异常，拷贝原型对象之后再看一下原型对象有没有发生变化：
```
public class Client {
    public static void main(String[] args) {
        //创建原型对象
        ConcreteProtoyype concreteProtoyype = new ConcreteProtoyype();
        concreteProtoyype.setAge(18);
        concreteProtoyype.setName("huyawen");
        List<String> hobbies = new ArrayList<String>();
        hobbies.add("吃");
        hobbies.add("睡");
        hobbies.add("拉屎");
        concreteProtoyype.setHobbies(hobbies);

        //拷贝原型对象
        ConcreteProtoyype concreteProtoyype1  = concreteProtoyype.clone();
        concreteProtoyype1.getHobbies().add("学习");
        System.out.println("原型对象："+concreteProtoyype);
        System.out.println("克隆对象："+concreteProtoyype1);
        System.out.println(concreteProtoyype == concreteProtoyype1);

        System.out.println("原型对象的爱好：" + concreteProtoyype.getHobbies());
        System.out.println("克隆对象的爱好：" + concreteProtoyype1.getHobbies());
        System.out.println(concreteProtoyype.getHobbies() == concreteProtoyype1.getHobbies());
    }
}
```
执行结果：
```
原型对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎, 学习]}
克隆对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎, 学习]}
false
原型对象的爱好：[吃, 睡, 拉屎, 学习]
克隆对象的爱好：[吃, 睡, 拉屎, 学习]
true
```
这时候发现原型对象发生了变化，但是仔细观察，克隆对象和原型对象并不是一个对象，那么为什么会随着变化呢?  
接着看对象的爱好地址是相同的，这时候就说明浅克隆的一个问题：对基本数据类型或String类型等类型可以使用，对引用型数据类型拷贝的是对象地址，而不是具体的元素。
这时候就要应用到了：  
##### 2. 深克隆
  通过序列化和反序列化实现的方式，遵循里氏替换原则；  
  增加方法（记得实现Serializable接口，否则会抛出序列化异常）：
```
    public ConcreteProtoyype deepClone() {
        try{
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(bos);
            oos.writeObject(this);

            ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bis);
            return (ConcreteProtoyype)ois.readObject();
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }
    }
```
客户端直接调用：
```
public class Client {
    public static void main(String[] args) {
        //创建原型对象
        ConcreteProtoyype concreteProtoyype = new ConcreteProtoyype();
        concreteProtoyype.setAge(18);
        concreteProtoyype.setName("huyawen");
        List<String> hobbies = new ArrayList<String>();
        hobbies.add("吃");
        hobbies.add("睡");
        hobbies.add("拉屎");
        concreteProtoyype.setHobbies(hobbies);

        //拷贝原型对象
        ConcreteProtoyype concreteProtoyype1  = concreteProtoyype.deepClone();
        concreteProtoyype1.getHobbies().add("学习");
        System.out.println("原型对象："+concreteProtoyype);
        System.out.println("克隆对象："+concreteProtoyype1);
        System.out.println(concreteProtoyype == concreteProtoyype1);

        System.out.println("原型对象的爱好：" + concreteProtoyype.getHobbies());
        System.out.println("克隆对象的爱好：" + concreteProtoyype1.getHobbies());
        System.out.println(concreteProtoyype.getHobbies() == concreteProtoyype1.getHobbies());
    }
}
```
打印结果：
```
原型对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎]}
克隆对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎, 学习]}
false
原型对象的爱好：[吃, 睡, 拉屎]
克隆对象的爱好：[吃, 睡, 拉屎, 学习]
false
```
可以看到这次符合了期望值，克隆对象的引用类型发生了变化，而原型对象还是原来那个。  
更好的实现方式：Json字符串反序列化方式。  
引发的问题：  
    1. 性能问题、占用I/O、安全性；
    2. 破坏单例：
例子：创建饿汉式单例：
```
public static ConcreteProtoyype instance = new ConcreteProtoyype();
    private ConcreteProtoyype(){}

    public static ConcreteProtoyype getInstance(){
        return instance;
    }
```
调用：
```
public class Client {
    public static void main(String[] args) {
        //创建原型对象
        ConcreteProtoyype concreteProtoyype = ConcreteProtoyype.getInstance();
        concreteProtoyype.setAge(18);
        concreteProtoyype.setName("huyawen");
        List<String> hobbies = new ArrayList<String>();
        hobbies.add("吃");
        hobbies.add("睡");
        hobbies.add("拉屎");
        concreteProtoyype.setHobbies(hobbies);

        //拷贝原型对象
        ConcreteProtoyype concreteProtoyype1  = concreteProtoyype.deepClone();
        concreteProtoyype1.getHobbies().add("学习");
        System.out.println("原型对象："+concreteProtoyype);
        System.out.println("克隆对象："+concreteProtoyype1);
        System.out.println(concreteProtoyype == concreteProtoyype1);

        System.out.println("原型对象的爱好：" + concreteProtoyype.getHobbies());
        System.out.println("克隆对象的爱好：" + concreteProtoyype1.getHobbies());
        System.out.println(concreteProtoyype.getHobbies() == concreteProtoyype1.getHobbies());
    }
}
```
打印：
```
原型对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎]}
克隆对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎, 学习]}
false
原型对象的爱好：[吃, 睡, 拉屎]
克隆对象的爱好：[吃, 睡, 拉屎, 学习]
false
```
发现这里两个对象不同，所以单例被破坏了。。。那么如何解决？  
  * 不实现Cloneable接口；（注意：**单例模式不能实现Cloneable接口**）
    
```
  @Override
    public ConcreteProtoyype clone() {
        return instance;
    }
```
 此时又变回了原型模式，没错，**原型和单例本身就是冲突的！！**，这种理论客观上就不存在。。
 作为一名架构师，要考虑代码的风险，要规避任何可能发生的情况，要防患于未来！
 有兴趣可以看一下JDK源码中的  
ArrayListClone方式：
```
     public Object clone() {
        try {
            ArrayList var1 = (ArrayList)super.clone();
            var1.elementData = Arrays.copyOf(this.elementData, this.size);
            var1.modCount = 0;
            return var1;
        } catch (CloneNotSupportedException var2) {
            throw new InternalError(var2);
        }
    }
```
```
     public static <T> T[] copyOf(T[] original, int newLength) {
        return (T[]) copyOf(original, newLength, original.getClass());
    }
```
```
     public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```
 
HashMap：
```
     public Object clone() {
        HashMap var1;
        try {
            var1 = (HashMap)super.clone();
        } catch (CloneNotSupportedException var3) {
            throw new InternalError(var3);
        }

        var1.reinitialize();
        var1.putMapEntries(this, false);
        return var1;
    }
```
```
     void reinitialize() {
        this.table = null;
        this.entrySet = null;
        this.keySet = null;
        this.values = null;
        this.modCount = 0;
        this.threshold = 0;
        this.size = 0;
    }
 ```
 ```
     final void putMapEntries(Map<? extends K, ? extends V> var1, boolean var2) {
        int var3 = var1.size();
        if (var3 > 0) {
            if (this.table == null) {
                float var4 = (float)var3 / this.loadFactor + 1.0F;
                int var5 = var4 < 1.07374182E9F ? (int)var4 : 1073741824;
                if (var5 > this.threshold) {
                    this.threshold = tableSizeFor(var5);
                }
            } else if (var3 > this.threshold) {
                this.resize();
            }

            Iterator var8 = var1.entrySet().iterator();

            while(var8.hasNext()) {
                Entry var9 = (Entry)var8.next();
                Object var6 = var9.getKey();
                Object var7 = var9.getValue();
                this.putVal(hash(var6), var6, var7, false, var2);
            }
        }
    }
```
 用这种方式改造一下我们的代码：
```
     public ConcreteProtoyype deepCloneHobbies() {
        try{
            ConcreteProtoyype result = (ConcreteProtoyype)super.clone();
            result.hobbies = (List) ((ArrayList)result.hobbies).clone();
            return result;
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }
    }
```
 客户端调用：
```
 ConcreteProtoyype concreteProtoyype1  = concreteProtoyype.deepCloneHobbies();
```
 结果：
 
```
原型对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎]}
克隆对象：ConcreteProtoyype{age=18, name='huyawen', hobbies=[吃, 睡, 拉屎, 学习]}
false
原型对象的爱好：[吃, 睡, 拉屎]
克隆对象的爱好：[吃, 睡, 拉屎, 学习]
false
```

### 总结

   只要是Cloneable接口下的都是浅克隆：ArrayList看似为深克隆，假设ArrayList里有引用类型呢？  
    怎么做才能实现深克隆：  
           序列化  
           转Json  
    如果不是架构师没必要了解原型模式，因为用不到；（apach.lang.util包；jdk实现了浅克隆；spring）  
#### 原型模式优点  

    性能优良，java自带的原型模式，是基于内存的二进制流的拷贝，比直接new一个对象性能更高；  
    可以使用深克隆方式保存对象的状态，使用原型模式将原对象复制一份并保存，简化了创建过程  
#### 原型模式的缺点

     必须配备克隆（或拷贝）方法  
     当对已有类进行改造的时候，需要修改代码，违反了开闭原则；  
     深拷贝，浅拷贝需要运用得当。  
