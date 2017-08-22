## `init()`
在`Connector`的构造函数中已经创建了`ProtocolHandler`对象，接下来在`Connector`的`initInternal()`方法中，主要是为`ProtocolHandler`创建并设置一个适配器`CoyoteAdapter`，然后调用了`ProtocolHandler#init()`方法。</br>
`Connector#init()`---></br>
`AbstractProtocol#init()`---></br>
`AbstractEndpoint#init()`---></br>
`bind()`</br>
`bind()`是一个抽象方法。</br>
`NioEndpoint#bind()`</br>
* initServerSocket().创建并设置`ServerSocketChannel`的属性，并在ip地址和端口号上进行bind。
* NioSelectorPool#open() 如果`NioSelectorPool#SHARED`为`true`（默认为true，可以通过属性`org.apache.tomcat.util.net.NioSelectorShared`设置），会调用`Selector.open()`，将结果保存到`NioSelectorPool#SHARED_SELECTOR`，同时会创建一个`NioBlockingSelector`对象，并调用`NioBlockingSelector#open()`方法。这种情况下，`Selector`只有一个`NioSelectorPool#SHARED_SELECTOR`，是共享的。`NioBlockingSelector#open()`方法中，创建了一个`NioBlockingSelector.BlockPoller`类实例。`NioSelectorPool`是一个`Selector`池，使用`ConcurrentLinkedQueue<Selector> selectors`保存所有的`Selector`，`NioSelectorPool#spare`保存`selectors`中`Selector`数量，也就是空闲的`Selector`数量。`NioSelectorPool#active`保存了当前系统中活跃的`Selector`数量。所以将`Selector`加入到`selectors`中时（put()方法），空闲的`Selector`（spare）增加，而活跃的`Selector`（active）减少。当从`selectors`中取出`Selector`时（get()方法），空闲的`Selector`（spare）减少，而活跃的`Selector`（active）增加。
