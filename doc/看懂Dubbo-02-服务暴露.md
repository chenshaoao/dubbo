# 看懂Dubbo（二）服务暴露

## 找到起点 
书上都是从上往下讲解 （装逼满满）
我们需要从下往上排解 （心中疑虑）
想办法先找到 ServerBootstrap 启动的点，然后 debug 看如何调用的
看日志输出，找到分别在哪打印的

doOpen:66, NettyServer (com.alibaba.dubbo.remoting.transport.netty)
<init>:63, AbstractServer (com.alibaba.dubbo.remoting.transport)
<init>:61, NettyServer (com.alibaba.dubbo.remoting.transport.netty)
bind:32, NettyTransporter (com.alibaba.dubbo.remoting.transport.netty)
bind:-1, Transporter$Adaptive (com.alibaba.dubbo.remoting)
bind:56, Transporters (com.alibaba.dubbo.remoting)
bind:44, HeaderExchanger (com.alibaba.dubbo.remoting.exchange.support.header)
bind:70, Exchangers (com.alibaba.dubbo.remoting.exchange)
createServer:285, DubboProtocol (com.alibaba.dubbo.rpc.protocol.dubbo)
openServer:264, DubboProtocol (com.alibaba.dubbo.rpc.protocol.dubbo)
export:251, DubboProtocol (com.alibaba.dubbo.rpc.protocol.dubbo)
export:57, ProtocolListenerWrapper (com.alibaba.dubbo.rpc.protocol)
export:100, ProtocolFilterWrapper (com.alibaba.dubbo.rpc.protocol)
export:-1, Protocol$Adaptive (com.alibaba.dubbo.rpc)
doLocalExport:169, RegistryProtocol (com.alibaba.dubbo.registry.integration)
export:132, RegistryProtocol (com.alibaba.dubbo.registry.integration)
export:55, ProtocolListenerWrapper (com.alibaba.dubbo.rpc.protocol)
export:98, ProtocolFilterWrapper (com.alibaba.dubbo.rpc.protocol)
export:-1, Protocol$Adaptive (com.alibaba.dubbo.rpc)
doExportUrlsFor1Protocol:513, ServiceConfig (com.alibaba.dubbo.config)
doExportUrls:358, ServiceConfig (com.alibaba.dubbo.config)
doExport:317, ServiceConfig (com.alibaba.dubbo.config)
export:216, ServiceConfig (com.alibaba.dubbo.config)
export:291, ServiceBean (com.alibaba.dubbo.config.spring)
onApplicationEvent:131, ServiceBean (com.alibaba.dubbo.config.spring)
onApplicationEvent:53, ServiceBean (com.alibaba.dubbo.config.spring)
doInvokeListener:172, SimpleApplicationEventMulticaster (org.springframework.context.event)
invokeListener:165, SimpleApplicationEventMulticaster (org.springframework.context.event)
multicastEvent:139, SimpleApplicationEventMulticaster (org.springframework.context.event)
publishEvent:393, AbstractApplicationContext (org.springframework.context.support)
publishEvent:347, AbstractApplicationContext (org.springframework.context.support)
finishRefresh:883, AbstractApplicationContext (org.springframework.context.support)
refresh:546, AbstractApplicationContext (org.springframework.context.support)
<init>:139, ClassPathXmlApplicationContext (org.springframework.context.support)
<init>:93, ClassPathXmlApplicationContext (org.springframework.context.support)
main:27, Provider (com.alibaba.dubbo.demo.provider)

流程讲解:
找实现类的步骤是: adaptive -> filter -> listener -> 实现类
com.alibaba.dubbo.config.spring.ServiceBean.export
com.alibaba.dubbo.config.ServiceConfig.export
com.alibaba.dubbo.config.ServiceConfig.doExportUrls
com.alibaba.dubbo.config.ServiceConfig.doExportUrlsFor1Protocol
// 本地暴露
exportLocal(url);
// 远端暴露
Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString()));

com.alibaba.dubbo.registry.integration.RegistryProtocol.export
com.alibaba.dubbo.registry.integration.RegistryProtocol.doLocalExport
final Invoker<?> invokerDelegete = new InvokerDelegete<T>(originInvoker, getProviderUrl(originInvoker));

com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol.export
com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol.openServer

// RegistryProtocol.doLocalExport 后注册 zk 数据
com.alibaba.dubbo.registry.integration.RegistryProtocol.export
com.alibaba.dubbo.registry.integration.RegistryProtocol.register
com.alibaba.dubbo.registry.support.FailbackRegistry.register
com.alibaba.dubbo.registry.zookeeper.ZookeeperRegistry.doRegister
com.alibaba.dubbo.remoting.zookeeper.support.AbstractZookeeperClient.create
com.alibaba.dubbo.remoting.zookeeper.curator.CuratorZookeeperClient.createEphemeral

// 订阅 dubbo 转 provider
com.alibaba.dubbo.registry.integration.RegistryProtocol.export
...

看 Zookeeper 创建节点调用栈，dubbo 是临时节点
createEphemeral:86, CuratorZookeeperClient (com.alibaba.dubbo.remoting.zookeeper.curator)
create:65, AbstractZookeeperClient (com.alibaba.dubbo.remoting.zookeeper.support)
doRegister:114, ZookeeperRegistry (com.alibaba.dubbo.registry.zookeeper)
register:137, FailbackRegistry (com.alibaba.dubbo.registry.support)
register:126, RegistryProtocol (com.alibaba.dubbo.registry.integration)
export:146, RegistryProtocol (com.alibaba.dubbo.registry.integration)
export:55, ProtocolListenerWrapper (com.alibaba.dubbo.rpc.protocol)
export:98, ProtocolFilterWrapper (com.alibaba.dubbo.rpc.protocol)
export:-1, Protocol$Adaptive (com.alibaba.dubbo.rpc)
doExportUrlsFor1Protocol:513, ServiceConfig (com.alibaba.dubbo.config)
doExportUrls:358, ServiceConfig (com.alibaba.dubbo.config)
doExport:317, ServiceConfig (com.alibaba.dubbo.config)
export:216, ServiceConfig (com.alibaba.dubbo.config)
export:291, ServiceBean (com.alibaba.dubbo.config.spring)
onApplicationEvent:131, ServiceBean (com.alibaba.dubbo.config.spring)
onApplicationEvent:53, ServiceBean (com.alibaba.dubbo.config.spring)
doInvokeListener:172, SimpleApplicationEventMulticaster (org.springframework.context.event)
invokeListener:165, SimpleApplicationEventMulticaster (org.springframework.context.event)
multicastEvent:139, SimpleApplicationEventMulticaster (org.springframework.context.event)
publishEvent:393, AbstractApplicationContext (org.springframework.context.support)
publishEvent:347, AbstractApplicationContext (org.springframework.context.support)
finishRefresh:883, AbstractApplicationContext (org.springframework.context.support)
refresh:546, AbstractApplicationContext (org.springframework.context.support)
<init>:139, ClassPathXmlApplicationContext (org.springframework.context.support)
<init>:93, ClassPathXmlApplicationContext (org.springframework.context.support)
main:27, Provider (com.alibaba.dubbo.demo.provider)