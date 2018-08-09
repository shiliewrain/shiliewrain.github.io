---
layout : post
title : "Springboot启动过程简略分析（一）"
category : Springboot
tags : springboot spring
---
* content
{:toc}

　　Springboot的启动过程比较复杂，慢慢学习，慢慢分析。




### Springboot的启动类

　　我们新建的每一个Springboot项目都有一个启动类，该类里面有一个@SpringBootApplication注解，用于启用Springboot的特性，还有一个main函数，用于启动Springboot项目。

```java
@SpringBootApplication
public class SpringbootTestApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootTestApplication.class, args);
    }
}
```

　　由这个SpringApplication.run开始启动Springboot项目，进一步阅读源码：

```java
public static ConfigurableApplicationContext run(Class<?> primarySource,
			String... args) {
		return run(new Class<?>[] { primarySource }, args);
	}

	public static ConfigurableApplicationContext run(Class<?>[] primarySources,
			String[] args) {
		return new SpringApplication(primarySources).run(args);
	}
```

　　由源码可以看出，真正执行的方法是新创建的SpringApplication的run方法。我们先来看看新创建的SpringApplication实例包含哪些内容。

#### SpringApplication实例

　　阅读SpringApplication的源码，其构造函数如下：

```java
public SpringApplication(Class<?>... primarySources) {
		this(null, primarySources);
	}

	@SuppressWarnings({ "unchecked", "rawtypes" })
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;//1
		Assert.notNull(primarySources, "PrimarySources must not be null");//2
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));//3
		this.webApplicationType = deduceWebApplicationType();//4
		setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class));//5
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));//6
		this.mainApplicationClass = deduceMainApplicationClass();//7
	}
```

　　分析上面源码可得：

1. resourceLoader为null；

2. primarySources为SpringbootTestApplication.class；

3. 代码3是为了将primarySources去重；

4. 代码4是推断该Springboot项目的类型，有NONE、SERVLET、REACTIVE；

5. 代码5设置该应用上下文初始化器；

6. 代码6设置该应用上下文监听器；

7. 代码7是推断该应用的main函数所在类。

　　其中最为复杂的代码是代码5和代码6，但这两句代码都调用了getSpringFactoriesInstances方法，理解其中之一即可。

#### getSpringFactoriesInstances

　　先来看看getSpringFactoriesInstances的源码实现：

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
		return getSpringFactoriesInstances(type, new Class<?>[] {});
	}

	private <T> Collection<T> getSpringFactoriesInstances(Class<T> type,
			Class<?>[] parameterTypes, Object... args) {
		ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
		// Use names and ensure unique to protect against duplicates
		Set<String> names = new LinkedHashSet<>(
				SpringFactoriesLoader.loadFactoryNames(type, classLoader));//1
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);//2
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
```

　　而该方法中，最为重要的是1和2两句。代码1是根据type加载相应SpringFactory，代码2是根据加载的SpringFactory创建相应的实例。

##### SpringFactory加载

　　SpringFactory的加载主要是根据type去META-INF/spring.factories文件中加载相应的类型。其过程是首先在SpringFactoriesLoader的Map型缓存中查找是否有key为classLoader的元素，如果有，则直接返回结果，然后根据factoryClassName找到对应的list；如果没有，则去读取META-INF/spring.factories，将该文件中的内容放在一个以classLoader为key的map中，再返回该map，通过factoryClassName取出对应的list。直接看源码来理解：

```java
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
		String factoryClassName = factoryClass.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
	}

	private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				UrlResource resource = new UrlResource(url);
				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					List<String> factoryClassNames = Arrays.asList(
							StringUtils.commaDelimitedListToStringArray((String) entry.getValue()));
					result.addAll((String) entry.getKey(), factoryClassNames);
				}
			}
			cache.put(classLoader, result);
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
```

　　代码中主要加载的是ApplicationContextInitializer和ApplicationListener这两种类型的类，在spring.factories中可以看到它们的相应配置，内容如下：

```
# Application Context Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\
org.springframework.boot.context.ContextIdApplicationContextInitializer,\
org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.ClearCachesApplicationListener,\
org.springframework.boot.builder.ParentContextCloserApplicationListener,\
org.springframework.boot.context.FileEncodingApplicationListener,\
org.springframework.boot.context.config.AnsiOutputApplicationListener,\
org.springframework.boot.context.config.ConfigFileApplicationListener,\
org.springframework.boot.context.config.DelegatingApplicationListener,\
org.springframework.boot.context.logging.ClasspathLoggingApplicationListener,\
org.springframework.boot.context.logging.LoggingApplicationListener,\
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener
```

##### SpringFactory实例化

　　在加载了SpringFactory之后，即从META-INF/spring.factories文件中获取到ApplicationListener和ApplicationContextInitializer的name列表，这些类即为需要被实例化的类。它们的实例化是通过反射来实现的，源码如下：

```java
private <T> List<T> createSpringFactoriesInstances(Class<T> type,
			Class<?>[] parameterTypes, ClassLoader classLoader, Object[] args,
			Set<String> names) {
		List<T> instances = new ArrayList<>(names.size());
		for (String name : names) {
			try {
				Class<?> instanceClass = ClassUtils.forName(name, classLoader);
				Assert.isAssignable(type, instanceClass);
				Constructor<?> constructor = instanceClass
						.getDeclaredConstructor(parameterTypes);
				T instance = (T) BeanUtils.instantiateClass(constructor, args);
				instances.add(instance);
			}
			catch (Throwable ex) {
				throw new IllegalArgumentException(
						"Cannot instantiate " + type + " : " + name, ex);
			}
		}
		return instances;
	}
```

　　通过一张图来看看整个SpringApplication的实例化过程。

[![springboot-init](https://raw.githubusercontent.com/shiliewrain/shiliewrain.github.io/master/img/SpringApplication-init.png)]()

