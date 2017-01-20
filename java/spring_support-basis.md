spring_support_basis
========

```
        ApplicationContext content = new ClassPathXmlApplicationContext("spring.xml");
```

```
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// 准备刷新上下文环境
			prepareRefresh();

			// 初始化BeanFactory，并进行xml文件读取
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// 对beanFacotry进行各种功能填充
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				// 提供给子类覆盖方法做额外处理
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				// 激活各种BeanFactory处理器
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				// 注册拦截Bean创建的Bean处理器(这里只是注册,真正调用在getBean的时候)
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				// 初始化应用消息广播器,即不同语言的消息体,国际化处理
				initMessageSource();

				// Initialize event multicaster for this context.
				// 初始化广播器,并放到"applicationEventMulticaster" beank中										initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				// 提供给子类初始化其他的Bean
				onRefresh();

				// Check for listener beans and register them.
				//  在所有注册的bean中查找Listener bean,注册到消息广播中
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				// 初始化剩下的单实例(InitializingBean,DisposableBean, Aware对象等)
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				// 完成刷新过程,通知生命周期处理器
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

ClassPathXmlApplicationContext
AbstractXmlApplicationContext
AbstractRefreshableConfigApplicationContext
AbstractRefreshableApplicationContext
AbstractApplicationContext
```
@Override
public Object getBean(String name) throws BeansException {
	assertBeanFactoryActive();
	return getBeanFactory().getBean(name);
}
```
可以看到，通过beanFactory获取bean。


ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

```
public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
		throws BeansException {

	super(parent);
	setConfigLocations(configLocations);
	if (refresh) {
		refresh();
	}
}
```

AbstractBeanFactory.beanPostProcessors 属性


