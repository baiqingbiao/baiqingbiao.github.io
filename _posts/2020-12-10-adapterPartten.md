---
layout: post
title: 详解23种设计模式之--------（10）适配器模式
description: designPatterns_10
---

[回到目录](./designPatterns#directory)

#### 适配器模式的定义
适配器模式（Adapter Pattern）又称为变压器模式，顾名思义，它的功能是将一个类的接口变成客户端所期望的另一种接口，
从而使原本接口不匹配而导致无法在一起工作的两个类能够一起工作；

属于结构型模式。

**生活中的场景**
* 变压器（传输时使用高电压，用户使用220V等）
* 两脚插头转三角
* 手机充电接口（Type-C/Lighting）
* 显示器转换头

**适配器模式的适用场景**
* 已经存在的类，他的方法和需求不匹配（方法结果相同或相似）的情况；
* 适配器模式不是软件设计阶段考虑的设计模式，是随着软件维护，由于不同的产品/不同厂家造成的功能类似而接口不相同情况写的解决方案（后期的一种处理方案）。
* 符合开闭原则，只做扩展，可以用门面/代理//装饰器等搭配使用，主要是为了解决兼容性问题。

适配器通用写法：  
适配器模式的三个角色:
* **目标角色** 期望要实现兼容的接口
* **原角色** 在系统中原有的已经满足需求的功能
* **适配器**  在原有基础上，能够实现与满足现在目标接口与原功能的相互兼容

**适配器的三种写法**
* 1.类适配器
* 2.对象适配器
* 3.接口适配器

用一个经典案例看看,假设我现在要用220V的电压连接适配器给5V手机充电：  
**类适配器的实现方式（继承原有的，实现目标接口，作出处理）**
原有对象：220V
```
public class AC220 {
    public int output() {
        int output = 220;
        System.out.println("输出电压:220V");
        return output;
    }
}
```
目标对象：
```
public interface DC5 {
    int output5V();
}
```
现在调用是不行的，直接就烧坏了，这时候需要一个适配器做出转换：
```
public class PowerAdapter extends AC220 implements DC5{

    public int output5V() {
        int adapterInput = super.output220V();//
        int adapterOputput = adapterInput / 44;
        System.out.println("使用Adapter输入："+ adapterInput + "V,输出：" + adapterOputput + "V");
        return adapterOputput;
    }
}
```
用户使用：
```
public class Usr {
    public static void main(String[] args) {
        DC5 powerAdapter = new PowerAdapter();
        powerAdapter.output5V();
        ((PowerAdapter) powerAdapter).output220V();

        /*PowerAdapter powerAdapter1 = new PowerAdapter();
        powerAdapter1.output5V();
        powerAdapter1.output220V();*/
    }
}

```
输出：
```
输出电压:220V
使用Adapter输入：220V,输出：5V
输出电压:220V
```
是一个标准的适配器，但是这时候它违背了类的最少知道原则，现在来看对象适配器组合复用（使用目标对象的方式，不继承）：  
修改适配器
```
public class PowerAdapter implements DC5 {
    private AC220 ac220v;

    public PowerAdapter(AC220 ac220v) {
        this.ac220v = ac220v;
    }

    public int output5V() {
        int adapterInput = ac220v.output220V();//
        int adapterOputput = adapterInput / 44;
        System.out.println("使用Adapter输入："+ adapterInput + "V,输出：" + adapterOputput + "V");
        return adapterOputput;
    }
}
```
用户：
```
public class Usr {
    public static void main(String[] args) {
        DC5 powerAdapter = new PowerAdapter(new AC220());
        powerAdapter.output5V();
    }
}
```
接口适配器的关注点不一样，类适配器和对象适配器关注实现方式，接口适配器是为了解决实现接口时实现了很多没必要的类/类的冗余问题
从而只在一个类中需要实现必要的方法；  
假如说现在不仅将220V转换为5V，还要转换为12V，24V等等：  
这时候就只改造DC5接口增加方法：
```
public interface DC {
    int output5V();
    int output12V();
    int output24V();
    int output36V();
}
```
适配器更改：
```
public class PowerAdapter implements DC {
    private AC220 ac220v;

    public PowerAdapter(AC220 ac220v) {
        this.ac220v = ac220v;
    }

    public int output5V() {
        int adapterInput = ac220v.output220V();//
        int adapterOputput = adapterInput / 44;
        System.out.println("使用Adapter输入："+ adapterInput + "V,输出：" + adapterOputput + "V");
        return adapterOputput;
    }

    public int output12V() {
        int adapterInput = ac220v.output220V();//
        int adapterOputput = adapterInput / (220/12);
        System.out.println("使用Adapter输入："+ adapterInput + "V,输出：" + adapterOputput + "V");
        return adapterOputput;
    }

    public int output24V() {
        return 0;
    }

    public int output36V() {
        return 0;
    }

```
用户调用就随便调用了，但是这时候有点违背单一职责原则，所以用的时候还要具体情况具体分析。

**适配器模式在业务场景中的应用**
假设现在某系统有一些存量用户，按照原有的方式就是先去注册然后再登陆系统，西安在要对系统做一次升级，就是可以通过第三方或应用登录比如QQ，微信，手机号等
，那么在这个场景中原有的登陆方式就是原角色，第三方登录为目标角色，适配器就是通过第三方来实现注册+登录。  
原先登录场景中用到的实例：
```
@Data
public class Member {
    private String username;
    private String password;
    private String mid;
    private String info;
}

```
返回信息：
```
@Data
public class ResultMsg {
    private int code;
    private String msg;
    private Object data;

    public ResultMsg(int code, String msg, Object data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }
}
```
注册登陆方法:
```
public class PassportService {
    public ResultMsg register(String username,String password){
        return new ResultMsg(0,"成功","");
    }

    public ResultMsg login(String username,String password){
        return null;
    }
}
```
使用适配器模式新增目标接口，假设现在要使用QQ登录，那么就要调用腾讯服务器那边的一个api/sdk来对接，但是腾讯不可能告诉我qq号与密码，所以只能拿到一个openId
，假如微信也只能拿到一个openId，手机号的话可能还需要一个验证码，看手机号是否有效：
```
public interface IPassportForThird {
    ResultMsg loginForQQ(String openId);

    ResultMsg loginForWechat(String openId);

    ResultMsg loginForMobilephone(String phone,String validateNo);

}
```
那么现在原角色/目标角色有了，创建一个适配器（类适配器 继承+实现）：
```
public class PassportAdapterForThird extends PassportService implements IPassportForThird {
    public ResultMsg loginForQQ(String openId) {
        //openId获取逻辑
        return loginForRegister(openId,null);
    }

    public ResultMsg loginForWechat(String openId) {
        return loginForRegister(openId,null);
    }

    public ResultMsg loginForMobilephone(String phone, String validateNo) {
        //与通信方对接，验证码校验逻辑
        return loginForRegister(phone,null);
    }

    public ResultMsg loginForRegister(String username,String password){
        //如果使用QQ，微信登录是不需要密码的，所以做一个约定
        if(null == password){
            password = "THIRD_EMPTY";//在后台做一个操作，如果密码是这个，就认为使用第三方登录
        }
        super.register(username,password);//先注册
        return super.login(username,password);//在登陆
    }
}

```
登录测试：
```
public class Test {
    public static void main(String[] args) {
        PassportAdapterForThird thirdlogin = new PassportAdapterForThird();
        thirdlogin.login("baiqb","123456");//原有的登陆方式
        thirdlogin.loginForQQ("saefsadfdsaDFASSDFAS23");//qq登陆方式
    }
}
```
这个适配器的作用就是实现一个兼容，暗号原先的模式需要先注册，所以需要一个内部兼容方法；  
但是这个适配器可以看到，每种的处理逻辑都不一样，如果再加上处理逻辑，这个类就会变得很臃肿，
包括以后再有新的登陆方式，如抖音微博小红书等登录，还需要再对这个类进行修改，这个时候就不再符合开闭原则，所以这时候
还需要再做一次升级。  
需要对适配器进行一个扩展。
```
public class PassportAdapterForThird implements IPassportForThird {
    public ResultMsg loginForQQ(String openId) {
        //openId获取逻辑
        return processLogin(openId,LoginForQQAdapter.class);
    }

    public ResultMsg loginForWechat(String openId) {
        return processLogin(openId,LoginForWechatAdapter.class);
    }

    public ResultMsg loginForMobilephone(String phone, String validateNo) {
        //与通信方对接，验证码校验逻辑
        return processLogin(phone,LoginForMobilephoneAdapter.class);
    }

    public ResultMsg processLogin(String id,Class<? extends ILoginAdapter> clazz){//传递class的作用就是让适配器起到一个调度的作用，具体的实现逻辑去对应的类区实现
        try {
            ILoginAdapter iLoginAdapter = clazz.newInstance();
            if(iLoginAdapter.support(iLoginAdapter)){
                return  iLoginAdapter.login(id,iLoginAdapter);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
增加一个登录适配器接口（作用是判断是否支持）：
```
public interface ILoginAdapter {
    boolean support(Object object);//判断是否支持

    ResultMsg login(String id,Object adapter);//进行登录
}
```
增加各种登陆方式的处理逻辑，这里方便调用与统一起来，抽象出一个适配器继承原有的PassportAdapter与实现ILoginAdapter：
```
public abstract class AbstractAdapter extends PassportService implements ILoginAdapter {
    protected ResultMsg loginforRegister(String username, String password) {
        //如果使用QQ，微信登录是不需要密码的，所以做一个约定
        if(null == password){
            password = "THIRD_EMPTY";//在后台做一个操作，如果密码是这个，就认为使用第三方登录
        }
        super.register(username,password);//先注册
        return super.login(username,password);//在登陆
    }
}
```
QQ:
```
public class LoginForQQAdapter extends AbstractAdapter {
    public boolean support(Object adapter) {
        return adapter instanceof LoginForQQAdapter;
    }

    public ResultMsg login(String id, Object adapter) {
        return super.loginforRegister(id,null);
    }

}
```
Wechat:
```
public class LoginForWechatAdapter extends AbstractAdapter {
    public boolean support(Object adapter) {
        return adapter instanceof LoginForWechatAdapter;
    }

    public ResultMsg login(String id, Object adapter) {
        return super.loginforRegister(id,null);
    }
}
```
Phone
```
public class LoginForMobilephoneAdapter extends AbstractAdapter {
    public boolean support(Object adapter) {
        return adapter instanceof LoginForMobilephoneAdapter;
    }

    public ResultMsg login(String id, Object adapter) {
        return super.loginforRegister(id,null);
    }

}
```
调用起来就非常简单了：
```
public class Test {
    public static void main(String[] args) {
        PassportAdapterForThird loginThird = new PassportAdapterForThird();
        loginThird.loginForQQ("asdgfadfdasgaesfgre");
    }
}
```
那么现在分析一下，假设现在要扩展一个用抖音进行登录的方式，那么需要修改第三方接口IPassportForThird，实现类PassportAdapterForThird，新加抖音登录处理逻辑类；
这个问题值得考虑一下。在Spring源码中处理得非常好，对外的是一个handler（AdvisorAdapter,HandlerAdapter）。

![类图](/images/designPattern/10-1.PNG)  

**适配器模式的优点**
* 能提高类的透明性和复用，现有的类复用但不需要改变；
* 目标类和适配器类j解耦，提高程序的扩展性；
* 在很多业务场景中符合开闭原则；

**适配器模式的缺点**
* 适配器编写过程需要全面考虑，可能会增加系统的复杂性；
* 增加代码阅读难度，降低代码的可读性，过多的使用适配器会是代码变得凌乱。

设计模式不要滥用，适合才是最好的。
