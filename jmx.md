Tomcat的entry point是：Bootstrap.java
```java
/**
 * Main method and entry point when starting Tomcat via the provided
 * scripts.
 *
 * @param args Command line arguments to be processed
 */
public static void main(String args[]) {
}
```

启动web项目的时候，首先从`Bootstrap.java`一直到`StandardServer.initInternal()`
```
"main@1" prio=5 tid=0x1 nid=NA runnable
  java.lang.Thread.State: RUNNABLE
	  at org.apache.catalina.core.StandardServer.initInternal(StandardServer.java:806)
	  at org.apache.catalina.util.LifecycleBase.init(LifecycleBase.java:136)
	  - locked <0x910> (a org.apache.catalina.core.StandardServer)
	  at org.apache.catalina.startup.Catalina.load(Catalina.java:622)
	  at org.apache.catalina.startup.Catalina.load(Catalina.java:645)
	  at sun.reflect.NativeMethodAccessorImpl.invoke0(NativeMethodAccessorImpl.java:-1)
	  at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	  at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	  at java.lang.reflect.Method.invoke(Method.java:498)
	  at org.apache.catalina.startup.Bootstrap.load(Bootstrap.java:309)
	  at org.apache.catalina.startup.Bootstrap.main(Bootstrap.java:492)
```
在`StandardServer.initInternal()`中，创建并注册
```java
protected void initInternal() throws LifecycleException {
    super.initInternal();

    // Register global String cache
    // Note although the cache is global, if there are multiple Servers
    // present in the JVM (may happen when embedding) then the same cache
    // will be registered under multiple names
    onameStringCache = register(new StringCache(), "type=StringCache");

    // Register the MBeanFactory
    MBeanFactory factory = new MBeanFactory();
    factory.setContainer(this);
    onameMBeanFactory = register(factory, "type=MBeanFactory");
// ...
}
```

```java
// MBeanFactory.java
public String createStandardContext(String parent, String path, String docBase, boolean xmlValidation, boolean xmlNamespaceAware){
    // ...
    mserver.invoke(deployer, "manageApp", new Object[] {context}, new String[] {"org.apache.catalina.Context"});
    // ...
}
public void removeContext(String contextName) throws Exception {
    // ...
    mserver.invoke(deployer,"unmanageApp", new Object[] {pathStr}, new String[] {"java.lang.String"});
    // ...
}

// HostConfig.java
/**
 * Add a new Context to be managed by us.
 * Entry point for the admin webapp, and other JMX Context controllers.
 * @param context The context instance
 */
public void manageApp(Context context)  {
    // ...
    host.addChild(context);
    // ...
}

/**
 * Remove a webapp from our control.
 * Entry point for the admin webapp, and other JMX Context controllers.
 * @param contextName The context name
 */
public void unmanageApp(String contextName) {
    // ...
    if(isServiced(contextName)) {
        deployed.remove(contextName);
        host.removeChild(host.findChild(contextName));
    }
    // ...
}
```
