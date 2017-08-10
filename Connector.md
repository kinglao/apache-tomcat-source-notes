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
`Connector`对象的创建是在`ConnectorCreateRule`类中完成的。而`SetAllPropertiesRule`类的参数列表表示的是使用反射进行setProperties的时候应该忽略的参数，
因为"executor"、"sslImplementationName"、"protocol"三个属性在`ConnectorCreateRule`类的时候就设置好了，所以在`SetAllPropertiesRule`类中会忽略这三个属性
