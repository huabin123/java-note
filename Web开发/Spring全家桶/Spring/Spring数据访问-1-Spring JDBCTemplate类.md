# Spring配置JDBC

在 Spring 中，JDBC 的相关信息在配置文件中完成，其配置模板如下所示（MySQL）。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http:/www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd"> 
   
    <!-- 配置数据源 --> 
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <!--数据库驱动-->
        <property name="driverClassName" value="com.mysql.jdbc.Driver" /> 
        <!--连接数据库的url-->
        <property name= "url" value="jdbc:mysql://localhost/xx" />
        <!--连接数据库的用户名-->
        <property name="username" value="root" />
        <!--连接数据库的密码-->
        <property name="password" value="root" />
    </bean>
    <!--配置JDBC模板-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!--默认必须使用数据源-->
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--配置注入类-->
    <bean id="xxx" class="xxx">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>
    ...
</beans>
```

# 参数说明

上述代码中定义了 3 个 Bean，分别是 dataSource、jdbcTemplate 和需要注入类的 Bean。

## dataSource

dataSource 对应的是 DriverManagerDataSource 类，用于对数据源进行配置

| 属性名          | 说明                                            |
| --------------- | ----------------------------------------------- |
| driverClassName | 所使用的驱动名称，对应驱动 JAR 包中的 Driver 类 |
| url             | 数据源所在地址                                  |
| username        | 访问数据库的用户名                              |
| password        | 访问数据库的密码                                |

## jdbcTemplate

在定义 JdbcTemplate 时，需要将 dataSource 注入到 JdbcTemplate 中。而在其他的类中要使用 JdbcTemplate，也需要将 JdbcTemplate 注入到使用类中（通常注入 dao 类中）。

在 JdbcTemplate 类中，提供了大量的查询和更新数据库的方法，如 query()、update() 等，如下表所示。

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| public int update(String sql)                                | 用于执行新增、修改、删除等语句 args 表示需要传入到 query 中的参数 |
| public int update(String sql,Object... args)                 |                                                              |
| public void execute(String sql)                              | 可以执行任意 SQL，一般用于执行 DDL 语句 action 表示执行完 SQL 语句后，要调用的函数 |
| public T execute(String sql, PreparedStatementCallback action) |                                                              |
| public T query(String sql, ResultSetExtractor rse)           | 用于执行查询语句 以 ResultSetExtractor 作为参数的 query 方法返回值为 Object，使用查询结果需要对其进行强制转型 以 RowMapper 作为参数的 query 方法返回值为 List |
| public List query(String sql, RowMapper rse)                 |                                                              |

## 示例