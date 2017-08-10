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
`ProtocolHandler`类型是`org.apache.coyote.http11.Http11NioProtocol`、`org.apache.coyote.http11.Http11Nio2Protocol`，区别是一个使用NIO，第二个使用NIO2。使用`NioEndpoint`、`Nio2Endpoint`。NIO是Non-Blocking IO，非阻塞IO，从java1.4引入，是IO Multiplexing技术的实现。而NIO2是Asynchronous-IO，异步IO，从java1.7开始引入。关于NIO2的更多内容可以参考https://www.ibm.com/developerworks/java/library/j-nio2-1/index.html 、http://www.jianshu.com/p/07d3d421a877
### APR
`ProtocolHandler`类型是`org.apache.coyote.http11.Http11AprProtocol`。使用`AprEndpoint`
## ajp
### NIO
`ProtocolHandler`类型是`org.apache.coyote.ajp.AjpNioProtocol`、`org.apache.coyote.ajp.AjpNio2Protocol`，区别是一个使用NIO，第二个使用NIO2。使用`NioEndpoint`、`Nio2Endpoint`
### APR
`ProtocolHandler`类型是`org.apache.coyote.ajp.AjpAprProtocol`。使用`AprEndpoint`


