想打开Tomcat日志，来看一下Digester解析过程。Tomcat使用的是jdk Logger。
```java
public class ClassLoaderLogManager extends LogManager {
}
```
读取`output/build/conf/logging.properties`配置的调用关系：
`LogFactory#getLog(Class<?> clazz)`---> </br>
`LogFactory#getInstance(Class<?> clazz)`---> </br>
`LogFactory#getInstance(String name)`---> </br>
`DirectJDKLog#getInstance(String name)`---> </br>
`DirectJDKLog#DirectJDKLog(String name)`---> </br>
`Logger#getLogger(String name)`---> </br>
`Logger#demandLogger(String name, String resourceBundleName, Class<?> caller)`---> </br>
`Logger#addLogger(Logger logger)`---> </br>
`LoggerContext#addLocalLogger(Logger logger)`---> </br>
`LoggerContext#requiresDefaultLoggers()`---> </br>
`LogManager#ensureLogManagerInitialized()`---> </br>
`LogManager#readPrimordialConfiguration()`---> </br>
`ClassLoaderLogManager#readConfiguration()重载父类`---> </br>
`ClassLoaderLogManager#readConfiguration(ClassLoader classLoader)` </br>
或者 </br>
`LogManager#getLogManager()`---> </br>
`LogManager#ensureLogManagerInitialized()`---> </br>
`LogManager#readPrimordialConfiguration()`---> </br>
`ClassLoaderLogManager#readConfiguration()重载父类`---> </br>
`ClassLoaderLogManager#readConfiguration(ClassLoader classLoader)` </br>
