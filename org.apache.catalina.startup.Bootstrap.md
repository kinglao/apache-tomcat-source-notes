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
注：windows下面，使用鼠标右键的方法是无法创建..的目录的
## main()
### 调用Bootstrap的init()方法
#### initClassLoaders()
