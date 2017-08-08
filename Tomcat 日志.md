想打开Tomcat日志，来看一下Digester解析过程。Tomcat使用的是jdk Logger。
```java
public class ClassLoaderLogManager extends LogManager {
}
```
读取`output/build/conf/logging.properties`配置的调用关系：</br>
`LogFactory#getLog(Class<?> clazz)`---> </br>
`LogFactory#getInstance(Class<?> clazz)`---> </br>
`LogFactory#getInstance(String name)`---> </br>
`DirectJDKLog#getInstance(String name)`---> </br>
`DirectJDKLog#DirectJDKLog(String name)`---> </br>
`Logger#getLogger(String name)`---> </br>
`Logger#demandLogger(String name, String resourceBundleName, Class<?> caller)`---> </br>
或者</br>
`LogManager#demandLogger(String name, String resourceBundleName, Class<?> caller)`
`LogManager#addLogger(Logger logger)`---> </br>
`LoggerContext#addLocalLogger(Logger logger)`---> </br>
`LoggerContext#requiresDefaultLoggers()`---> </br>
`LogManager#ensureLogManagerInitialized()`---> </br>
`LogManager#readPrimordialConfiguration()`---> </br>
`LogManager#readConfiguration()`---> </br>
`LogManager#readConfiguration(ClassLoader classLoader)` </br>
或者 </br>
`LogManager#getLogManager()`---> </br>
`LogManager#ensureLogManagerInitialized()`---> </br>
`LogManager#readPrimordialConfiguration()`---> </br>
`LogManager#readConfiguration()`---> </br>
`LogManager#readConfiguration(ClassLoader classLoader)` </br>

在`LogManager`的`static{}`中会创建'LogManager'对象：
```java
    static {
        manager = AccessController.doPrivileged(new PrivilegedAction<LogManager>() {
            @Override
            public LogManager run() {
                LogManager mgr = null;
                String cname = null;
                try {
                    cname = System.getProperty("java.util.logging.manager");
                    if (cname != null) {
                        try {
                            Class<?> clz = ClassLoader.getSystemClassLoader()
                                    .loadClass(cname);
                            mgr = (LogManager) clz.newInstance();
                        } catch (ClassNotFoundException ex) {
                            Class<?> clz = Thread.currentThread()
                                    .getContextClassLoader().loadClass(cname);
                            mgr = (LogManager) clz.newInstance();
                        }
                    }
                } catch (Exception ex) {
                    System.err.println("Could not load Logmanager \"" + cname + "\"");
                    ex.printStackTrace();
                }
                if (mgr == null) {
                    mgr = new LogManager();
                }
                return mgr;

            }
        });
    }
```
可以看到，如果我们指定了`java.util.logging.manager`属性值，则会根据指定的类创建`LogManager`实例，否则会调用默认的`new LogManager()`构造函数，而默认的`LogManager`会读取的文件`logging.properties`位于：${java.home}/lib/logging.properties。例如`D:\Program Files\Java\jdk1.8.0_91\jre\lib\logging.properties`。我们想要读取Tomcat自己的`logging.properties`，则需要更改配置`-Dcatalina.home="E:\apache-tomcat-9.0.0.M22-src\apache-tomcat-9.0.0.M22-src\output\build" -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.util.logging.config.file=E:\apache-tomcat-9.0.0.M22-src\apache-tomcat-9.0.0.M22-src\output\build\conf\logging.properties`。这样的话，如果我们想看类`Digester`的输出日志，需要在`logging.properties`里面配置：`org.apache.tomcat.util.digester.Digester.level=FINE`</br>

` -Dorg.apache.juli.ClassLoaderLogManager.debug=true`的作用是如果设置为`true`，会通过`System.err.println()`输出错误信息。</br>

参考：</br>
http://loveshisong.cn/%E7%BC%96%E7%A8%8B%E6%8A%80%E6%9C%AF/2015-06-04-java%E6%97%A5%E5%BF%97(%E4%BA%8C)JUL%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.html
