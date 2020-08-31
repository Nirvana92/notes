**dubbo版本 2.7.8**

dubbo 整合spring 之后, 有一个`DubboLifecycleComponentRegistrar`.实现了`ImportBeanDefinitionRegistrar`.在该类中添加注册的bean.

包括: 

1. `DubboLifecycleComponentApplicationListener`
2. `DubboBootstrapApplicationListener`

```java
public class DubboLifecycleComponentRegistrar implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        registerBeans(registry, DubboLifecycleComponentApplicationListener.class);
        registerBeans(registry, DubboBootstrapApplicationListener.class);
    }
}
```

其中dubbo的服务暴露会在`DubboBootstrapApplicationListener`中监听`ContextRefreshedEvent`事件来来进行处理。