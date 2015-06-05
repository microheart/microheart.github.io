---
layout: post
title: Java注解与拦截器实例(二)
category: Java
tags: 注解，反射
keywords: 
description: 
---

## 简介

上一篇对注解一些基础内容进行了介绍，这篇通过一个简单的拦截器实例来说明如何运用注解。
在Web开发中，我们会用到拦截器这些工具，通过拦截器可以达到AOP编程，减少代码的耦合。Struts2中定义了非常多的拦截器。

本篇通过在Action类或者方法上配置注解，利用反射机制，达到某个路径(通过类型和方法组合生成)与一系列拦截器的关联，而不使用配置文件。

## 定义注解

定义Before注解，用于在类或者方法时，可在运行时读取。可配置继承Interceptor接口的Class数组。

	import java.lang.annotation.ElementType;
	import java.lang.annotation.Inherited;
	import java.lang.annotation.Retention;
	import java.lang.annotation.RetentionPolicy;
	import java.lang.annotation.Target;

	@Inherited
	@Retention(RetentionPolicy.RUNTIME)
	@Target({ElementType.TYPE, ElementType.METHOD})
	public @interface Before {
		Class<? extends Interceptor>[] value();
	}

## 定义拦截器

拦截器接口，为了方便模拟，方法的参数为Object对象

	public interface Interceptor {
		void intercept(Object obj);
	}

定义了三个实现Interceptor接口的类，分别为CommonInterceptor，AuthorityInterceptor和CacheInterceptor。

	public class CommonInterceptor implements Interceptor {
		@Override	
		public void intercept(Object obj) {
			System.out.println("Common Interceptor invoke ... ");
		}
	}

	public class AuthorityInterceptor implements Interceptor {
		@Override	
		public void intercept(Object obj) {
			System.out.println("Authority Interceptor invoke ... ");
		}
	}

	public class CacheInterceptor implements Interceptor {
		@Override	
		public void intercept(Object obj) {
			System.out.println("Cache Interceptor invoke ... ");
		}
	}


## 定义Action控制类

对Action类添加了注解和方法上添加了0到多个拦截器。

	@Before(CommonInterceptor.class)
	public class Action {

		@Before(AuthorityInterceptor.class)
		public void save() {
			System.out.println("foo");
		}

		@Before({CacheInterceptor.class, AuthorityInterceptor.class})
		public void view() {
			System.out.println("view");
		}

		public void list() {
			System.out.println("list");
		}
	}

## 定义注解处理器

方法上面的拦截器集合有方法上自定义的拦截器和类上定义的拦截器所组成。
步骤：(1) 取得Class上的Before注解，获得此Class上配置的拦截器；(2) 对Class的每个方法获得Method对象，并获得在此Method上配置的拦截器；(3) 将上面两步获得的拦截器合并，对应方法上所配置的拦截器集合。(4) 配置路由和拦截器的关联关系，路由简单的由类名和方法名组成。

	import java.util.HashMap;
	import java.util.Map;
	import java.lang.reflect.Method;

	public class InterceptorParse {

		private static final Interceptor[] NULL_INTERCEPTOR_ARRAY = new Interceptor[0];

		public Map<String, Interceptor[]> parse(Class klass) {
			Map<String, Interceptor[]> result = new HashMap<String, Interceptor[]>();

			String className = klass.getName();

			Before classBefore = (Before)klass.getAnnotation(Before.class);
			Interceptor[] classInterceptors = getInterceptors(classBefore);

			//Method[] methods = klass.getMethods();
			Method[] methods = klass.getDeclaredMethods();
			for (Method method : methods) {
				String methodName = method.getName();

				Before methodBefore = (Before)method.getAnnotation(Before.class);
				Interceptor[] methodInterceptors = getInterceptors(methodBefore);

				Interceptor[] availableInterceptors = combineInterceptors(classInterceptors, methodInterceptors);
				
				result.put(className+"/"+methodName, availableInterceptors);
			}

			return result;
		}

		private Interceptor[] getInterceptors(Before beforeAnnotation) {
			if(beforeAnnotation == null)
				return NULL_INTERCEPTOR_ARRAY;

			Interceptor[] result = null;
			Class<Interceptor>[] interceptorClasses = (Class<Interceptor>[]) beforeAnnotation.value();
			if (interceptorClasses != null && interceptorClasses.length > 0) {
				result = new Interceptor[interceptorClasses.length];
				for (int i=0; i<result.length; i++) {
					try {
						result[i] = (Interceptor)interceptorClasses[i].newInstance();
					} catch (Exception e) {
						throw new RuntimeException(e);
					}
				}
			}
			return (result != null)?result:NULL_INTERCEPTOR_ARRAY;
		}

		private Interceptor[] combineInterceptors(Interceptor[] first, Interceptor[] second) {
			if(first.length == 0)
				return second;
			if(second.length == 0)
				return first;

			Interceptor[] result = new Interceptor[first.length + second.length];
			int idx = 0;
			for(Interceptor interceptor: first) {
				result[idx++] = interceptor;
			}
			for(Interceptor interceptor: second) {
				result[idx++] = interceptor;
			}

			return result;
		}

		public static void main(String[] args) {
			InterceptorParse interceptorParse = new InterceptorParse();
			Map<String, Interceptor[]> map = interceptorParse.parse(Action.class);

			for (Map.Entry<String, Interceptor[]> entry : map.entrySet()) {  
			    System.out.println("path: " + entry.getKey());

			    Interceptor[] interceptors = entry.getValue();
			    for(Interceptor interceptor: interceptors) {
			    	System.out.println("Interceptor:" + interceptor.getClass().getName());
			    	interceptor.intercept(null);
			    }

			    System.out.println();
			}  
		}

	}

运行结果

	path: Action/list
	Interceptor:CommonInterceptor
	Common Interceptor invoke ...

	path: Action/view
	Interceptor:CommonInterceptor
	Common Interceptor invoke ...
	Interceptor:CacheInterceptor
	Cache Interceptor invoke ...
	Interceptor:AuthorityInterceptor
	Authority Interceptor invoke ...

	path: Action/save
	Interceptor:CommonInterceptor
	Common Interceptor invoke ...
	Interceptor:AuthorityInterceptor
	Authority Interceptor invoke ...


实际的开发中，要比这个更加严格和灵活，比如路由的配置，拦截器大多可以使用单例模式，而不需要创建多个。