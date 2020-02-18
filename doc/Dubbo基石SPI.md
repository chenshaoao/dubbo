# Dubbo基石SPI

如果要分享，使用 dubbo-demo 项目
源码心得: 以接口为主线，属性是根基

SPI 本质分两步: 
1. new Object
2. set 属性

结论:
@SPI 表示接口需要 SPI 功能，根据 @SPI 注释的接口，找配置的实现类。
@SPI 不指定名称，表示走默认的自适应实现类 （实现类中注解 @Adaptive，实现类方法中注解 @Adaptive）
@SPI 指定名称，设置的是默认的实现类
@Adaptive 注解在子类中，表示该类是接口的默认实现类 （ExtensionFactory 和 Compiler 都是有具体子类实现逻辑的。）
@Adaptive 注解在方法中，根据方法中的 URL 参数，自适应找实现类

核心类 com.alibaba.dubbo.common.extension.ExtensionLoader 属性说明:
// 方法最后一段，执行后，停住，看 属性 值。

// 接口 class 类型
type
// 非默认接口实现扩展点 Map 集合 Class:Name
cachedNames
// 非默认接口实现扩展点 Map 集合 Name:Class
cachedClasses
// 接口上 @SPI 注解指定的 扩展点 名称
cachedDefaultName
// 接口默认的 扩展点 实现，在实现类中指定了 @Adaptive 注解 
cachedAdaptiveClass

cachedActivates

cachedInstances
cachedAdaptiveInstance

// 创建自适应扩展点时候的异常
createAdaptiveInstanceError
// 存所有异常
exceptions

核心类 com.alibaba.dubbo.common.extension.ExtensionLoader 方法说明:
// 获取扩展点加载器
public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type) {}
// 获取固定扩展点
public T getDefaultExtension() {}
// 获取自适应扩展点，方法调用的时候使用
// 如果子类中配置了 @Adaptive 就用子类，如果没有，就动态代码生成。
// ExtensionFactory 和 Compiler 都是有具体子类实现逻辑的。其他的都是 code 自生成，复制出来创建java文件看实现逻辑。
public T getAdaptiveExtension() {}

@Adaptive 隐式指定 key
key 就是 类名 全小写后，用点隔开: map.put("simple.ext", "impl2");
@Adaptive 显示指定 key 
@Adaptive({"key1", "key2"})

单元测试看加载机制是比较合适的，否则各种跳，直接挂了。
Debug 查看: 
com.alibaba.dubbo.common.extensionloader.ExtensionLoaderTest.test_getDefaultExtension （有错误的例子）
com.alibaba.dubbo.common.extensionloader.ExtensionLoader_Adaptive_Test.test_getAdaptiveExtension_defaultAdaptiveKey

TODO: 
active
Wrapper 