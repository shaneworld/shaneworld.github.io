---
title: Spring Boot 中使用 MyBatis
author: shane
date: 2024-09-27
description: Spring Boot 中使用 MyBatis 操作数据库
categories: [SpringBoot]
tags: [MyBatis, SpringBoot, SQL]
comments: true
toc: true
---

### 引入依赖

依赖如下：

 - org.mybatis
 - mysql-connector-java
 - logback-core
 - logback-classic
 - slf4j-api

### 连接数据库

在 `resources` 下创建 `mybatis-config.xml` 文件，配置 `mybatis` 连接数据库。相关配置选项参见：[MyBatis Documentation zh](https://mybatis.net.cn/getting-started.html)

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
    <environments default="development">
      <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
          <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
          <property name="url" value="jdbc:mysql:///mybatis?useSSL=false"/>
          <property name="username" value="admin"/>
          <property name="password" value="123456"/>
        </dataSource>
      </environment>
    </environments>
    <mappers>
      <!-- 加载sql的映射文件 -->
      <mapper resource="UserMapper.xml"/>
    </mappers>
  </configuration>
  ```

## 定义 pojo 类

`pojo` 中包含了所有数据库中对应的表所代表的类。

## 构建 SqlSessionFactory

在主函数中添加 `SqlSessionFactory` 实例构建代码。

```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) throws IOException {
    String resource = "mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //SpringApplication.run(DemoApplication.class, args);

    // 获取SqlSession对象
    SqlSession sqlSession = sqlSessionFactory.openSession();

    // 执行sql语句
    List<User> users = sqlSession.selectList("test.selectAll");

    System.out.println(users);

    sqlSession.close();
    }

}
```

其中 `List<User> users = sqlSession.selectList("test.selectAll");` 中的 `test.selectAll` 是在 `UserMapper.xml` 文件中定义的。

- 配置 mybatis 的时候要遵循一定的标签顺序。

### MyBatis mapper.xml 文件注意事项

- 参数占位符

    - `#{}` 会将其替换为`?`，为了防止SQL注入
    - `${}` 拼接SQL，会存在SQL注入问题
    - 使用时机：
        - 参数传递的时候：`#{}`
        - 表名或列名不固定的情况下：`${}`会存在SQL注入问题

- 参数类型：`paramenterType` 可以省略

- 特殊字符处理（比如小于号）

    - 转义字符，如小于号可以写成`&lt;`
    - CDATA区


## 条件查询

### 多条件查询

- 参数接收
    
    - 散装参数：如果方法中有多个参数，需要使用 `@Param` (SQL参数占位符名称)

    ```java
    List<Brand> selectByCondition(@Param("status")int status, @Param("companyName")String companyName, @Param("brandName")String brandName);
    ```

    - 对象参数：对象的属性名称要和参数占位符的参数类型一致

    ```java
    // 对象参数
    List<Brand> selectByCondition(Brand brand);
    ```

    - Map 集合参数

    ```java
    List<Brand> selectByCondition(Map map);
    ```

### 动态SQL

[MyBatis动态SQL](https://mybatis.net.cn/dynamic-sql.html)

- `if` 条件判断
    - `test` 逻辑表达式
- 问题
    - 恒等式，如`where 1=1`
    - 通过 `where` 标签

- 单条件动态查询
    - 使用 `choose` 和 `when` 标签实现 `switch` 的效果
    - 使用 `where` 标签或者添加 `otherwise` 标签处理不查询任何条件的情况
