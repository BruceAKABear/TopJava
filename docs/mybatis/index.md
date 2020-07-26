# mybatis

半自动 orm 框架，一级缓存默认开启时 sqlsession 级别的缓存，当 sqlsession 没有 close 的话，缓存不会清空。

## mybatis 的一级缓存的问题

### spring 中为什么失效

mybatis-spring 这个项目中，sqlsession 不整合 spring 使用的是 defaultsqlsession,整合 spring 时使用的是 sqlsessiontemplate 在模板内使用的却是 sqlsessionproxy,最后在执行 invoke 方法的时候，执行完了在 finally 代码块中会调用 closesqlsession 方法关闭 session。直接使用 mybatis，session 直接可以操作，所以相关就关，spring 无法获取 mybatis 的 sqlsession.

因为 mybatis-sring 整合包中扩展了 sqlsessiontemplate 类，在 spring 容器启动的时候注入给了 mapper 替代了 defaultsqssionfactory,sst 中的方法不是直接查询而是被代理了，在 invoke 方法中 finally 中调用了 closesqlsession.

### mybatis 一级缓存的实现原理
