<link rel="stylesheet" type="text/css" href="css/CoreJava.css">
#<span id="top">Core Java Volume II</span>
[TOC]

##数据库编程
###JDBC的设计
1. JDBC接口组织方式遵循了微软非常成功的ODBC模式。
    JDBC和ODBC都基于同一个思想：根据API编写的程序都可以与驱动管理器进行通信，而驱动管理器则通过驱动程序与实际的数据库进行通信。

2. JDBC连接数据库的一般操作
    1. 注册驱动类
    某些JDBC的JAR文件（包含META-INF/services/java.sql.Driver文件）可以自动注册驱动器类。
    通过DriverManager，可以使用两种方式手动注册驱动器。
    一种是在Java程序中加载驱动器类，如：
    `Class.forName("org.postgresql.Driver");`
    另一种方式是设置jdbc.driver属性，可以通过命令行参数来指定该属性，如：
    `java -Djdbc.driver=org.postgresql.Driver ProgramName`
    或者在应用中设置系统属性：
    `System.setProperties("jdbc.driver", "org.postgresql.Driver");`
    这种方式中可以提供多种驱动器，用“:”分隔
    2. 获取数据库连接
    使用数据库URL，通过DriverManager的`getConnection()`方法获得一个数据库连接：
    ```java
    String url = "jdbc:postgresql:myDB";
    String username = "dbuser";
    String password = "secret";
    Connection conn = DriverManager.getConnection(url, username, password);
    ```
    3. 获取Statement，执行SQL
    通过Connection对象的`createStatement()`方法获取Statement对象，然后使用`execute()`、`executeQuery()`或者`executeUpdate()`执行SQL语句
    ```java
    Statement  stat = conn.createStatement();
    String command = "UPDATE Books"
        + " SET Price = Price -5"
        + " WHERE Title NOT LIKE '%Introduction%'";
    stat.executeUpdate(command);
    ```
    或者使用Connection对象的`prepareStatement()`方法获取PrepareStatement。
    4. 处理结果（异常处理，结果集处理ResultSet）。
    5. 关闭ResultSet、Statement、Connection。


















[<button style="position: fixed; bottom: 50px;right: 50px;height:35px;width:50px;font-size:16px;">TOP</button>](#top)
