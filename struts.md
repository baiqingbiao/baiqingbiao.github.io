## 我的学习笔记

### struts2 执行过程：

   >输入localhost：8080/struts2...intruduction/...struts2 向服务器发送请求后，服务器寻找到其下的web application，执行web application下的
web.xml,web.xml下配置了一个Struts核心Filter，叫做StrutsPrepareAndExecuteFilter，这个过滤器会拦截所有的url地址，/*

``` java
   <package name = "default" namespace="/" extends="struts-default">
     <action name="user">
       <result>/hello.jsp</result>
      </action>
   </package>
```

   当他接收到url请求后，他首先寻找namespace=“/”，然后就会查找“/”后对应的hello然后他就会去package下寻找有没有name=hello 的action，有的话，找到对应的result是谁-----hello.jsp
### 通配符调用
   >如果不管用，一定是没有用这个：<global-allowed-methods>regex:.&#96;*</global-allowed-methods> 或者<allowd-methods>
请求转发： chain

### 修改参数标签的顺序
   >struts 依次向下查找，查找顺序为：1：struts-default.xml   2:struts-plugin.xml   3:struts.xml   4:srtuts.propreties
5:web.xml

ognl表达式就是它去哪里去找它这些对象，ognl表达式有一个根（root），他这个根就是我们的值栈，查找值栈当中的对象（值栈当中都是放的action，查找action中的属性，action当中的user）的时候不需要加 # 号，直接user.username
如果查找其他的一些对象（session，request，application等）他和根 （value stack（root）是并列的关系）所以查找的时候要加上“#”号。
attr的作用，查找放在哪个作用域里。

MethodFilterInterceptor里面有两个
1：excludemethods  这里面者的方法名都不会被拦截器拦截
2：includemethods  这里面指的方法名都会被拦截
自定义拦截器的设置
如果写一个自定义的拦截器，要先写一个类，然后根据需要实现interceptor接口，也可以继承AbstractInterceptor类或者MethodFilterInterceptor
如果第三个就要实现doInterceptor方法。tring result = invocation.invoke();  //invoke 是一个分水岭，在这个方法执行之前，都是在action调用之前执行。




































