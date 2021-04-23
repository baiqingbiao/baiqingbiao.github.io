---
layout: post
title: 详解23种设计模式之--------（14）策略模式
description: designPatterns_14
category: blog
---

[回到目录](#directory)

#### 策略模式的定义
* 策略模式（Srtategy Pattern）通常又叫政策（Policy Pattern）模式，它是将定义的算法家族，分别封装起来，让他们之间可以互相替换
从而让算法的变化不会影响到用户的使用；
* 可以避免多重分支的if...else...  switch语句；
* 属于行为型模式


**生活中的场景**
* 个人所得税，收入在某个范围内要交多少税
* 从一个地方到另一个地方去，选择什么交通工具；
* 使用不同的支付方式支付。

**策略模式的适用场景**
* 假如系统中有很多类，而它们的区别仅在于他们的行为不同。  
* 一个系统需要动态的在几种算法中选择一种；    
* 需要屏蔽算法规则

简单案例-标准的策略模式写法(假设现在买东西有几种优惠)：
```
public interface IPromotionStrategy {
    void doStrategy();//执行优惠策略
}
```
优惠方式:
```
public class CashBackStrategy implements IPromotionStrategy {
    public void doStrategy() {
        System.out.println("返现,打款到账户");
    }
}
```
```
public class CouponStrategy implements IPromotionStrategy {
    public void doStrategy() {
        System.out.println("使用优惠券抵扣");
    }
}
```
```
    public void doStrategy() {
        System.out.println("五人团购,优惠一半");
    }
}
```
```
public class NoStrategy implements IPromotionStrategy{

    public void doStrategy() {
        System.out.println("无优惠");
    }
}
```
策略分配：
```
public class STrategyActivity  {
    private IPromotionStrategy iPromotionStrategy;

    public STrategyActivity(IPromotionStrategy iPromotionStrategy) {
        this.iPromotionStrategy = iPromotionStrategy;
    }

    public void execute(){
        iPromotionStrategy.doStrategy();
    }
}
```
测试：
```
public class Test {
    public static void main(String[] args) {
        //String pomotion = "团购";
        //IPromotionStrategy iPromotionStrategy = null;
        STrategyActivity activity = new STrategyActivity(new GroupBuyStrategy());
        //iPromotionStrategy.doStrategy();
        activity.execute();
    }
}
```

那么这种写法的话是要通过传递一个类过来如GroupBuyStrategy，才能实现，而且这也是基于Java语言实现的，
如果说换一种语言就可能不行，所以需要改造一下。搭配工厂模式使用,新建一个工厂类（单例）：
```
public class PomotionStrategyFactory {

    private static Map<String,IPromotionStrategy> promotionStrategyMap = new HashMap<String, IPromotionStrategy>();
	
    static {
        promotionStrategyMap.put("CASHBACK",new CashBackStrategy());
        promotionStrategyMap.put("COUPON",new CouponStrategy());
        promotionStrategyMap.put("GROUPBUY",new GroupBuyStrategy());
    }
	
    private static IPromotionStrategy noStrategy = new NoStrategy();

    private PomotionStrategyFactory() {
    }

    public static IPromotionStrategy getPromotionStrategy(String pomotionKey){//通过参数获得对应实例
        return promotionStrategyMap.get(pomotionKey) == null ? noStrategy : promotionStrategyMap.get(pomotionKey);
    }

    public interface PomotionKey{//常量定义
        String CASHBACK = "CASHBACK";
        String COUPON = "COUPON";
        String GROUPBUY = "GROUPBUY";
    }

    public static Set<String> getPromotionKeys(){//创建返回key列表方法
        return promotionStrategyMap.keySet();//返回的选择列表,供用户选择
    }
}
```
然后调用的时候：
```
public class Test {
    public static void main(String[] args) {
        //String pomotion = "团购";
        //IPromotionStrategy iPromotionStrategy = null;
        //iPromotionStrategy.doStrategy();

        //STrategyActivity activity = new STrategyActivity(new GroupBuyStrategy());
        //activity.execute();

        PomotionStrategyFactory.getPromotionKeys();//先拿到所有的key给客户端选择
        String promotionKey = "GROUPBUY";//用户选择后返回的值
        IPromotionStrategy iPromotionStrategy = PomotionStrategyFactory.getPromotionStrategy(promotionKey);
        iPromotionStrategy.doStrategy();

    }
}
```
这样就完美的解决了问题，完成了从草根到正规军的转换。

在实际业务场景中的应用（使用不同的方式支付）,  
支付抽象类：
```
public abstract class Payment {
    //通用逻辑放在抽象方法里面
    public ResultMsg pay(String uid,Double amount ){
        if(queryBalance() < amount){
            return new ResultMsg("500","支付失败","余额不足");
        }
        return new ResultMsg("200","支付成功","");
    }

    protected abstract double queryBalance();
    public abstract String getName();

}
```
几种支付方式：
```
public class AliPay extends Payment {
    protected double queryBalance() {
        return 200;
    }

    public String getName() {
        return "支付宝";
    }
}
```
```
public class JD_pay extends Payment {
    protected double queryBalance() {
        return 300;
    }

    public String getName() {
        return "京东白条";
    }
}
```
```
public class UnionPay extends Payment {
    protected double queryBalance() {
        return 1000;
    }

    public String getName() {
        return "银联支付";
    }
}
```
```
public class Wechat extends Payment {
    protected double queryBalance() {
        return 250;
    }

    public String getName() {
        return "微信支付";
    }
}

```
返回结果信息类：
```
public class ResultMsg {

    private String code;
    private String data;
    private String msg;

    public ResultMsg(String code, String data, String msg) {
        this.code = code;
        this.data = data;
        this.msg = msg;
    }

    @Override
    public String toString() {
        return "ResultMsg{" +
                "code='" + code + '\'' +
                ", data='" + data + '\'' +
                ", msg='" + msg + '\'' +
                '}';
    }
}
```
支付策略工厂：
```
public class PayStrategy {
    public static final String ALI_PAY = "AliPay";
    public static final String JD_PAY = "JDPay";
    public static final String WECHART_PAY = "WechatPay";
    public static final String UNION_PAY = "UnionPay";
    public static final String DEFAULT = "UnionPay";

    private static Map<String,Payment> paymentMap = new HashMap<String, Payment>();

    static {
        paymentMap.put(ALI_PAY,new AliPay());
        paymentMap.put(JD_PAY,new JD_pay());
        paymentMap.put(WECHART_PAY,new Wechat());
        paymentMap.put(UNION_PAY,new UnionPay());
    }
    private PayStrategy() {
    }

    public static Payment getPayment(String keyPayment){
        if(!paymentMap.containsKey(keyPayment)){
            return paymentMap.get(DEFAULT);
        }
        return paymentMap.get(keyPayment);
    }
}
```
订单类:
```
public class Order {
    private String uid;
    private String orderId;
    private double amount;

    public Order(String uid, String orderId, double amount) {
        this.uid = uid;
        this.orderId = orderId;
        this.amount = amount;
    }

    public ResultMsg pay(){
        return pay(PayStrategy.DEFAULT);
    }

    public ResultMsg pay(String keyPayment) {
        Payment payment = PayStrategy.getPayment(keyPayment);
        System.out.println("欢迎使用" + payment.getName());
        System.out.println("本次交易金额为:" + amount + ",开始扣款");
        return payment.pay(uid,amount);
    }
}
```
测试：
```
public class Test {

    public static void main(String[] args) {
        Order order = new Order("2018123152982340","1241291244124124",800.5);
        System.out.println(order.pay());

        System.out.println("==");
        System.out.println(order.pay(PayStrategy.WECHART_PAY));
    }
}
```
执行结果：
```
欢迎使用银联支付
本次交易金额为:800.5,开始扣款
ResultMsg{code='200', data='支付成功', msg=''}
欢迎使用微信支付
本次交易金额为:800.5,开始扣款
ResultMsg{code='500', data='支付失败', msg='余额不足'}
```


**策略模式的优点**
* 策略符合开闭原则；
* 避免使用多重条件转移语句，如if...else..语句/switch语句；
* 使用策略模式可以提高算法的保密性和安全性；
**策略模式缺点**
* 客户端必须知道所有的策略(需要它来进行选择),并且自行决定使用哪一个策略；
* 代码中会产生非常多的策略类,增加维护难度；
