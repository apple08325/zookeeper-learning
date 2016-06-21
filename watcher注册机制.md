# Zookeeper学习笔记


## watcher注册机制
### 客户端watcher注册
ZKWatchManager存有三种watcher，分别是dataWatches、existWatches、childWatches
以getData(path,wather,stat)为例，注册watcher的步骤如下：
1. 构造成DataWatchRegistration对象传入ClientCnxn.submitRequest()
2. 通过queuePacket()构造Packet，把packet放入outgoingQueue队列，等待发送线程发送消息
queuePacket
> SendThread里的run方法里面最终通过这个方法clientCnxnSocket.doTransport(to, pendingQueue, ClientCnxn.this)，向服务端发送请求。

3. SendThread.readResponse里通过最后的操作finishPacket，注册watcher
4. p.watchRegistration.register(err)完成注册
### 客户端回调
SendThread.readResponse方法里面通过判断replyHdr.getXid() == -1，然后包装成WatchedEvent，通过eventThread.queueEvent()放入waitingEvents队列里


### 服务端watcher注册
####  DataTree维护了两个watchManager
* dataWatches保存监听数据结点变更的watchers
* childWatches保存监听子结点信息变更的watchers

watchManager内部维护了两种存储结构
* watchTable，记录的是Path对应的watchers
* watch2Paths，记录的是watcher（`其实是ServerCnxn实现了watcher接口，方便通知监听的节点`）对应的所有paths
#### 服务端触发
在setData、createNode等的操作下会去遍历并触发watcher的process方法，并且移除watcher
