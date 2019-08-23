# spring原理

### 解析xml生成BeanDefinition

spring根据配置文件解析xml生成bean的定义实例，常见标签有

- beans
- bean
- property

### Bean创建

xml解析完成之后，spring会进行bean的加载、实例化。

#### getBean方法

获取bean从BeanFactory的getBean方法进入

```
MyTestBean bean=(MyTestBean) bf . getBean( "myTestBean" )
```

#### doGetBean方法

getBean方法调用doGetBean方法

```
@Override
	public Object getBean(String name) throws BeansException {
		return doGetBean(name, null, null, false);
	}
```

```
protected <T> T doGetBean(
      final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly)
      throws BeansException {

   final String beanName = transformedBeanName(name);
   Object bean;
   //直接从缓存获取或者ObjectFactory获取
   Object sharedInstance = getSingleton(beanName);
```

如果获取不到bean 那么就会进行bean的创建，创建过程中会进行循环依赖检查，创建bean会进入

```
// Create bean instance.
创建bean的实例
if (mbd.isSingleton()) {
   sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
      @Override
      public Object getObject() throws BeansException {
         try {
         调用createBean方法创建
            return createBean(beanName, mbd, args);
         }
```

#### createBean方法

createBean方法创建进行bean创建前处理

**resolveBeforeInstantiation**方法先给**BeanPostProcessors**机会进行实例化的前置处理，当经过前置处理后返回的结果如果不为空，那么会直接略过后续的 bean 的创建而直接返回结果。 如果没有返回则进入doCreateBean方法进行bean的创建

```
protected Object createBean(String beanName, RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
		if (logger.isDebugEnabled()) {
			logger.debug("Creating instance of bean '" + beanName + "'");
		}
		RootBeanDefinition mbdToUse = mbd;

	//锁定 class ，根据设置的 class 属性或者根据 className 来解析 Class 
		Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
		if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
			mbdToUse = new RootBeanDefinition(mbd);
			mbdToUse.setBeanClass(resolvedClass);
		}

		// Prepare method overrides.
		try {
			mbdToUse.prepareMethodOverrides();
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
					beanName, "Validation of method overrides failed", ex);
		}
		try {
			// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
			 //给BeanPostProcessors 一个机会来返回代理来替代真正的实例
			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
			if (bean != null) {
				return bean;
			}
		}
		catch (Throwable ex) {
			throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
					"BeanPostProcessor before instantiation of bean failed", ex);
		}
		进入doCreateBean方法进行bean的创建
		Object beanInstance = doCreateBean(beanName, mbdToUse, args);
		if (logger.isDebugEnabled()) {
			logger.debug("Finished creating instance of bean '" + beanName + "'");
		}
		return beanInstance;
	}
```

##### BeanPostProcessors应用

**resolveBeforeInstantiation方法中BeanPostProcessors处理器的使用**

先给BeanPostProcessors机会进行实例化的前置处理，当经过前置处理后返回的结果如果不为空，那么会直接略过后续的 bean 的创建而直接返 回结果。 这一特性虽然很容易被忽略，但是却起着至关重要的作用，我们熟知的 AOP 功能就 是基于这里的判断的。

```
	protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
		Object bean = null;
		if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
			// Make sure bean class is actually resolved at this point.
			if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
				Class<?> targetType = determineTargetType(beanName, mbd);
				if (targetType != null) {
				
				//前置处理器的使用
					bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
					if (bean != null) {
					//如果前置处理器返回bean，则直接完成bean的创建，执行后置处理器
						bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
					}
				}
			}
			mbd.beforeInstantiationResolved = (bean != null);
		}
		return bean;
	}
```

如果前置处理器没有返回bean则进入doCreateBean方法进行bean的创建

#### doCreateBean方法

doCreateBean方法中几个重要的方法，代表着spring的生命周期。

```
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args)
			throws BeanCreationException {

		// Instantiate the bean.
		BeanWrapper instanceWrapper = null;
		if (mbd.isSingleton()) {
			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
		}
		if (instanceWrapper == null) {
			instanceWrapper = createBeanInstance(beanName, mbd, args);
		}
		final Object bean = (instanceWrapper != null ? instanceWrapper.getWrappedInstance() : null);
		Class<?> beanType = (instanceWrapper != null ? instanceWrapper.getWrappedClass() : null);
		mbd.resolvedTargetType = beanType;

		// Allow post-processors to modify the merged bean definition.
		synchronized (mbd.postProcessingLock) {
			if (!mbd.postProcessed) {
				try {
					applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
				}
				catch (Throwable ex) {
					throw new BeanCreationException(mbd.getResourceDescription(), beanName,
							"Post-processing of merged bean definition failed", ex);
				}
				mbd.postProcessed = true;
			}
		}

		// Eagerly cache singletons to be able to resolve circular references
		// even when triggered by lifecycle interfaces like BeanFactoryAware.
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isDebugEnabled()) {
				logger.debug("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
			addSingletonFactory(beanName, new ObjectFactory<Object>() {
				@Override
				public Object getObject() throws BeansException {
					return getEarlyBeanReference(beanName, mbd, bean);
				}
			});
		}

		// Initialize the bean instance.
		Object exposedObject = bean;
		try {
			populateBean(beanName, mbd, instanceWrapper);
			if (exposedObject != null) {
				exposedObject = initializeBean(beanName, exposedObject, mbd);
			}
		}
		catch (Throwable ex) {
			if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
				throw (BeanCreationException) ex;
			}
			else {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
			}
		}

		if (earlySingletonExposure) {
			Object earlySingletonReference = getSingleton(beanName, false);
			if (earlySingletonReference != null) {
				if (exposedObject == bean) {
					exposedObject = earlySingletonReference;
				}
				else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
					String[] dependentBeans = getDependentBeans(beanName);
					Set<String> actualDependentBeans = new LinkedHashSet<String>(dependentBeans.length);
					for (String dependentBean : dependentBeans) {
						if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
							actualDependentBeans.add(dependentBean);
						}
					}
					if (!actualDependentBeans.isEmpty()) {
						throw new BeanCurrentlyInCreationException(beanName,
								"Bean with name '" + beanName + "' has been injected into other beans [" +
								StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
								"] in its raw version as part of a circular reference, but has eventually been " +
								"wrapped. This means that said other beans do not use the final version of the " +
								"bean. This is often the result of over-eager type matching - consider using " +
								"'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
					}
				}
			}
		}

		// Register bean as disposable.
		try {
			registerDisposableBeanIfNecessary(beanName, bean, mbd);
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
		}

		return exposedObject;
	}

```



##### createBeanInstance

创建bean的实例

```
根据指定 bean 使用对应的策略创建新的实例， 如：工厂方法、构造函数自动注入 、简单初始化 
instanceWrapper = createBeanInstance(beanName, mbd, args);
```

##### populateBean属性填充方法

获取的属性是以PropertyValues 形式存在的，还并没有应用到已经实例化的 bean 中 ， 这一工作是在 applyProperty Values 中。 如果后处理器无任何处理，则进行属性填充，反之不进行属性填充。

```
	//属性填充
	populateBean(beanName, mbd, instanceWrapper);
```

populateBean方法详解

```
protected void populateBean(String beanName, RootBeanDefinition mbd, BeanWrapper bw) {
		PropertyValues pvs = mbd.getPropertyValues();
        .......省略

		// Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
		// state of the bean before properties are set. This can be used, for example,
		// to support styles of field injection.
		//给后处理器最后一次机会决定属性填充是否进行
		boolean continueWithPropertyPopulation = true;
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof InstantiationAwareBeanPostProcessor) {
					InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
					if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
						continueWithPropertyPopulation = false;
						break;
					}
				}
			}
		}
		......省略
		//属性填充
		applyPropertyValues(beanName, mbd, bw, pvs);
```

##### initializeBean方法

- **激活Aware方法**  例如beanFactoryAware接口

- **BeanPostProcessors处理器的应用**  ，在调用客户 自定义初始化方法前以及调用自定义初始化方法后分别会调用 BeanPostProcessor 的 postProcessBeforelnitialization 和 postProcessAfterlnitialization 方法，使用 户可以根据自己的业务需求进行响应的处理。 

- **invokeInitMethods初始化方法应用**，客户定制的初始化方法除了我们熟知的使用配置 init-method 外，还有使 自定义的 bean 实 现 **InitializingBean** 接口，并在 afterPropertiesSet 中实现自己的初始化业务逻辑。 init-method 与 afterPropertiesSet 都是在初始化 b巳an 时执行，执行顺序是 afterPropertiesSet 先执行，而 init-method 后执行。 在 invokelnitMethods 方法中就实现了这两个步骤的初始化方法调用。 

```
//初始化
exposedObject = initializeBean(beanName, exposedObject, mbd);
```

```
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
      .....省略
      //Aware方法
      invokeAwareMethods(beanName, bean);
  //前处理器
      wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
 //自定义初始化方法
      invokeInitMethods(beanName, wrappedBean, mbd);
  //后处理器
      wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);


}
```

##### 注册DisposableBean方法

Spring 中不但提供了对于初始化方法的扩展人口 ， 同样也提供了销毁方法的扩展入口，对 于销毁方法的扩展，除了我们熟知的配置属性 destroy-method 方法外，用户还可以注册后处理 器 DestructionAwareBeanPostProcessor 来统一处理 bean 的销毁方法。

```
protected void registerDisposableBeanIfNecessary(String beanName, Object bean, RootBeanDefinition mbd) {
		AccessControlContext acc = (System.getSecurityManager() != null ? getAccessControlContext() : null);
		if (!mbd.isPrototype() && requiresDestruction(bean, mbd)) {
			if (mbd.isSingleton()) {
				// Register a DisposableBean implementation that performs all destruction
				// work for the given bean: DestructionAwareBeanPostProcessors,
				// DisposableBean interface, custom destroy method.
				registerDisposableBean(beanName,
						new DisposableBeanAdapter(bean, beanName, mbd, getBeanPostProcessors(), acc));
			}
			else {
				// A bean with a custom scope...
				Scope scope = this.scopes.get(mbd.getScope());
				if (scope == null) {
					throw new IllegalStateException("No Scope registered for scope name '" + mbd.getScope() + "'");
				}
				scope.registerDestructionCallback(beanName,
						new DisposableBeanAdapter(bean, beanName, mbd, getBeanPostProcessors(), acc));
			}
		}
	}

```

# spring中有关接口使用

新建测试用类BeanTest

```
@Service
public class BeanTest implements InitializingBean, BeanFactoryAware {
    private BeanFactory beanFactory;

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("初始化");
        //调用hello方法
        hello();
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    public void hello() {
        Object o = beanFactory.getBean("springFactoryBean");
        BusinessService businessService = (BusinessService) o;
        System.out.println(o);
        ((BusinessService) o).test();
        System.out.println("hello spring");
    }

    public List<User> test() {
        UserDao userDao = (UserDao) beanFactory.getBean("userDao");
        return userDao.getAllUser();
    }

}


```



#### InitializingBean接口 

InitializingBean接口只有一个afterPropertiesSet（）方法，表示在Bean初始化，完成属性注入之后调用，在用户配置的init-method方法之前被调用。

**如上我们的测试类BeanTest类在初始化会调用afterPropertiesSet方法，控制台会打印 初始化和hello方法里的hello Spring**

```
public interface InitializingBean {
    void afterPropertiesSet() throws java.lang.Exception;
}
```

#### BeanFactoryAware接口

实现 BeanFactoryAware 接口的 bean 可以直接访问 Spring 容器，被容器创建以后，它会拥有一个指向 Spring 容器的引用。BeanFactoryAware 接口只有一个方法void setBeanFactory(BeanFactorybeanFactory)。

**如上我们的测试类BeanTest类在初始化会调用setBeanFactory方法，给成员变量beanFactory赋值，然后就可以在此类中获取spring容器中的bean并调用。**

```

public interface BeanFactoryAware extends org.springframework.beans.factory.Aware {
    void setBeanFactory(org.springframework.beans.factory.BeanFactory beanFactory) throws org.springframework.beans.BeansException;
}
```

#### FactoryBean接口

**BeanFacotry是spring中比较原始的Factory。如XMLBeanFactory就是一种典型的BeanFactory。原始的BeanFactory无法支持spring的许多插件，如AOP功能、Web应用等。** 
**ApplicationContext接口,它由BeanFactory接口派生而来，ApplicationContext包含BeanFactory的所有功能，通常建议比BeanFactory优先**

一旦某个类实现BeanFactory接口的Bean，在SpringIoc中Bean是 **getBean（）方法**返回的bean

isSingleton方法返回true是让bean为单例

新建SpringFactoryBean类

```
@Service
public class SpringFactoryBean implements FactoryBean {

    @Override
    public Object getObject() throws Exception {
        return new BusinessService();
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}


```

新建BusinessService类

```
public class BusinessService {
    public void test(){
        System.out.println("SpringIoc中SpringFactoryBean真正的bean");
    }
}
```

修改BeanTest类的hello方法

```
    public void hello() {
        Object o = beanFactory.getBean("springFactoryBean");
        BusinessService businessService = (BusinessService) o;
        System.out.println(o);
        ((BusinessService) o).test();
        System.out.println("hello spring");
    }
```

IOC初始化时控制台打印如下内容

```
初始化
com.book.MQ.BusinessService@63f855b
SpringIoc中SpringFactoryBean真正的bean
hello spring
```

