# spring 面试题集合

## 1. 什么是 IOC？

## 2. ioc 和 di 有什么区别？

## 3. bean 生命周期回调

一共有 3 中，xml 配置、实现接口、使用注解@postConstruct

## 2. 什么是 AOP？

## 3. sping 如何解决循环依赖

## 4. spring 的缓存

spring 创建 bean 的大致流程是，getBean 到 doGetBean 到 createBean 到 doCreateBean 方法，然后返回 bean.在 doCreateBean 的时候大概分为三不，createBeanInstance 创建 bean 的对象，然后 populateBean 进行属性封装，然后调用 initialBean 初始化 bean.

循环依赖主要发生在 populateBean 方法中，spring 采用三级缓存解决。一级 singletonObjects 单例缓存池 concurrenthashmap，二级是 earlySingletonObjects 是 hashmap，三级是 singletonFactories 也是 hahsmap

singletonObjects：用于存放完全初始化好的 bean，从该缓存中取出的 bean 可以直接使用
earlySingletonObjects：提前曝光的单例对象的 cache，存放原始的 bean 对象（尚未填充属性），用于解决循环依赖
singletonFactories：单例对象工厂的 cache，存放 bean 工厂对象，用于解决循环依赖

## 5. bean 的作用域?

singleton 默认单例，prototype

**注意：**单例 bean 引用 prototype bean 的时候，prototype 将会失去意义、将会失效。单例 bean 容器只会设置一次 bean 的属性，只会设置一次，因此 prototype 的 bean 就会失效

解决方法：

1. 实现 applicationcontextaware，重写 setApplicationContext 方法，重新获取 bean,.getbean 每次重新获取 bean

2. 在单例 bean 中使用@LookUp 注解，将 get 方法修改

```java

@LookUp
public abstract Dao getDao();

//调用的画要必须使用getDao()触发

```

## 6. spring 中用了哪些设计模式？

## 7. spring 中的循环依赖？为什么用三级缓存？

构造注入、prototype 两者循环依赖无法解决！

## springboot 自动配置原理

1. springboootApplication 注解是组合注解，里面包含了@enableAutoConfiguration 注解，这个@EnableAutoConfiguration 注解也是一个组合注解。里面使用@import 注解导入了 autoconfigurationimportselector.clss 这个类，主要的自动配置类的加载主要是在这里加载的。实现了 DeferredImportSelector 接口，重写了 selectImports方法，这个方法起始就是去扫描 boot-autoconfigura 包的 meta-inf 目录下的 spring.factories 这个文件，这这个文件中，key 以 EnableAutoConfiguration 包名，value 是所有的 xxxxautoconfigura 自动配置类，然后遍历所有的配置类包名，加载所有的配置类，根据配置类上 conditiononXXX 这类注解决定要不要将配置类加载到 spring 容器中。这就实现了自动配置

```java
//1. springboot中的apringbootapplication注解是组合注解

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

//2. AutoConfigurationPackage默认扫描当前包及子包
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```

### springmvc javaconfig 如何配置

实现 webapplicationinnitializer 接口重写 onstart 方法，这是 servlet3.0 新规范
