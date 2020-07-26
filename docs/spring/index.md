# Spring 框架

## 1. Spring 框架的配置方式

### 1.1 xml 方式

### 1.2 注解方式

（主要看使用哪一个启动类，如果 xml 还是需要主 xml 开启注解，使用 annotation 的话已经默认开启注解了）

### 1.3 java 配置方式

## 2. @autowried 和@resource

autowire 注解默认 bytype 依据类型注入，但是，如果没有找到相应的类型，会根据属性名注入。resource 会根据属性的名进行注入当然也可以指定 type 注入，不是通过 set 方法注入

## 3. 修改 beanNameGenerater 修改注入规则

## 1. IOC

将对象的创建，对象的调用权力交给 spring 管理。目的是降低耦合度，实际上完全没有耦合度是不可能的，使用 ioc 知识尽可能第的降低耦合度。ioc 容器实质上就是 bean 工厂。

ioc 容器实际上是由一堆组件构成的，包括 singletonOnjects 单例缓存池、bean 定义 beanDifinationMap、beanPostProcessers bean 后置处理器

### 1.1 实现原理

1. 配置解析（可以是 xml 解析）

2. 工厂模式

3. 反射：反射的原理就是通过加载字节码文件，获取对象实例

### bean 实例化

1. 默认调用无参构造
2. 构造器注入、set 方法注入

DI 是依赖注入，实际上就是注入属性，当然前提是要创建对象

## 2. AOP
相较于oop面向对象来的，aop主要解决oop横切性问题。主要用在日志、事务、异常处理等。

de

连接点的集合是切点。

aspectj是切面，是一种概念。



## 3. bean 管理

spring 中有两种类型的 bean,一种是我们自己写的普通 bean，一种是 FactoryBean

   1. 普通 bean，定义什么类型就返回什么类型

   2. 工厂 bean，在配置文件中定义的类型和返回的类型可以不同

创建类，实现接口 FactoryBean

3. bean 作用域： bean 默认是单例，通过 socpe 属性设置 singleton、prototype

4. bean 的生命周期
      4.1 通过构造器创建 bean 实例
      4.2 为 bean 的属性设置值，和对其他 bean 的引用（set 方法）
      4.3 调用 bean 的初始化方法
      4.4 bean 使用
      4.5 bean 的销毁，调用 bean 的销毁方法。需要手动调用 context.close()
5. 属性自动注入
   使用 ref 属性
