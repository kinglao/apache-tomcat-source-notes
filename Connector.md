`server.xml`
```xml
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />          
```
`Catalina#createStartDigester()`
```java
digester.addRule("Server/Service/Connector",new ConnectorCreateRule());
digester.addRule("Server/Service/Connector", 
                new SetAllPropertiesRule(new String[]{"executor", "sslImplementationName", "protocol"}));
digester.addSetNext("Server/Service/Connector",
                   "addConnector","org.apache.catalina.connector.Connector");
```
`Connector`对象的创建是在`ConnectorCreateRule`类中完成的。而`SetAllPropertiesRule`类的参数列表表示的是使用反射进行setProperties的时候应该忽略的参数，因为`executor`、`sslImplementationName`、`protocol`三个属性在`ConnectorCreateRule`类里面设置好了，所以在`SetAllPropertiesRule`类中会忽略这三个属性。</br>
在`ConnectorCreateRule#begin()`方法中，调用了有参构造函数`Connector(String protocol)`，根据传入的`protocol`，构造对应的`ProtocolHandler`对象。 </br>
Tomcat默认支持两种协议：`HTTP/1.1`和`AJP/1.3`。在每种协议下，分别有两种接收请求的处理方式：APR和NIO。如果对APR进行了配置，会优先使用APR。因为APR效率更高。参考：http://blog.csdn.net/xyang81/article/details/51502766 、https://my.oschina.net/czg/blog/142616</br>
## http
### NIO
`ProtocolHandler`类型是`org.apache.coyote.http11.Http11NioProtocol`、`org.apache.coyote.http11.Http11Nio2Protocol`，区别是一个使用NIO，第二个使用NIO2。使用`NioEndpoint`、`Nio2Endpoint`。NIO是Non-Blocking IO，支持阻塞和非阻塞IO，从java1.4引入，是IO Multiplexing技术的实现，在原来的流式IO的基础上增加了非阻塞特性。而NIO2是Asynchronous IO，异步IO，从java1.7开始引入，在NIO的基础上增加了四个异步的Channel。阻塞、非阻塞与同步、异步是两组不同的概念：
> 简单而言, 凡是需要由应用程序将数据读写到应用程序内存中的IO, 都是同步IO, 比如上面的流IO与NIO. 相对的, 凡是由OS来完成读写的, 就是异步IO. 这个说法有些迷惑. 举例而言, 在NIO中, 当应用程序检测到某个Channel有可读数据时, 必须显示发起一个read请求. 而在异步IO中, 应用程序仅仅需要告诉OS, 我需要什么数据, 并提供给OS一个Buffer和一个回调. OS会自己检测Channel的可读性, 但其发起其可读, 会自动将数据复制到Buffer中, 并通知应用程序任务完成. 异步IO的典型实现是NodeJS及Boost.ASIO. 显然, 由于将任务进一步下发到了OS, 应用程序的可伸缩性及性能会大大增强. 并且, 比起非阻塞的NIO, 异步IO编程更加容易一些, 性能也基本上总是优于它的。

关于NIO2的更多内容可以参考https://www.ibm.com/developerworks/java/library/j-nio2-1/index.html 、http://www.jianshu.com/p/07d3d421a877

### APR
`ProtocolHandler`类型是`org.apache.coyote.http11.Http11AprProtocol`。使用`AprEndpoint`
## ajp
### NIO
`ProtocolHandler`类型是`org.apache.coyote.ajp.AjpNioProtocol`、`org.apache.coyote.ajp.AjpNio2Protocol`，区别是一个使用NIO，第二个使用NIO2。使用`NioEndpoint`、`Nio2Endpoint`
### APR
`ProtocolHandler`类型是`org.apache.coyote.ajp.AjpAprProtocol`。使用`AprEndpoint`

## `init()`
在`Connector`的构造函数中已经创建了`ProtocolHandler`对象，接下来在`Connector`的`initInternal()`方法中，主要是为`ProtocolHandler`创建并设置一个适配器`CoyoteAdapter`，然后调用了`ProtocolHandler#init()`方法。
`Connector#init()`---></br>
`AbstractProtocol#init()`---></br>
`AbstractEndpoint#init()`---></br>
`bind()`</br>
`bind()`是一个抽象方法。
