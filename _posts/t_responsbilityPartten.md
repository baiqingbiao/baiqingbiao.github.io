---
layout: post
title: 详解23种设计模式之--------（15）责任链模式
description: designPatterns_15
category: blog
---

[回到目录](#directory)

#### 责任链模式的定义
* 责任链模式（Chain Of Responsibility Pattern）是将每一个节点看作是一个对象每个节点处理的请求均不同,且内部自动维护
一个下一节点对象.当一个请求从链式的首段发出时,会沿着链的路径依次传递给每一个节点对象,直至有对象处理这个请求为止；
* 属于行为型模式

**生活中的场景**
* 工作生活中的审批流程;
* 产业链;

**责任链模式的适用场景**
* 多个对象可以处理同一请求,但具体有哪个对象处理则在运行时动态决定;  
* 在不明确指定接收者的情况下,向多个对象中的一个提交一个请求；    
* 可动态指定一组对象处理请求;

简单案例-优化登录：  
我们在写某个登录场景的时候，如果不用到设计模式的话，一般是这么做的：
```
public class login{
	public void login(String username,String password){
		if(StringUtils.isEmpty(username) || StringUtils.isEmpty(password)){
			System.out.println("用户名和密码为空");
			return;
		}
		System.out.println("用户名密码不为空，可以继续执行");
		Member member = checkExists(username,password);
		if(null == member){
			System.out.println("用户不存在！");
			return;
		}
		System.out.println("登陆成功！");
		if("admin".equals(member.getRoleName())){
			System.out.println("您不是管理员，没有操作权限！");
			return;
		}
	}
}
```
那么现在只是举了个例子，实际我们在这种场景中的逻辑可能会更加复杂，那么这里就可以用到责任链模式来进行优化一下
，将这三步分别封装起来形成一个链表；新建一个Handler（抽象链表）：
```
public abstract class Handler {
    protected Handler handler;
    public void next(Handler handler){
        this.handler = handler;
    }

    public abstract void doHandler(Member member);
}
```
三個步驟分別執行：
```
public class ValidateHandler extends Handler {
    public void doHandler(Member member) {
        if(StringUtils.isEmpty(member.getUsername()) || StringUtils.isEmpty(member.getPassword())){
            System.out.println("用户名和密码为空");
            return;
        }
        System.out.println("用户名密码不为空，可以继续执行");
        handler.doHandler(member);
    }
}
```
```
public class LoginHandler extends Handler {
    public void doHandler(Member member) {
        if(null == member){
            System.out.println("用户不存在！");
            return;
        }
        System.out.println("登陆成功！");
        handler.doHandler(member);
    }
}
```
```
public class AuthHandler extends Handler {
    public void doHandler(Member member) {
        if(!"admin".equals(member.getRoleNam(member.getUsername(),member.getPassword()))){
            System.out.println("您不是管理员，没有操作权限！");
            return;
        }
        System.out.println("操作成功");

    }
}
```
登录实体类：
```
public class Member {

    private String username;
    private String password;

    public Member(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getRoleNam(String username, String password){
        if("admin".equals(username) && "admin".equals(password)){
            return "admin";
        }
        return null;
    }
}

```
service供调用（并设置层级关系）：
```
public class MemberService {
    public void login(String username,String password){
        ValidateHandler validateHandler = new ValidateHandler();
        LoginHandler loginHandler = new LoginHandler();
        AuthHandler authHandler = new AuthHandler();

        validateHandler.next(loginHandler);
        loginHandler.next(authHandler);

        Member member = new Member(username,password);
        validateHandler.doHandler(member);
    }
}
```
测试：
```
public class Test {
    public static void main(String[] args) {
        MemberService memberService = new MemberService();
        memberService.login("admin","admin");
    }
}
```
执行结果：
```
用户名密码不为空，可以继续执行
登陆成功！
操作成功
```

那么现在看就成了一个链式的模式，包括让他先去执行哪一个再去执行哪一个；这时候又想到，不是有个链式编程吗，，
所以代码还可以这样改造（将抽象链表改为抽象双向链表）（与建造者模式相结合）：
Handler(通过内部类传递泛型):
```
public abstract class Handler<T> {
    protected Handler<T> handler;
    public void next(Handler<T> handler){
        this.handler = handler;
    }

    public abstract void doHandler(Member member);
    public static class Build<T>{
        private Handler<T> head;
        private Handler<T> tail;

        public Build<T> addBuilder(Handler handler){
            if(this.head == null){
                this.head = this.tail = handler;
            }
            this.tail.next(handler);
            this.tail = handler;
            return this;
        }

        public Handler build(){
            return this.head;
        }
    }
}
```
service：
```
public class MemberService {
    public void login(String username,String password){
        Handler.Build build = new Handler.Build();
        build.addBuilder(new ValidateHandler())
                .addBuilder(new LoginHandler())
                .addBuilder(new AuthHandler());

        build.build().doHandler(new Member(username,password));
//用过Netty的人肯定见过
    }
}
```

那么此时逻辑性就更加强了，但是双向链表还没有真正形成，有兴趣可以自己研究一下。
```
public abstract class Handler<T> {
    protected Handler<T> handler;
    public void next(Handler<T> handler){
        this.handler = handler;
    }

    public abstract void doHandler(Member member);
    public static class Build<T>{
        private Handler<T> head;
        private Handler<T> tail;

        public Build<T> addBuilder(Handler handler){
            do{
                if(this.head == null){
                    this.head = this.tail = handler;
                    break;
                }
                this.tail.next(handler);
                this.tail = handler;
            }while(false);
            return this;
        }

        public Handler build(){
            return this.head;
        }
    }
}
```

源码应用：（javax.servlet.Filter,FilterChain，MorkFilterChain,ChainHandler）

**责任链模式的优点**
* 将请求与处理解耦；
* 请求处理者（节点对象）只需关注自己感兴趣的请求进行处理即可；
* 具备链式传递处理请求功能，请求发送者无需知晓链路结构，只需等待处理结果就好；
* 链路结构灵活，可以通过改变链路结构动态的新增或删减责任；
* 易于扩展新的请求处理类（节点），符合开闭原则；
**责任链模式缺点**
* 责任链太长或者处理时间过长，会影响整体性能；
* 如果节点对象存在循环引用时，会造成系统死循环，导致系统奔溃；
