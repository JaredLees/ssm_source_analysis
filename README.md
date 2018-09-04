# ssm_source_analysis
> 此项目记录了对SpringMVC，Spring，Mybatis的理解和源码分析

## spring扩展点
```java

  //Spring中AbstractApplicationContext抽象类的refresh()方法是用来刷新Spring的应用上下文的。
  //下面Spring的应用上下文我都叫作context
  @Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
    
			// Prepare this context for refreshing.
      // ①、这个方法设置context的启动日期。
　　  // ②、设置context当前的状态，是活动状态还是关闭状态。
　　  // ③、初始化context environment（上下文环境）中的占位符属性来源。
     // ④、验证所有必需的属性。
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
      // 让这个类（AbstractApplicationContext）的子类刷新内部bean工厂。实际上就是重新创建一个bean工厂。
      //  loadBeanDefinitions(beanFactory); 加载BeanDefinition
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
      // 已经把工厂建好了，但是还不能投入使用，因为工厂里什么都没有，还需要配置一些东西
      // 配置这个工厂的标准环境，比如context的类加载器和后处理器(BeanPostProcessor扩展点)加载到BeanFactory中
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
        // ①、添加一个ServletContextAwareProcessor到bean工厂中。
　　    // ②、在bean工厂自动装配的时候忽略一些接口。如：ServletContextAware、ServletConfigAware
       // ③、注册WEB应用特定的域（scope）到bean工厂中，以便WebApplicationContext可以使用它们。比如"request", 
       // "session", "globalSession", "application"，
       // ④、注册WEB应用特定的Environment bean到bean工厂中，以便WebApplicationContext可以使用它们。
       // 如："contextParameters", "contextAttributes"
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
        // 调用所有的bean工厂处理器（BeanFactoryPostProcessor）对bean工厂进行一些处理
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
        // 注册用来拦截bean创建的BeanPostProcessor bean.这个方法需要在所有的application bean初始化之前调用。
        // 把这个注册的任务委托给了PostProcessorRegistrationDelegate来完成
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
        // 初始化MessageSource接口的一个实现类。这个接口提供了消息处理功能。主要用于国际化/i18n。
				initMessageSource();

				// Initialize event multicaster for this context.
        // 为这个context初始化一个事件广播器（ApplicationEventMulticaster）
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
        // 在AbstractApplicationContext的子类中初始化其他特殊的bean
				onRefresh();

				// Check for listener beans and register them.
        // 注册应用的监听器。就是注册实现了ApplicationListener接口的监听器bean，这些监听器是注册到ApplicationEventMulticaster中的。
        // 这不会影响到其它监听器bean。在注册完以后，还会将其前期的事件发布给相匹配的监听器。
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
        // 完成bean工厂的初始化工作。这一步非常复杂，也非常重要，涉及到了bean的创建。
        // 第二步中只是完成了BeanDefinition的定义、解析、处理、注册。但是还没有初始化bean实例。这一步将初始化所有非懒加载的单例bean
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
        // 完成context的刷新。主要是调用LifecycleProcessor的onRefresh()方法，并且发布事件（ContextRefreshedEvent）
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
