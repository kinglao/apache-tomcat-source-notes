org.apache.catalina.startup.Bootstrap.java是Tomcat的入口。
## static{}
* 读取"catalina.home"属性的值。对应的目录对象(File)保存为Bootstrap#catalinaHomeFile
* 如果"catalina.home"不存在，指向系统属性${user.dir}，就是工程根目录。或者${user.dir}/../目录，如果${user.dir}下面有bootstrap.jar文件
* 读取"catalina.base"属性的值。对应的目录对象(File)保存为Bootstrap#catalinaBaseFile
* 如果"catalina.base"属性不存在，则"catalina.base"和"catalina.home"是一样的
* 将"catalina.home"和"catalina.base"的值设置为系统属性（System.setProperty）
```java
String userDir = System.getProperty("user.dir");
File f = new File(userDir, "..");
System.out.println(userDir);//E:\apache-tomcat-9.0.0.M22-src
System.out.println(f.getPath());//E:\apache-tomcat-9.0.0.M22-src\..
System.out.println(f.isDirectory());//true
```
注：windows下面，使用鼠标右键的方法是无法创建`..`的目录的
## main()

### 调用Bootstrap的init()方法
创建`Bootstrap`对象，调用`init()`方法进行初始化，将对象保存到`Bootstrap#daemon`中。
#### initClassLoaders()
主要创建了三个ClassLoader：commonLoader、catalinaLoader、sharedLoader
> CatalinaProperties
默认从${catalina.base}/conf/catalina.properties文件中读取属性。可以通过"catalina.config"属性名设置从其他文件中读取属性。

1. commonLoader
parent ClassLoader为null。<br/>
从CatalinaProperties中读取"common.loader"属性值。默认情况下值为：</br>`"${catalina.base}/lib","${catalina.base}/lib/*.jar","${catalina.home}/lib","${catalina.home}/lib/*.jar"`。</br>
将里面的${属性名}替换为对应的值：</br>`"E:\apache-tomcat-9.0.0.M22-src\apache-tomcat-9.0.0.M22-src\output\build/lib","E:\apache-tomcat-9.0.0.M22-src\apache-tomcat-9.0.0.M22-src\output\build/lib/*.jar","E:\apache-tomcat-9.0.0.M22-src\apache-tomcat-9.0.0.M22-src\output\build/lib","E:\apache-tomcat-9.0.0.M22-src\apache-tomcat-9.0.0.M22-src\output\build/lib/*.jar"`。</br>
可以看到其实是两个目录和两个通配符。然后进行分类：
* RepositoryType.URL     new URL()没有异常
* RepositoryType.GLOB    \*.jar   通配符
* RepositoryType.JAR     .jar     单个jar包
* RepositoryType.DIR     其他     目录
将目录或者文件转换为`URL`对象保存起来:
```java
private static URL buildClassLoaderUrl(File file) throws MalformedURLException {
    // Could be a directory or a file
    String fileUrlString = file.toURI().toString();
    fileUrlString = fileUrlString.replaceAll("!/", "%21/");
    return new URL(fileUrlString);
}
```
例如，转换为URL之后，对于目录：`file:/E:/apache-tomcat-9.0.0.M22-src/apache-tomcat-9.0.0.M22-src/output/build/lib/`。对于jar包：`file:/E:/apache-tomcat-9.0.0.M22-src/apache-tomcat-9.0.0.M22-src/output/build/lib/annotations-api.jar`。</br>
最后调用如下代码创建出ClassLoader。其中array保存了上面提到的所有`URL`对象。
```java
        return AccessController.doPrivileged(
                new PrivilegedAction<URLClassLoader>() {
                    @Override
                    public URLClassLoader run() {
                        if (parent == null)
                            return new URLClassLoader(array);
                        else
                            return new URLClassLoader(array, parent);
                    }
                });
```
`new URLClassLoader()`可能会抛出`SecurityException`异常（if a security manager exists and its checkCreateClassLoader method doesn't allow creation of a class loader.）。使用`AccessController.doPrivileged`可以避免做权限检查。参考：http://www.blogjava.net/DLevin/archive/2012/11/02/390637.html</br>
2. catalinaLoader
同上。对应属性名"server.loader"，parent ClassLoader是commonLoader。默认情况下属性值为空，则catalinaLoader的值为其parent，即commonLoader
3. sharedLoader
同上。对应属性名"shared.loader"，parent ClassLoader是commonLoader。默认情况下属性值为空，则sharedLoader的值为其parent，即commonLoader
#### Thread.currentThread().setContextClassLoader(catalinaLoader);

#### SecurityClassLoad.securityClassLoad(catalinaLoader);
如果`System.getSecurityManager()`不为null时，会调用`catalinaLoader.loadClass()`方法，加载一系列的类。默认情况是null

#### init()方法最后一步
1. 加载类`Class<?> startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina")`
2. 创建实例`Object startupInstance = startupClass.newInstance();`
3. 通过反射，调用的`Catalina#setParentClassLoader()`方法。方法的参数是`sharedLoader`
4. 将`startupInstance`保存到`Bootstrap#catalinaDaemon`中

### Bootstrap#main()的args[]参数列表
根据Bootstrap#main()的args[]参数列表分别调用`Bootstrap#daemon`对应的方法。里面使用反射的方式，实际上调用的是`Bootstrap#catalinaDaemon`对应的方法。所以`Bootstrap`类可以看做是`Catalina`类的代理。
#### setAwait(boolean await)
调用`Catalina`类的`setAwait(boolean b)`，对`Catalina#await`域赋值
#### void load(String[] arguments)
调用`Catalina`类的`load()`方法，有两种重载形式：`void load()`和`load(String args[])`.

解析${catalina.base}/build/conf/server.xml文件
Digester:
startDocument()
startElement()
qName:Server
```xml
<Server port="8005" shutdown="SHUTDOWN">
```
配置了两个属性
有三个Rule：
```java
//Catalina#createStartDigester()
digester.addObjectCreate("Server", "org.apache.catalina.core.StandardServer", "className");
digester.addSetProperties("Server");
digester.addSetNext("Server", "setServer", "org.apache.catalina.Server");
```
ObjectCreateRule
从属性列表中查找"className"属性，没有设置则使用默认的`org.apache.catalina.core.StandardServer`.使用反射创建`StandardServer`实例，并保存到Digester的`ArrayStack<Object> stack`域中，Digester的`root`域保存的是`Catalina`实例。

