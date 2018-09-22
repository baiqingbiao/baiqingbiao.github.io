## Struts2学习笔记

struts2 执行过程：

输入localhost：8080/struts2...intruduction/...struts2 向服务器发送请求后，服务器寻找到其下的web application，执行web application下的web.xml,web.xml下配置了一个Struts核心Filter，叫做
StrutsPrepareAndExecuteFilter，。这个过滤器会拦截所有的url地址，/*

   <package name = "default" namespace="/" extends="struts-default">
     <action name="user">
       <result>/hello.jsp</result>
      </action>
   </package>


-----hello.jsp
-----hello.jsp
