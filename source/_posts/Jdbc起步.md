---
title: Jdbc的起步
date: 
tags: [java,jdbc]
categories: [java]
top_img: https://cdn.jsdelivr.net/gh/MyPoi/cloudimg/img10.png
cover:  https://cdn.jsdelivr.net/gh/MyPoi/cloudimg/img10.png
---

# JDBC

前言

![image-20211108110936488](https://gitee.com/yoyosave/blog-image/raw/master/imgs/image-20211108110936488.png)

模拟sun公司以及mysql公司之间的关系

```java
package com.study.jdbc;

/**
 * @author YoMo
 * @since pro3 1.0
 */

public class OracleCompany implements JdbcInterface{
    @Override
    public void getConnetion() {
        System.out.println("成功连接Oracl数据库");
    }
}
```

```java
package com.study.jdbc;

/**
 * @author YoMo
 * @since pro3 1.0
 */

public class MySqlCompany implements JdbcInterface{

    @Override
    public void getConnetion() {
        System.out.println("成功连接Mysql数据库");
    }
}

```

```java
package com.study.jdbc;

/**
 * @author YoMo
 * @since pro3
 */

public class JavaProgrammer {
    public static void main(String[] args) {
        //父类引用指向子类对象
        JdbcInterface jdbc1 = new MySqlCompany();
        JdbcInterface jdbc2 = new OracleCompany();

        jdbc1.getConnetion();
        jdbc2.getConnetion();

    }
}
```

```java
package com.study.yy;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

/**
 * @author YoMo
 * @since pro3 1.0
 */

public class JdbcDemo01 {
    public static void main(String[] args) {
        try {
            //注册驱动
            DriverManager.registerDriver(new com.mysql.cj.jdbc.Driver());

            //获取连接
            String url = "jdbc:mysql://localhost:3306/test";
            String user = "root";
            String password = "123456";
            Connection connection = DriverManager.getConnection(url, user, password);

            System.out.println(connection);

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

<font color="red">server time zone value '�й���׼ʱ��' is unrecognized</font>

没有设置时区

```java
String url = "jdbc:mysql://localhost:3306/test?serverTimezone=GMT&useSSL=false";
//设置时区
serverTimezone=GMT
//关闭安全协议
useSSL=false
```

### 步骤

### 注册驱动

```java
DriverManager.registerDriver( new com.mysql.cj.jdbc.Driver());
```

### 获取连接

```java
String url = "jdbc:mysql://localhost:3306/test?serverTimezone=GMT&useSSL=false";
String user = "root";
String password = "123456";
Connection connection = DriverManager.getConnection(url, user, password);
```

### 执行Sql语句

### 插入数据

增删改都调用了`executeUpdate(sql)`方法

```java
System.out.println(" 实例化Statement对象...");
stmt = connection.createStatement();
String sql;
sql = "insert into test(id,username,age,password)values(3,'陈小二',15,'12467')";
int count = stmt.executeUpdate(sql);
System.out.println("影响了数据数据库中的多少数据："+count);
```

#### 查询数据

```java
sql = "SELECT * FROM test";
ResultSet rs = stmt.executeQuery(sql);
```

展开结果集

```java
// 展开结果集数据库
while(rs.next()){
    // 通过字段检索
    int id  = rs.getInt("id");
    String username = rs.getString("username");
    String age = rs.getString("age");
    String pwd = rs.getString("password");

    // 输出数据
    System.out.print("ID: " + id);
    System.out.print(",name " + username);
    System.out.print(",age " + age);
    System.out.print(",password " + pwd);
    System.out.print("\n");
}
```

#### 关闭流

```java
if (stmt != null){
    try {
        stmt.close();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
```

为保证资源一定能得到释放，在finally语句块中关闭资源，并且遵循从小到大一次关闭

```java
if (stmt != null){
    try {
        stmt.close();
    } catch (SQLException throwables) {
        throwables.printStackTrace();
    }
}

if (connection != null){
    try {
        connection.close();
    } catch (SQLException throwables) {
        throwables.printStackTrace();
    }
}
```

注册驱动的第二种方式

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```

创建配置文件`XXX.properties`

```properties
driverClass=com.mysql.cj.jdbc.Driver

url=jdbc:mysql://localhost:3306/test?serverTimezone=GMT&useSSL=false&characterEncoding=utf-8

user=root

password = 123456
```

调用配置文件

```java
package com.study.yy;

import java.net.URL;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ResourceBundle;

/**
 * @author YoMo
 * @since pro3 1.0
 *
 * 从数据库配置文件中获取数据库配置信息
 */

public class JdbcDemo04 {
    public static void main(String[] args) {
        Connection connection = null;
        Statement  statement = null;

        //获取数据库配置信息
        ResourceBundle resourceBundle = ResourceBundle.getBundle("jdbc");

        String driverClass = resourceBundle.getString("driverClass");

        String url = resourceBundle.getString("url");

        String user = resourceBundle.getString("user");

        String password = resourceBundle.getString("password");

        //注册驱动
        try {
            Class.forName(driverClass);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

        try {
            connection = DriverManager.getConnection(url,user,password);

            statement = connection.createStatement();

            String sql;
            sql = "update test set username='王小二' where username='陈小二'";

            int i = statement.executeUpdate(sql);

            System.out.println("修改了数据的数据量"+ i );

        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }finally {
            if (statement != null){
                try {
                    statement.close();
                } catch (SQLException throwables) {
                    throwables.printStackTrace();
                }
            }

            if (connection != null){
                try {
                    connection.close();
                } catch (SQLException throwables) {
                    throwables.printStackTrace();
                }
            }
        }

    }
}
```

好处：之后就不用一个一个去配置
