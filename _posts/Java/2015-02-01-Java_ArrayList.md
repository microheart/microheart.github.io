---
layout: post
title: Java容器ArrayList
category: Java
tags: Java
keywords: 
description: 
---


## 容器对null的特殊处理

AbstractCollection实现Collection接口，允许包含null元素，在contains()和remove()方法中需要对null进行特殊处理，不能简单调用equal()方法进行比较。
ArrayList和LinkedList继承AbstractList，AbstractList继承AbstractCollection。

    public boolean contains(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return true;
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return true;
        }
        return false;
    }

    public boolean remove(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext()) {
                if (it.next()==null) {
                    it.remove();
                    return true;
                }
            }
        } else {
            while (it.hasNext()) {
                if (o.equals(it.next())) {
                    it.remove();
                    return true;
                }
            }
        }
        return false;
    }    

ArrayList.indexOf()/lastIndexOf()很多方法都对null进行了特殊的处理。
AbstractMap.containsValue()/containsKey()/get()/remove()等一系列方法也对null进行了处理。


## ArrayList

ArrayList默认元素数组大小为10。

	private static final int DEFAULT_CAPACITY = 10;

看过ArrayList源码的也许对底层存储的数组elementData前面加了transient关键字很疑惑，这样数组怎么序列化？
因为变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。
    
    private transient Object[] elementData;

ArrayList重写了writeObject()/readObject()方法，自定义了序列化行为。
从代码中可以看到，ArrayList并没有把整个elementData数组的元素都进行了序列化，而只序列化了size个元素，仅序列化实际存储的元素，而不是整个数组，这样能有效的节省空间。elementData是一个缓存数组。

    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }    

ArrayList进行扩展时，通常已1.5倍的大小进行扩展(需要考虑最大，最小元素个数)。

	int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);

与ArrayList类似，LinkedList也重写了writeObject()/readObject()方法，LinkedList使用双向循环链表存储元素。

    transient int size = 0;
    transient Node<E> first;
    transient Node<E> last;