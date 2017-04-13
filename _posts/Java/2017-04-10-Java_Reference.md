---
layout: post
title: Java 强/软/弱/虚引用介绍
category: Java
keywords:
description:
---

## 简介
从JDK1.2开始，引入了强引用, 软引用, 弱引用和虚引用等概念，
其引用级别强引用 &gt; 软引用 &gt; 弱引用 &gt; 虚引用。

## 引用介绍

### 强引用
强引用为最普通的引用，如`StringBuilder builder = new StringBuilder()`。如果一个对象具有强引用，那么它不会被垃圾回收器回收，
当内存空间不足时，JVM将抛出`OutOfMemoryError`。

### 软引用(SoftReference)
软引用用来引用一些有用但并非必须的对象，如缓存。当内存不足时，GC会把这些对象在第二次GC时回收。

### 弱引用(WeakReference)
比SoftReference的引用级别更弱一些，当GC工作时，不管内存是否足够，它都将把对象回收。

### 虚引用(PhantomReference)
也称幽灵引用，为最弱的一种引用关系，一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。
调用`PhantomReference.get()`都会得到`null`。


## 实例

实例验证

    package com.learning.reference;
    
    import java.lang.ref.PhantomReference;
    import java.lang.ref.ReferenceQueue;
    import java.lang.ref.SoftReference;
    import java.lang.ref.WeakReference;
    
    public class ReferenceDemo {
    
        public static void main(String[] args) {
            softReferenceTest();
            weakReferenceTest();
            phantomReferenceTest();
        }
    
        public static void softReferenceTest() {
            SoftReference<Person> person = new SoftReference<>(new Person("james"));
    
            System.out.println("soft: " + person.get());
            System.gc();
            System.out.println("after gc soft: " + person.get());
        }
    
        public static void weakReferenceTest() {
            WeakReference<Person> person = new WeakReference<>(new Person("steve"));
    
            System.out.println("weak: " + person.get());
            System.gc();
            System.out.println("after gc weak: " + person.get());
        }
    
        public static void phantomReferenceTest() {
            ReferenceQueue<Person> referenceQueue = new ReferenceQueue<>();
            PhantomReference<Person> person = new PhantomReference<>(new Person("trump"), referenceQueue);
    
            System.out.println("phantom: " + person.get());
            System.gc();
            System.out.println("after gc phantom: " + person.get());
        }
    
        private static class Person {
            private final String name;
    
            public Person(String name) {
                this.name = name;
            }
    
            public String getName() {
                return this.name;
            }
    
            @Override
            public String toString() {
                return this.name;
            }
    
            @Override
            protected void finalize() throws Throwable {
                System.out.println(this.name + " recycle... ");
            }
        }
    }

### 运行结果

设置JVM参数：`-XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps  -Xloggc:reference_gc.log` 观察GC，结果如下：

    soft: james
    after gc soft: james
    weak: steve
    steve recycle... 
    james recycle... 
    after gc weak: null
    phantom: null
    trump recycle... 
    after gc phantom: null

`Person`类实现了保护的`finalize()`方法，当GC回收对象时，将可能调用其`finalize()`方法，但并不保证一定会调用。

在实例中，通过`System.gc()`建议JVM进行垃圾回收，但也不保证一定会进行GC操作，另外，在实际程序中，并不建议显示调用`System.gc()`。

通过设定JVM参数，可以显示在此次运行中，进行了3次GC，即每次调用了`System.gc()`都触发了GC。

从运行结果中可以观察到，软引用在第二次GC中被回收；弱引用在第一次GC中即被回收；虚引用无法通过获得关联的对象，它可用来跟踪GC回收的过程。

`WeakReference`在集合`WeakHashMap`得到应用，`WeakHashMap`的键为弱键，当弱键关联的对象不再被强引用时，它会被GC回收，
弱键被添加到`ReferenceQueue`中，相应的元素也将被`WeakHashMap`移除。