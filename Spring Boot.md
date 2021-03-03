# Spring Boot

## Spring Boot 快速搭建

​	优势：Spring Boot 统一了依赖的版本，解决了版本冲突问题，也简化了繁琐的配置文件。

​	快速搭建Spring Boot 项目步骤：

​		1、在 pom.xml 文件中 引入 Spring Boot 依赖以及对应的版本号；（用于 Spring Boot 包的版本号统一）

​		2、引入 Spring Boot - web 包；（该包中包含了 java web 开发中 常用的jar包）

​		3、简单配置端口号以及项目访问前缀

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Maven配置 -->
    <groupId>org.example</groupId>
    <artifactId>SpringBootTest</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>SpringBootTest</name>
    <description>Demo project for Spring Boot</description>

    <!-- 声明JDK版本 -->
    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!-- Spring Boot 依赖 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <!-- 依赖包 -->
    <dependencies>

        <!-- web相关包 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 测试包依赖 -->
        <dependency>
        <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

    </dependencies>

    <!-- Spring Boot打包依赖 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

### application.yml

```yaml
server:
  # 设置端口号
  port: 8081
  servlet:
  # 设置访问前缀
    context-path: /SpringBootTest
```

## Spring Boot 集成Mybatis

​	Mybatis特点：动态SQL、映射结果集

​	集成Mybatis步骤：

​		1、pom.xml 中 引入 Spring Boot 集成 Mybatis 依赖 以及 MySQL驱动依赖。

​		2、在 application.yml 中配置数据源信息 以及 dao 层包扫描路径 和 mapper文件路径

### pom.xml

```xml
<!--Spring Boot 整合 Mybatis -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

### application.yml

```yaml
spring:
  # 设置数据源
  datasource:
  	# 设置数据源 种类、IP、端口、数据库名称
    url: jdbc:mysql://10.58.241.10:3306/testspringboot?useunicode=true&characterEncoding=utf8
    # 数据源账号
    username: root
    # 数据源密码
    password: Wopt54321
    # 数据源驱动
    driver-class-name: com.mysql.jdbc.Driver
    
mybatis:
  # dao层接口扫描
  type-aliases-package: com.liay.dao
  # mapper文件路径
  mapper-locations: classpath:mybatis/mapper/*.xml
```

## Spring Boot 集成 Druid

​	Druid概念：连接池，sql监控。

​	集成Druid步骤：

​		1、在 pom.xml 中 引入druid 依赖

​		2、在 application.yml 中 替换原有的 Mybatis 数据源配置

### pom.xml

```xml
<!--阿里druid数据库连接池 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.10</version>
</dependency>
```

### application.yml

```yaml
spring:
  # 数据源
  datasource:
  	  # 数据源类型
      type: com.alibaba.druid.pool.DruidDataSource
      # 驱动类型
      driver-class-name: com.mysql.cj.jdbc.Driver
      # 设置数据源 种类、IP、端口、数据库名称 
      url: jdbc:mysql://10.58.241.10:3306/testspringboot?useUnicode=true&characterEncoding=UTF-8&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC
      # 数据源账号
      username: root
      # 数据源密码
      password: Wopt54321
      druid:
        #初始化连接池的连接数量 大小，最小，最大
        initial-size: 5
        min-idle: 5
        max-active: 20
        #配置获取连接等待超时的时间
        max-wait: 60000
        #配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
        time-between-eviction-runs-millis: 60000
        # 配置一个连接在池中最小生存的时间，单位是毫秒
        min-evictable-idle-time-millis: 30000
        validation-query: SELECT 1 FROM DUAL
        test-while-idle: true
        test-on-borrow: true
        test-on-return: false
```



## Spring Boot 集成 Mybatis 使用多个数据源

​	优势：可以同时使用多个数据源，可以是相同数据源，也可能是不同数据源。

​	配置多个数据源步骤：

​		1、在 pom.xml 中 引入 Mybatis 依赖 以及 Mysql驱动包

​		2、在 application.yml 中 配置多个数据源信息

​		3、在 config 包下 建立对应的数据源配置信息

​		4、创建对应的 mapper接口 与 mapper文件

### 配置结构

![多个数据源配置结构](D:\资料\笔记\Spring Boot\images\多个数据源配置结构.jpg)

### pom.xml

​	与集成单个数据源使用依赖相同。

```xml
<!--Spring Boot 整合 Mybatis -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

### application.yml

​	sqlone与sqltwo是用来区分数据源的，无特殊意义。

​	使用的多个数据源 **可以相同**，**也可以不相同**，只需要修改对应的 **driver-class-name** 与 **jdbc-url** 即可。

```yaml
spring:
  datasource:

    sqlone:
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: com.mysql.cj.jdbc.Driver
      jdbc-url: jdbc:mysql://10.58.241.10:3306/testspringboot?useUnicode=true&characterEncoding=UTF-8&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC
      username: root
      password: Wopt54321
      druid:
        #初始化连接池的连接数量 大小，最小，最大
        initial-size: 5
        min-idle: 5
        max-active: 20
        #配置获取连接等待超时的时间
        max-wait: 60000
        #配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
        time-between-eviction-runs-millis: 60000
        # 配置一个连接在池中最小生存的时间，单位是毫秒
        min-evictable-idle-time-millis: 30000
        validation-query: SELECT 1 FROM DUAL
        test-while-idle: true
        test-on-borrow: true
        test-on-return: false

    sqltwo:
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: com.mysql.cj.jdbc.Driver
      jdbc-url: jdbc:mysql://10.58.241.10:3306/testspringboottwo?useUnicode=true&characterEncoding=UTF-8&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC
      username: root
      password: Wopt54321
      druid:
        #初始化连接池的连接数量 大小，最小，最大
        initial-size: 5
        min-idle: 5
        max-active: 20
        #配置获取连接等待超时的时间
        max-wait: 60000
        #配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
        time-between-eviction-runs-millis: 60000
        # 配置一个连接在池中最小生存的时间，单位是毫秒
        min-evictable-idle-time-millis: 30000
        validation-query: SELECT 1 FROM DUAL
        test-while-idle: true
        test-on-borrow: true
        test-on-return: false
```

### DataSourceConfig.java

​	配置对应数据源数量的配置文件。

​	以下以两个数据源为例，一个主数据库，一个从数据库；主数据库中使用@Primary注解，从数据库中不需使用。

#### DataSourceOneConfig

```java
package com.liay.common.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;

@Configuration
/**
 * basePackages： 这里的路径与对应的 dao 层路径对应
 */
@MapperScan(basePackages = "com.liay.dao.sqlone", sqlSessionTemplateRef  = "test1SqlSessionTemplate")
public class DataSourceOneConfig {

    @Bean(name = "test1DataSource")
    /**
     * prefix： 这里的名称需与yml中设置的名称对应
     */
    @ConfigurationProperties(prefix = "spring.datasource.sqlone")
    @Primary
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "test1SqlSessionFactory")
    @Primary
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("test1DataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        /**
         * 这里的路径与对应的 mapper.xml 路径对应
         */
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/sqlone/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "test1TransactionManager")
    @Primary
    public DataSourceTransactionManager testTransactionManager(@Qualifier("test1DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "test1SqlSessionTemplate")
    @Primary
    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("test1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

#### DataSourceTwoConfig.java

```java
package com.liay.common.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;

@Configuration
/**
 * basePackages： 这里的路径与对应的 dao 层路径对应
 */
@MapperScan(basePackages = "com.liay.dao.sqltwo", sqlSessionTemplateRef  = "test2SqlSessionTemplate")
public class DataSourceTwoConfig {

    @Bean(name = "test2DataSource")
    /**
     * prefix： 这里的名称需与yml中设置的名称对应
     */
    @ConfigurationProperties(prefix = "spring.datasource.sqltwo")
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "test2SqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("test2DataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        /**
         * 这里的路径与对应的 mapper.xml 路径对应
         */
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/sqltwo/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "test2TransactionManager")
    public DataSourceTransactionManager testTransactionManager(@Qualifier("test2DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "test2SqlSessionTemplate")
    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("test2SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

### mapper

​	创建对应的 dao 层接口，与对应的 mapper.xml 文件。

​	以下以两个数据源为例。

#### sqlone

##### StudentMapper.java

```java
package com.liay.dao.sqlone;

import com.liay.pojo.Student;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public interface StudentMapper {

    List<Student> getAllStudent();
}

```

##### StudentMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.liay.dao.sqlone.StudentMapper">

    <select id="getAllStudent" resultType="com.liay.pojo.Student">
        SELECT * FROM student
    </select>

</mapper>
```

#### sqltwo

##### UserMapper.java

```java
package com.liay.dao.sqltwo;

import com.liay.pojo.User;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public interface UserMapper {

    List<User> getAllUser();
}

```

##### UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.liay.dao.sqltwo.UserMapper">

    <select id="getAllUser" resultType="com.liay.pojo.User">
        SELECT * FROM user
    </select>

</mapper>
```