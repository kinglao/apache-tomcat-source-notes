在前面的文档中介绍过了，在`Bootstrap`、`Catalina`中调用`load()`方法中，使用`Digester`类将`server.xml`中的元素解析为对应的类，最后就是调用`StandardServer`的`init()`方法。对应于
