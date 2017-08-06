1. 下载Tomcat源码，使用的是apache-tomcat-9.0.0.M17
2. 默认使用ant构建。好像ant是由tomcat的作者开发的
3. 使用maven添加依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>groupId</groupId>
    <artifactId>apache-tomcat-9.0.0.M17-src</artifactId>
    <version>1.0-SNAPSHOT</version>
        
    <dependencies>
        <dependency>
            <groupId>ant</groupId>
            <artifactId>ant</artifactId>
            <version>1.7.0</version>
        </dependency>
        <dependency>
            <groupId>ant</groupId>
            <artifactId>ant-apache-log4j</artifactId>
            <version>1.6.5</version>
        </dependency>
        <dependency>
            <groupId>ant</groupId>
            <artifactId>ant-commons-logging</artifactId>
            <version>1.6.5</version>
        </dependency>
        <dependency>
            <groupId>wsdl4j</groupId>
            <artifactId>wsdl4j</artifactId>
            <version>1.6.2</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/javax.xml/jaxrpc-api -->
        <dependency>
            <groupId>javax.xml</groupId>
            <artifactId>jaxrpc-api</artifactId>
            <version>1.1</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.eclipse.jdt.core.compiler/ecj -->
        <dependency>
            <groupId>org.eclipse.jdt.core.compiler</groupId>
            <artifactId>ecj</artifactId>
            <version>4.6.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/junit/junit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.easymock/easymock -->
        <dependency>
            <groupId>org.easymock</groupId>
            <artifactId>easymock</artifactId>
            <version>3.4</version>
        </dependency>
    </dependencies>
</project>
```
4. 有些报错：</br>
点alt+回车就弄好了

5. 运行maven compile

6. 运行ant deploy</br>
会在output目录下生成输出。output/build就是从官网上下载的文件解压结果。</br>
如果ant运行失败，可能是下载的jar包有问题。一定要让ant运行成功。build.xml会有一些错误，不用管

7. 启动</br>
-Main class: org.apache.catalina.startup.Bootstrap</br>
-Vm options: -Dcatalina.home="E:\apache-tomcat-9.0.0.M22-src\apache-tomcat-9.0.0.M22-src\output\build"。output/build目录是运行ant命令创建的，而ant配置指定该目录是在build.xml中：
```xml
<project name="Tomcat 9.0" default="deploy" basedir=".">
        <property name="tomcat.output"         value="${basedir}/output"/>
        <property name="tomcat.build"          value="${tomcat.output}/build"/>
```
basedir="."表示${basedir}属性的值是build.xml的当前目录。通过查找build.xml中对`${tomcat.build}`属性的引用，可以观察到ant是如何生成output/build目录以及下面的一系列文件。</br>
启动报错：`Error:java: Annotation processing is not supported for module cycles. Please ensure that all modules from cycle [WEB-INF,apache-tomcat-9.0.0.M17-src] are excluded from annotation processing`
有循环依赖，在project structure里面看看。只要保留一个module就行：apache-tomcat-9.0.0.M17-src </br>
还有一个问题：http://tomcat.10.x6.nabble.com/package-trailers-does-not-exist-td5064196.html. 文中作者的解决方法是：I added webapps/examples/WEB-INF/classes as a "Test Sources Root" and now it works.。我也遇到了，根本原因是需要运行build.xml中的"test-compile"，但是运行该target的时候报错：`javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target`
ant的<get>标签可以从网络上下载文件：
```xml
  <target name="downloadfile" unless="exist" depends="testexist,setproxy">
    <!-- Download the file -->
    <local name="temp.file"/>
    <mkdir dir="${base.path}"/>
    <tempfile property="temp.file" destdir="${base.path}" prefix="download-" suffix=".tmp"/>
    <get src="${sourcefile}" httpusecaches="${trydownload.httpusecaches}" dest="${temp.file}"/>
    <mkdir dir="${destdir}"/>
    <move file="${temp.file}" tofile="${destfile}"/>
  </target>        
```
下载地址配置在build.prope
```xml
base-apache.loc.1=http://www.apache.org/dyn/closer.lua?action=download&filename=
base-apache.loc.2=http://archive.apache.org/dist
base-commons.loc.1=${base-apache.loc.1}/commons
base-commons.loc.2=${base-apache.loc.2}/commons
base-tomcat.loc.1=${base-apache.loc.1}/tomcat
base-tomcat.loc.2=${base-apache.loc.2}/tomcat

base-sf.loc=http://downloads.sourceforge.net
base-maven.loc=http://repo.maven.apache.org/maven2        
```

8. 测试启动</br>
http://127.0.0.1:8080 </br>


参考：</br>
http://www.jianshu.com/p/d05ef74694f7
