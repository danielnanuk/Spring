# Spring Core

## 从容器启动说起
配置文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">
    <bean class="ren.eggpain.SimpleBean"/>
</beans>
```

启动代码:

```java
public class Application {
  public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
    SimpleBean simpleBean = context.getBean(SimpleBean.class);
    simpleBean.hello();
  }
}
```

SimpleBean:

```java
public class SimpleBean {
  public void hello() {
    System.out.println("Hello from SimpleBean");
  }
}
```
### 构造器
观察ClassPathXmlApplicationContext类，发现最终调用的构造器都是:

```java
public ClassPathXmlApplicationContext(String[] configLocations, 
									  boolean refresh,
									  ApplicationContext parent) throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) { // 始终为true，只有通过ContextSingletonBeanFactoryLocator来创建时会是false
			refresh();
		}
	}
```

父类构造器的调用直到AbstractApplicationContext为止:

```java
public AbstractApplicationContext(ApplicationContext parent) {
		this();
		setParent(parent);
}

public AbstractApplicationContext() {
		this.resourcePatternResolver = getResourcePatternResolver();
}

protected ResourcePatternResolver getResourcePatternResolver() {
		return new PathMatchingResourcePatternResolver(this);
}
```

PathMatchingResourcePatternResolver支持Ant风格路径解析

### 设置配置文件路径

```java
public void setConfigLocations(String... locations) {
		if (locations != null) {
			Assert.noNullElements(locations, "Config locations must not be null");
			this.configLocations = new String[locations.length];
			for (int i = 0; i < locations.length; i++) {
				this.configLocations[i] = resolvePath(locations[i]).trim();
			}
		}
		else {
			this.configLocations = null;
		}
	}
```

resolvePath:

```java
protected String resolvePath(String path) {
		return getEnvironment().resolveRequiredPlaceholders(path);
}
```
getEnvironment判断当前environment是否存在，不存在就创建一个:

```java
public ConfigurableEnvironment getEnvironment() {
		if (this.environment == null) {
			this.environment = createEnvironment();
		}
		return this.environment;
}
protected ConfigurableEnvironment createEnvironment() {
		return new StandardEnvironment();
}

```
### 加载配置，启动容器
调用refresh方法:

```java
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




