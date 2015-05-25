---
layout: post
title: 数组复制性能比较
category: Java
tags: 数组复制
keywords: 
description: 
---

## 简介
平常数组复制用的比较多，一些性能优化的书籍上推荐使用System.arraycopy()代替自己通过for循环的复制。
现在基于此进行一个简单的测试。

    public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);

## 测试代码

	public class ArrayCopyExample {

		public static void main(String[] args) {
			
			final int ARRAY_SIZE = 100000;
			final int ITER_COUNT = 10000;
			int[] srcArr = new int[ARRAY_SIZE];
			int[] destArr = new int[ARRAY_SIZE];
			
			// init
			for(int i = 0; i < ARRAY_SIZE; ++i) {
				srcArr[i] = i;
			}
			
			// Base foreach copy
			long beginTime = System.currentTimeMillis();
			for(int j = 0; j < ITER_COUNT; ++j)
				for(int i = 0; i < ARRAY_SIZE; ++i) {
					destArr[i] = srcArr[i];
				}
			long totalMS = System.currentTimeMillis() - beginTime;
			System.out.println("BaseCopy: " + totalMS + " ms");
			
			// Base arraycopy()
			beginTime = System.currentTimeMillis();
			for(int j = 0; j < ITER_COUNT; ++j)
				System.arraycopy(srcArr, 0, destArr, 0, ARRAY_SIZE);
			totalMS = System.currentTimeMillis() - beginTime;
			System.out.println("System.arraycopy(): " + totalMS + " ms");
		}
	}



ITER_COUNT分别设为1000,10000。
输出分别为：
BaseCopy: 31 ms
System.arraycopy(): 32 ms

BaseCopy: 250 ms
System.arraycopy(): 312 ms

JDK版本为1.7.0_71
经过多次测试，次数为1000时，两者的时间基本接近，
但次数为10000，System.arraycopy()效率反而低。

难道是JDK对此进行了优化？从其他资料上看到System.arraycopy()的效率高于普通的复制7-8倍。