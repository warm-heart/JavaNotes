### starter原理

springboot启动时会扫描依赖的jar包 resource/META-INF下的spring.factories文件，寻找相应的类,根据配置类扫描类并注册到IOC容器中。

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.study.service.HelloServiceAutoConfiguration
```

HelloServiceAutoConfiguration类

```
@Configuration
@ComponentScan({"com.study.service"})
//仅当当前目录下有com.study.service.helloService时才创建
@ConditionalOnClass(name = {"com.study.service.helloService"})
public class HelloServiceAutoConfiguration {
}
```

### 条件注解

通常与starter配合使用，仅当满足一些特定条件，才可以完成bean的创建。

- @ConditionalOnBean
- @ConditionalOnMissingBean
- @ConditionalOnClass
- @ConditionalOnMissingClass
- @ConditionalOnProperty
- @ConditionalOnExpression

@ConditionalOnClass注解 ：仅当当前目录下有指定的类才会创建Bean，其他注解原理类似

```
@Configuration
@ConditionalOnClass(name = {"com.book.config.condition"})
public class IOCConfid {
    @Bean
    public Role role() {
        Role role = new Role();
        role.setRoleName("cdkfdsd");
        return role;
    }
}
```

如果当前class路径没有com.book.config.condition类则会报错

```
The following candidates were found but could not be injected:
	- Bean method 'role' in 'IOCConfid' not loaded because @ConditionalOnClass did not find required class 'com.book.config.condition'
```

# 源码分析

```
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // Prepare the bean factory for use in this context.
      prepareBeanFactory(beanFactory);

      try {
         // Allows post-processing of the bean factory in context subclasses.
         postProcessBeanFactory(beanFactory);

         // Invoke factory processors registered as beans in the context.
         //执行BeanFactoryPostProcessors
         invokeBeanFactoryPostProcessors(beanFactory);

         // Register bean processors that intercept bean creation.
         registerBeanPostProcessors(beanFactory);

         // Initialize message source for this context.
         initMessageSource();

         // Initialize event multicaster for this context.
         initApplicationEventMulticaster();

         // Initialize other special beans in specific context subclasses.
         onRefresh();

         // Check for listener beans and register them.
         registerListeners();

         // Instantiate all remaining (non-lazy-init) singletons.
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         finishRefresh();
      }

      catch (BeansException ex) {
         if (logger.isWarnEnabled()) {
            logger.warn("Exception encountered during context initialization - " +
                  "cancelling refresh attempt: " + ex);
         }

         // Destroy already created singletons to avoid dangling resources.
         destroyBeans();

         // Reset 'active' flag.
         cancelRefresh(ex);

         // Propagate exception to caller.
         throw ex;
      }

      finally {
         // Reset common introspection caches in Spring's core, since we
         // might not ever need metadata for singleton beans anymore...
         resetCommonCaches();
      }
   }
}
```

### invokeBeanFactoryPostProcessors

执行BeanFactoryPostProcessor的postProcessBeanFactory方法 ，

1.先执行是事先加入的BeanFactoryPostProcessor，其中就有ConfigurationClassPostProcessor负责解析项目的启动类，进行bean的扫描，当扫描到自定义的BeanFactoryPostProcessor时然后在执行postProcessBeanFactory的postProcessBeanFactory方法。（BeanFactoryPostProcessor是分批执行的，不是一次执行的，因为由事先手动加入spring容器的先执行，在执行扫描出的自定的BeanFactoryPostProcessor）；

### onRefresh

applicationContext提供给子类扩展的。

实例化所有的bean。并启动servlet容器。

## bean的扫描顺序

1. 先扫描用户工程下的所有配置类，递归解析
2. 扫描import下的配置类，递归解析
3. invokeBeanFactoryPostProcessor中扫描所有的beanPostProcessor处理器
4. onfersh方法启动tomcat容器并实例化所有bean
5. getBean时调用处理器的方法对Bean进行特殊处理（AOP）

