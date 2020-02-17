# 看懂Dubbo（一）起步.md

## 配置 Zookeeper
1. 下载 Zookeeper
2. 到 conf 目录复制一份 zoo.cfg
3. 到 bin 目录启动 ./zkServer.sh start
4. 验证启动成功: netstat -nat | grep 2181

## 启动 Provider
1. 修改配置 
dubbo-demo/dubbo-demo-provider/src/main/resources/META-INF/spring/dubbo-demo-provider.xml
<dubbo:registry address="zookeeper://127.0.0.1:2181"/>
2. 启动 
com.alibaba.dubbo.demo.provider.Provider.main

## 启动 Consumer
1. 修改配置
dubbo-demo/dubbo-demo-consumer/src/main/resources/META-INF/spring/dubbo-demo-consumer.xml
<dubbo:registry address="zookeeper://127.0.0.1:2181"/>
2. 启动
com.alibaba.dubbo.demo.consumer.Consumer.main