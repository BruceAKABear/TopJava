# auth-server 搭建

我认为用户的登录注册应该放在 auth 这个服务中。因为如果你的项目以后成了，其他网站可以使用你的项目进行第三方登录。

## 1. pom 依赖

```java
<!--security oauth2已经包含了security，所以别在引入sercurity-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>

```

## 2. 集中配置认证服务器和 spring security

### 2.1 spring-security 配置

```java


```

### 2.2 oauth2.0 认证服务主要配置

1. 出现postman 401的情况我的情况是oauth2本身的安全策略没配置好，导致获取token的时候401
