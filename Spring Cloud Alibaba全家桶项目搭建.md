# Spring Cloud Alibaba全家桶项目搭建

## 一、概述



## N、内容项目

- Nacos的配置

在Nacos中添加配置项时，其中的Data Id值的设置，需要可以使用项目的名称，需要加上文件扩展名

![image-20220328171759669](C:\Users\WangWei\AppData\Roaming\Typora\typora-user-images\image-20220328171759669.png)

Naocs中的配置信息示例如下：

``` yaml
useLocalCache: true
customer:
  name: 周秉义
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db_scali?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: root

mybatis:
  mapper-locations: classpath:mapping/*Mapper.xml
  type-aliases-package: com.example.mybatisdemo.entity # 使这个配置，在Mapper.xml中就可以不再使用实体类的全名了

logging:
  level:
    root: info
```

