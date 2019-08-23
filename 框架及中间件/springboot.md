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
