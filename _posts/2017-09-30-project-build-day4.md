---
laytou : post
title : "项目搭建问题（四）"
category : JavaWeb
tags : 项目实战 springmvc interceptor
---
* content
{:toc}

　　国庆放假的前一天，上班。




　　自然是没有什么效率的。上午把登录拦截的问题给搞定了，采用的是特定注解（@NoNeedLogin）标识该类或者方法不需要验证是否登录。下午来写写实现过程中碰到的问题。

#### 登录拦截

　　设计的方案是对所有的请求进行拦截，针对部分被特定注解（@NoNeedLogin）标识的直接放行，其余则需要验证用户的登录状态。

#### 自定义注解

　　自定义注解@NoNeedLogin，参考之前的文章[自定义注解示例](https://shiliewrain.github.io/2017/07/25/annotation/)。

#### 自定义拦截器

　　自定义拦截器需要实现HandlerInterceptor接口，或者继承实现该接口的子类。然后主要实现其中preHandle()方法。

```java
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		HandlerMethod handlerMethod = (HandlerMethod) handler;//1
		//something to do
		return true;
	}
```

　　在原来项目的实现中，用到了上面1的代码，因此我也使用到了。但在请求被拦截之后，执行该代码会报错，这是因为handler不能被强转为HandlerMethod。追踪两个项目的源码，我发现两个项目执行preHandle的方法不是一样的，这时才发现两个项目的Spring-mvc-web包的版本是不一样的，一个是4.2.8，一个是3.1.6。

　　网上查找解决方案，发现了一篇相同问题的文章，[Spring拦截器中通过request获取到该请求对应Controller中的method对象](http://chenzhou123520.iteye.com/blog/1702563)。

　　按照文中的解决方案，我的项目也OK了。主要是在dispatch-servlet.xml中加入以下配置：

```xml
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">   
	    <property name="interceptors">  
	        <list>  
	            <bean class="com.xxx.interceptor.FrontInterceptor"/>  
	        </list>  
	    </property>  
	</bean> 
	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>
```

　　大致的原因是默认使用的DefaultAnnotationHandlerMapping只会返回请求的Controller对象，不会映射到方法上，就造成了强转失败。而使用RequestMappingHandlerMapping，会采用RequestMappingHandlerAdapter将请求封装为HandlerMethod的实例。

### 扩展阅读

[springMVC源码分析--HandlerInterceptor拦截器（一）](http://blog.csdn.net/qq924862077/article/details/53524507)

