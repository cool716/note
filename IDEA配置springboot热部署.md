# IDEA配置springboot热部署

分为两部分：一，实现页面热部署；二，修改类文件后，idea不会自动编译，得修改idea设置

## 一，实现页面热部署

1，导入jar包

```pom.xml
<!--  热部署  -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <!-- optional=true, 依赖不会传递, 该项目依赖devtools;
              之后依赖boot项目的项目如果想要使用devtools, 需要重新引入 -->
            <optional>true</optional>
            <scope>true</scope>
        </dependency>
```

2，配置application.yml,  实现页面的热部署

```application.yml
spring: 
  ##热部署工具
  devtolls:
    restart:
      enabled: true  # 开启热部署
      additional-paths: src/main/java #重启目录
      exclude: templates/**  #classpath目录下的templates文件夹内容修改不重启
```

## 二，修改类文件后，idea不会自动编译，得修改idea设置

（1）File-Settings-(buid,Execution,Deployment)-Compiler-Build Project automatically

<img src="C:\Users\O20110004\Desktop\tyopra图片\热部署.png" alt="热部署" style="zoom:67%;" />

（2）ctrl + shift + alt + /,选择Registry,勾上 Compiler autoMake allow when app running



<img src="C:\Users\O20110004\Desktop\tyopra图片\02.png" alt="02" style="zoom:67%;" />