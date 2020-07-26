# Java 中类加载机制

Java 类的生命周期：加载、验证、准备、解析、初始化、使用、卸载 七个阶段

加载：通过全限定名，字节码加载以后放在方法区

1. 根加载器 bootstrap，c++语言写的。父类加载器是 null，也就是说没有父类加载器。加载 java_home/lib/下的 rt.jar\resources.jar\charsets.jar 和 class 等
2. 扩展加载器 ExtClassLoader，父类加载器是 bootstrap。 %JRE_HOME/lib/ext 目录下的 jar 和 class 等
3. 系统加载器 AppClassLoader。Java 类,继承自 URLClassLoader 系统类加载器,对应的文件是应用程序 classpath 目录下的所有 jar 和 class 等
4. 自定义类加载器，继承 classloader 类，重写 findclass 方法

## 类加载器的双亲委派机制

类加载时，类加载器请求父类加载器加载**父概念不是继承，而是复用父加载器中代码**

1. 避免重复加载

2. 安全性因素
   加载时会去查找是否已经加载相同类，防止加载例如第三方 object 等类

## 自定义类加载器

1. 继承 ClassLoader
2. 重新 findClass 方法
