---
layout: post
title: ArrayBlockingQueue
category: Java
tags: ArrayBlockingQueue
keywords: 
description: 
---


ArrayBlockingQueue阻塞队列适合使用在生产者-消费者模式中，例如用在ThreadPoolExecutor线程池中。
这是一个典型的“有界缓存区”，固定大小的数组在其中保持生产者插入的元素和使用者提取的元素。一旦创建了这样的缓存区，就不能再增加其容量。 

ArrayBlockingQueue通过锁和条件变量进行同步。

    /** Main lock guarding all access */
    final ReentrantLock lock;
    /** Condition for waiting takes */
    private final Condition notEmpty;
    /** Condition for waiting puts */
    private final Condition notFull;

添加元素和移除元素将会调用Condition.signal()方法进行唤醒。

    private void insert(E x) {
        items[putIndex] = x;
        putIndex = inc(putIndex);
        ++count;
        notEmpty.signal();
    }

    private E extract() {
        final Object[] items = this.items;
        E x = this.<E>cast(items[takeIndex]);
        items[takeIndex] = null;
        takeIndex = inc(takeIndex);
        --count;
        notFull.signal();
        return x;
    }


阻塞方法take()/put()，如果队列为空，take()将一直等待，直到非空或被中断；如果队列满了，put()将一直等待，直到有空的位置或者被中断。

    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return extract();
        } finally {
            lock.unlock();
        }
    }    

    public void put(E e) throws InterruptedException {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length)
                notFull.await();
            insert(e);
        } finally {
            lock.unlock();
        }
    }  