# SpringBoot配置多数据源

**实现效果：**

一个SpringBoot项目，同时连接两个数据库：比如一个是pgsql数据库，一个是oracle数据库（啥数据库都一样，连接两个同为oracle的数据库，或两个不同的数据库，只需要更改对应的**driver-class-name**和**jdbc-url**等即可）

```
server:
  port: 7101
spring:
  jpa:
    show-sql: true
  datasource:
    test1:
      driver-class-name: org.postgresql.Driver
      jdbc-url: jdbc:postgresql://127.0.0.1:5432/test  #测试数据库
      username: root
      password: root

    test2:
      driver-class-name: oracle.jdbc.driver.OracleDriver
      jdbc-url: jdbc:oracle:thin:@127.0.0.1:8888:orcl  #测试数据库
      username: root
      password: root
```

### 1、修改application.yml，添加一个数据库连接配置

  在数据源前加上数据源别名，如： mysql1， mysql2，

```
spring：
 datasource:
    #url: jdbc:mysql://10.58.241.10:3306/testspringboot?serverTimezone=GMT%2B8
    #username: root
    #password: Wopt54321
    #driver-class-name: com.mysql.cj.jdbc.Driver
    mysql1:
      jdbc-url: jdbc:mysql://10.58.241.10:3306/testspringboot?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8&useSSL=false
      username: root
      password: Wopt54321
      driver-class-name: com.mysql.cj.jdbc.Driver
      type: com.alibaba.druid.pool.DruidDataSource
      druid:
        #初始化大小
        initialSize: 5
        #最小值
        minIdle: 5
        #最大值
        maxActive: 20
        #最大等待时间，配置获取连接等待超时，时间单位都是毫秒ms
        maxWait: 60000
        #配置间隔多久才进行一次检测，检测需要关闭的空闲连接
        timeBetweenEvictionRunsMillis: 60000
        #配置一个连接在池中最小生存的时间
        minEvictableIdleTimeMillis: 300000
        validationQuery: SELECT 1 FROM DUAL
        testWhileIdle: true
        testOnBorrow: false
        testOnReturn: false
        poolPreparedStatements: true
        # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，
        #'wall'用于防火墙，SpringBoot中没有log4j，我改成了log4j2
        filters: stat,wall,log4j2
        #最大PSCache连接
        maxPoolPreparedStatementPerConnectionSize: 20
        useGlobalDataSourceStat: true
        # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
        connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
        # 配置StatFilter
        web-stat-filter:
          #默认为false，设置为true启动
          enabled: true
          url-pattern: "/*"
          exclusions: "*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*"
        #配置StatViewServlet
        stat-view-servlet:
          url-pattern: "/druid/*"
          #允许那些ip
          allow: 127.0.0.1
          login-username: admin
          login-password: 123456
          #禁止那些ip
          deny: 192.168.1.102
          #是否可以重置
          reset-enable: true
          #启用
          enabled: true
    mysql2:
      jdbc-url: jdbc:mysql://10.58.241.10:3306/testspringboottwo?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8&useSSL=false
      username: root
      password: Wopt54321
      driver-class-name: com.mysql.cj.jdbc.Driver
      type: com.alibaba.druid.pool.DruidDataSource
      druid:
        #初始化大小
        initialSize: 5
        #最小值
        minIdle: 5
        #最大值
        maxActive: 20
        #最大等待时间，配置获取连接等待超时，时间单位都是毫秒ms
        maxWait: 60000
        #配置间隔多久才进行一次检测，检测需要关闭的空闲连接
        timeBetweenEvictionRunsMillis: 60000
        #配置一个连接在池中最小生存的时间
        minEvictableIdleTimeMillis: 300000
        validationQuery: SELECT 1 FROM DUAL
        testWhileIdle: true
        testOnBorrow: false
        testOnReturn: false
        poolPreparedStatements: true
        # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，
        #'wall'用于防火墙，SpringBoot中没有log4j，我改成了log4j2
        filters: stat,wall,log4j2
        #最大PSCache连接
        maxPoolPreparedStatementPerConnectionSize: 20
        useGlobalDataSourceStat: true
        # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
        connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
        # 配置StatFilter
        web-stat-filter:
          #默认为false，设置为true启动
          enabled: true
          url-pattern: "/*"
          exclusions: "*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*"
        #配置StatViewServlet
        stat-view-servlet:
          url-pattern: "/druid/*"
          #允许那些ip
          allow: 127.0.0.1
          login-username: admin
          login-password: 123456
          #禁止那些ip
          deny: 192.168.1.102
          #是否可以重置
          reset-enable: true
          #启用
          enabled: true
```

### **2、使用代码进行数据源注入，和扫描dao层路径**（以前是在yml文件里配置mybatis扫描dao的路径）

新建config包，包含数据库1和数据库2的配置文件

（1）第一个数据库作为主数据库，项目启动默认连接此数据库，　　**主数据库都有 @Primary注解，从数据库都没有**

　　DataSource1Config.java

```
package com.wist.config;

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
@MapperScan(basePackages = "com.wist.dao.one", sqlSessionTemplateRef  = "test1SqlSessionTemplate")
public class DataSource1Config {

    @Bean(name = "test1DataSource")
    @ConfigurationProperties(prefix = "spring.datasource.mysql1")
    @Primary
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "test1SqlSessionFactory")
    @Primary
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("test1DataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/one/*.xml"));
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

（2）第二个数据库作为从数据库

　　DataSource2Config.java

```
package com.wist.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;

@Configuration
@MapperScan(basePackages = "com.wist.dao.two", sqlSessionTemplateRef  = "test2SqlSessionTemplate")
public class DataSource2Config {

    @Bean(name = "test2DataSource")
    @ConfigurationProperties(prefix = "spring.datasource.mysql2")
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "test2SqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("test2DataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/two/*.xml"));
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

### **3、 在dao文件夹下，新建one和two两个包，分别放两个不同数据库的dao层文件**

<img src="C:\Users\O20110004\Desktop\tyopra图片\screenshot_20201112_151252.png" alt="screenshot_20201112_151252" style="zoom:67%;" />

```StudentDao
package com.wist.dao.one;

import com.wist.pojo.Student;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;
@Mapper
public interface StudentDao {
    List<Student> findAll();
    Student findById(Long id);
    int updateStudent(Student stu);
}

```

```UserDao
package com.wist.dao.two;

import com.wist.pojo.User;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;
@Mapper
public interface UserDao {
    List<User> findAllUser();
    User findUserById(Long id);
    int updateUser(User user);
}

```

### **4、 在resource下的mapper新建one和two两个文件夹，分别放入对应dao层的xml文件**



<img src="C:\Users\O20110004\Desktop\tyopra图片\screenshot_20201112_151620.png" alt="screenshot_20201112_151620" style="zoom:67%;" />

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.wist.dao.one.StudentDao">
    <select id="findAll" resultType="com.wist.pojo.Student">
        select  * from `student`
    </select>

    <select id="findById" resultType="com.wist.pojo.Student">
        select  * from student where id=#{id}
    </select>
    <update id="updateStudent" parameterType="com.wist.pojo.Student">
        UPDATE `testspringboot`.`student`
        SET  `name` = #{name},`age` = #{age}, `addr` = #{addr}
        WHERE `id` = #{id};
    </update>
</mapper>
```

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.wist.dao.two.UserDao">
    <select id="findAllUser" resultType="com.wist.pojo.User">
        SELECT `id`,`name`, `age` FROM `user`
    </select>
    <select id="findUserById" resultType="com.wist.pojo.User">
        select  * from `user` where id=#{id}
    </select>
    <update id="updateUser" parameterType="com.wist.pojo.User">
        UPDATE `user`
        SET  `name` = #{name},`age` = #{age}
        WHERE `id` = #{id};
    </update>
</mapper>

```

### **5、测试**

在controller文件里，注入两个数据库的dao，分别查询数据

```
package com.wist.controller;

import com.wist.dao.one.StudentDao;
import com.wist.dao.two.UserDao;
import com.wist.pojo.Student;
import com.wist.pojo.User;
import com.wist.utils.JsonResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.ModelAndView;

import java.util.List;

@RestController
public class StudentController {
    @Autowired
    private StudentDao studentDao;
    @Autowired
    private UserDao userDao;

    @RequestMapping("/student/studentList.htm")
    public ModelAndView studentAll(){

        ModelAndView mav = new ModelAndView();
        mav.setViewName("student/studentList");
        return mav;
    }

    @RequestMapping("/findAll")
    public JsonResult findAll(){
        List<Student> studentList=this.studentDao.findAll();
        return new JsonResult(0,studentList);
    }
    @RequestMapping("/findAllUser")
    public List<User> findAllUser(){
        System.out.println("userDao"+"==============="+userDao);
        List<User> userList=this.userDao.findAllUser();
        return userList;
    }
}

```

