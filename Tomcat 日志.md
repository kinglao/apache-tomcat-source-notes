想打开Tomcat日志，来看一下Digester解析过程。Tomcat使用的是jdk Logger。
```java
public class ClassLoaderLogManager extends LogManager {
}
```
读取`output/build/conf/logging.properties`配置的调用关系：
`ClassLoaderLogManager构造函数`--->`LogManager构造函数`--->`LogManager#ensureLogManagerInitialized()`--->`LogManager#readPrimordialConfiguration()`--->`ClassLoaderLogManager#readConfiguration()重载父类`--->`ClassLoaderLogManager#readConfiguration(ClassLoader classLoader)`
