---
layout: post
title: Java注解与拦截器实例(一)
category: Java
tags: 注解，反射
keywords: 
description: 
---

## 简介
Annotation(注解)，也叫元数据。一种代码级别的说明，是JDK5.0引入的。它可以用于创建文档，跟踪代码中的依赖性，甚至执行基本编译时检查。
Annotion是一个接口，程序可以通过反射来获取指定程序元素的Annotion对象，然后通过Annotion对象来获取注解里面的元数据。

## 注解基础

Annotation能被用来为某个程序元素（类、方法、成员变量等）关联任何的信息。Annotation不影响程序代码的执行，无论增加、删除 Annotation，代码都始终如一的执行，但可以通过Java反射工具对Annotation进行访问和处理。

根据注解使用方法和用途，可以将Annotation分为三类：

### JDK内置系统注解

注解的语法比较简单，除了@符号的使用外，他基本与Java固有的语法一致，JavaSE中内置三个标准注解，定义在java.lang中：

* @Override：用于修饰此方法覆盖了父类的方法;
* @Deprecated：用于修饰已经过时的方法，可能在未来版本废弃;
* @SuppressWarnnings:用于通知java编译器禁止特定的编译警告。

### 元注解

元注解的作用就是负责注解其他注解。Java5.0定义了4个标准的meta-annotation类型，它们被用来提供对其它 annotation类型作说明。

* @Documented
* @Target
* @Retention
* @Inherited

这些类型和它们所支持的类在java.lang.annotation包中可以找到。

#### Documented

	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.ANNOTATION_TYPE)
	public @interface Documented {
	}

@Documented可以用于javadoc此类的工具文档化。Documented是一个标记注解，没有成员。

#### Target注解

	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.ANNOTATION_TYPE)
	public @interface Target {
	    ElementType[] value();
	}

@Target说明了Annotation所修饰的对象范围，它所支持的范围在ElementType枚举中描述。

	public enum ElementType {
	    TYPE, /** Class, interface (including annotation type), or enum declaration */
	    FIELD,  /** Field declaration (includes enum constants) */
	    METHOD, /** Method declaration */
	    PARAMETER,  /** Parameter declaration */
	    CONSTRUCTOR, /** Constructor declaration */
	    LOCAL_VARIABLE,  /** Local variable declaration */
	    ANNOTATION_TYPE,  /** Annotation type declaration */
	    PACKAGE /** Package declaration */
	}

#### Retention注解

	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.ANNOTATION_TYPE)
	public @interface Retention {
	    RetentionPolicy value();
	}

@Retention定义了该Annotation被保留的时间长短。

	public enum RetentionPolicy {
	    SOURCE,
	    CLASS,
	    RUNTIME
	}

* SOURCE：被编译器丢弃，仅源文件保留
* CLASS：编译器记录注解在class文件里面，运行时被虚拟机丢弃，默认。
* RUNTIME：编译器记录注解在class文件里面，运行时虚拟机会保留，可通过反射读取。

#### Inherited注解

	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.ANNOTATION_TYPE)
	public @interface Inherited {
	}

@Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。


### 自定义注解

使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口。


定义注解格式：
	
	public @interface AnnotationName {body...}

注解的每一个方法实际上是声明了一个配置参数。方法的名称就是参数的名称。
注解参数的可支持数据类型：基本数据类型，String，CLass，enum，Annotation及前面类型的数组。
可以通过default来声明参数的默认值。

仅仅定义注解还不够，重要的是要通过注解处理器来处理自定义的注解，Java SE5扩展了反射机制的API，以帮助程序员快速的构造自定义注解处理器。

Java使用Annotation接口来代表程序元素前面的注解，该接口是所有Annotation类型的父接口。除此之外，Java在java.lang.reflect 包下新增了AnnotatedElement接口，该接口代表程序中可以接受注解的程序元素。

	public interface AnnotatedElement {
	    /** 判断该程序元素上是否包含指定类型的注解，存在则返回true，否则返回false.   */
	     boolean isAnnotationPresent(Class<? extends Annotation> annotationClass);

	   /** 返回存在的指定类型的注解，否则返回null  */
	    <T extends Annotation> T getAnnotation(Class<T> annotationClass);

	    /** 返回所有注解   */
	    Annotation[] getAnnotations();

	    /** 返回直接存在于此元素上的所有注释。  */
	    Annotation[] getDeclaredAnnotations();
	}

Class, Constructor, Field, Method和Package都实现了AnnotatedElement接口。
