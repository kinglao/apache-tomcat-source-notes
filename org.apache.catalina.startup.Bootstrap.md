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
#### initClassLoaders()
主要创建了三个ClassLoader：commonLoader、catalinaLoader、sharedLoader
> CatalinaProperties
默认从${catalina.base}/conf/catalina.properties文件中读取属性。可以通过"catalina.config"属性名设置属性。

1. commonLoader
parent ClassLoader为null。<br/>
从CatalinaProperties中读取"common.loader"属性值。默认情况下值为：</br>`"${catalina.base}/lib","${catalina.base}/lib/*.jar","${catalina.home}/lib","${catalina.home}/lib/*.jar"`。</br>
将里面的${属性名}替换为对应的值：</br>`"E:\apache-tomcat-9.0.0.M22-src\apache-tomcat-9.0.0.M22-src\output\build/lib","E:\apache-tomcat-9.0.0.M22-src\apache-tomcat-9.0.0.M22-src\output\build/lib/*.jar","E:\apache-tomcat-9.0.0.M22-src\apache-tomcat-9.0.0.M22-src\output\build/lib","E:\apache-tomcat-9.0.0.M22-src\apache-tomcat-9.0.0.M22-src\output\build/lib/*.jar"`。</br>
可以看到其实是四个目录。然后进行分类：
* RepositoryType.URL     new URL()没有异常
* RepositoryType.GLOB    \*.jar   通配符
* RepositoryType.JAR     .jar     单个jar包
* RepositoryType.DIR     其他     目录
