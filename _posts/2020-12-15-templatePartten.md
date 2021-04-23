---
layout: post
title: 详解23种设计模式之--------（13）模板方法模式
description: designPatterns_13
---

[回到目录](./designPatterns#directory)

#### 模板方法模式的定义
* 模板方法模式（Template Method Pattern）通常又叫模板模式，是指定义一个算法的骨架，并允许子类为其中的一个
或多个提供实现；
* 模板方法可以在子类不改变算法结构的情况下，重新定义算法的某些步骤；
* 属于行为型模式


**生活中的场景**
* 正所谓模板，就是有定义了一个框架，先做什么在做什么（如做一道菜的方法），并允许微调（放多少香料）。
* 打开冰箱门，把大象装进冰箱，关闭冰箱门；打开冰箱门，把大象拿出来，把长颈鹿装进去，关闭冰箱门，不变的步骤是打开冰箱门、~、关闭冰箱门。

**模板方法模式的适用场景**
1.一次性实现一个算法不变的部分，并将可变的行为留给子类实现；  
2.各子类公共的行为被提取出来并集中到一个公共的父类中，从而避免代码重复；    

了解模板方法模式常用的钩子方法：  
假设现在A和B都要执行上课的一套流程，在‘检查作业’阶段要判断是不是需要检查，其余步骤按部就班：  
模板流程：
```
package src.templatePattren;

public abstract  class AbstractCourse {
    protected final void process(){
        //1.发布预习资料
        postResource();

        //2.制作学习课件
        makePPT();

        //3.直播上课
        liveHouse();

        //4.上传课后资料
        uploadResource();

        //5.布置作业
        makeWork();

        //6.检查作业，这一步是要看子类需不需要，子类通过重写needHomeWork并设置返回值为true，子类的needHomeWork方法就会被执行
        if(needHomeWork()){
            checkHomeWork();
        }
    }

    abstract void checkHomeWork();

    protected boolean needHomeWork(){
        return false;//默认false
    }

    protected void liveHouse(){
        System.out.println("发布预习资料");
    }

    protected void makeWork(){
        System.out.println("制作学习课件");
    }

    protected void uploadResource(){
        System.out.println("直播上课");
    }

    protected void postResource(){
        System.out.println("上传课后资料");
    }

    protected void makePPT(){
        System.out.println("布置作业");
    }
}

```
Java课程：
```
public class JavaCourse extends AbstractCourse {
    private boolean isNeedCheckHomework;

    public void setNeedCheckHomework(boolean needCheckHomework) {
        isNeedCheckHomework = needCheckHomework;
    }

    void checkHomeWork() {
        System.out.println("检查java作业");
    }

    @Override
    protected boolean needHomeWork() {
        return isNeedCheckHomework;
    }
}
```
python课程：
```
public class PythonCourse extends AbstractCourse {
    private boolean isNeedCheckHomework;

    public void setNeedCheckHomework(boolean needCheckHomework) {
        isNeedCheckHomework = needCheckHomework;
    }

    void checkHomeWork() {
        System.out.println("检查python作业");
    }

    @Override
    protected boolean needHomeWork() {
        return isNeedCheckHomework;
    }
}
```
测试（javaCourse设置了true）：
```
public class Test {
    public static void main(String[] args) {
        JavaCourse javaCourse = new JavaCourse();
        javaCourse.setNeedCheckHomework(true);
        javaCourse.process();

        System.out.println();

        PythonCourse pythonCourse = new PythonCourse();
        pythonCourse.setNeedCheckHomework(false);
        pythonCourse.process();
    }
}
```
执行l
```
发布预习资料
制作学习课件
直播上课
上传课后资料
布置作业
检查java作业

发布预习资料
制作学习课件
直播上课
上传课后资料
布置作业
```

通过一个业务场景深入了解下模板方法模式。
模拟JdbcTemplate：  

实体类：
```
@Data//实体类对应库表
public class Member {
    private String username;
    private String password;
    private String nickname;
    private int age;
}
```
创建Jdbc模板类（其中parseResultset为钩子方法，其在接口RowMapper中定义，并将参数设置泛型）：
```
public abstract  class JdbcTemplate {
    //有些步骤没完成,要设置成抽象类
    //首先需要一个数据源
    private DataSource dataSource;

    public JdbcTemplate(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public final List<?> executeQuery(String sql,RowMapper<?> rowMapper,Object []values) {

        try {
            //1.获取链接
            Connection connection = this.getConnection(dataSource);

            //2.创建语句集
            PreparedStatement ps = this.getPreparedStatement(connection, sql);

            //3.执行语句集
            ResultSet resultSet = this.executeQuery(ps,values);

            //4.处理结果集
            List<?> result = this.parseResultset(resultSet,rowMapper);

            //5.关闭语句集
            ps.close();

            //6.关闭结果集
            resultSet.close();

            //7.关闭连接
            connection.close();

            return  result;
        } catch (Exception e) {
            e.printStackTrace();
        }

        return null;
    }

    //钩子方法
    private List<?> parseResultset(ResultSet resultSet, RowMapper<?> rowMapper) throws Exception {
        List<Object> result = new ArrayList<Object>();
        int rownum = 1;
        while(resultSet.next()){
            result.add(rowMapper.mapRow(resultSet, rownum++));
        }
        return result;
    }

    protected ResultSet executeQuery(PreparedStatement ps, Object[] values) throws SQLException {

        for (int i = 0; i < values.length ; i++) {
            ps.setObject(i,values[i]);
        }
        return ps.executeQuery();
    }

    protected PreparedStatement getPreparedStatement(Connection connection, String sql) throws SQLException {
        return connection.prepareStatement(sql);
    }

    protected Connection getConnection(DataSource dataSource) throws SQLException {
        return  this.dataSource.getConnection();
    }
}
```
对外RowMapper：
```
public interface RowMapper<T> {
    T mapRow(ResultSet resultSet, int rownum) throws Exception;
}
```
MemberDao层处理逻辑：
```
public class MemberDao extends JdbcTemplate {
    public MemberDao(DataSource dataSource) {
        super(dataSource);
    }
    public List<?> selectAll(){
        String sql = "select * from t_member";
        return super.executeQuery(sql, new RowMapper<Member>(){
            public Member mapRow(ResultSet resultSet, int rownum) throws Exception {
                Member member = new Member();
                //在JdbcTemplate内部,会将这些值处理
                member.setAge(resultSet.getInt("age"));
                member.setUsername(resultSet.getString("username"));
                member.setPassword(resultSet.getString("password"));
                member.setNickname(resultSet.getString("nickname"));
                return member;
            }
        },null);
    }
}
```
调用:
```
public class Test {
    //调用
    public static void main(String[] args) {
        MemberDao memberDao = new MemberDao(null);
        List<?> result = memberDao.selectAll();
    }
}

```
(有兴趣去查JdbcTemplate，AbstractList.get(),HttpServlet.service.doget(),dopost())给用户实现的钩子方法。


**模板方法模式的优点**
* 利用模板方法将相同处理逻辑的代码放到抽象父类中，可以提高代码的复用性；
* 将不同的代码分别放在不同的子类中，通过对子类的扩展增加新的行为，提高代码的扩展性；
* 把不变的行为写在父类上，去除子类的重复代码，提供一个很好的代码复用平台，符合开闭原则；
**模板方法模式缺点**
*  类数目的增加，每一个抽象类都需要一个子类来实现，这样导致类的个数增加；
* 类数量的增加，间接的增加了系统的复杂度；
* 继承关系自身的缺点，如果父类增加新的抽象方法，所有子类都要新加；
