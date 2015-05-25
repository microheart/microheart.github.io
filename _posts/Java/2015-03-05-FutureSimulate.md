---
layout: post
title: 模拟Future模式
category: Java
tags: Future
keywords: 多线程
description: 
---

## 简介
Future模式属于Java并发编程的一种，是一种异步调用方式，若我们请求某个服务不能立即得到结果，
我们可以先拿到一个伪结果，得到可以并发的事情做完后，再去得到真实的结果，利于并发。
此文模拟一个简单的Future模式的实现方式，当然Java的内置实现比此复制并功能更加强大，但其核心概念是一样的。

## 角色分配

+ FutureSimulate：程序入口，调用Client获取结果和处理其他逻辑
+ Client: 客户端，发送请求，得到FutureData，启动额外线程创建RealData对象，并将RealData对象设置给FutureData对象
+ Data: 数据接口，FutureData和RealData都实现此接口
+ FutureData: 伪结果对象，构造迅速，立即返回，但得到最终的结果需要依靠RealData对象。
+ RealData：实际数据对象，构造缓慢

## 代码实现


模拟程序，通过Thread.sleep()来模拟其他业务逻辑的处理

	public class FutureSimulate {

		public static void main(String[] args) {
			Client client = new Client();
			Data data = client.request("future simulate");
			
			System.out.println("do other thing... ");
			
			// do something
			try {
				//Thread.sleep(2000);
				Thread.sleep(10000);
			} catch (InterruptedException e) {
			}
			
			System.out.println("get result... ");
			String result = data.getResult();
			
			System.out.println("result: " + result);
		}
	}

Client发送请求，并立即获得FutureData对象，与此同时，启动一个线程构造RealData对象。

	class Client {
		public Data request(final String query) {
			final FutureData futureData = new FutureData();
			
			new Thread() {
				public void run() {
					RealData realData = new RealData(query);
					futureData.setRealData(realData);
				}
			}.start();
			
			return futureData;
		}
	}

数据接口
	
	interface Data {
		public String getResult();
	}

FutureData，Future模式的核心，相当于RealData的一个代理，能够快速构造返回，当调用getResult()方法时将一直阻塞，直到真实的RealData对象已经构造完毕并填充到FutureData中。

	class FutureData implements Data {
		private RealData realData;
		private volatile boolean isReady = false;
		
		public synchronized void setRealData(RealData realData) {
			this.realData = realData;
			isReady = true;
			notifyAll();
		}

		@Override
		public synchronized String getResult() {
			
			while(!isReady) {
				try {
					wait();
				}
				catch(InterruptedException e) {
					// ignore
				}
			}
			return realData.getResult();
		}
		
	}

RealData，真实数据，构造此对象耗时，通过Thread.sleep()模拟

	class RealData implements Data {
		private String result ;
		public RealData(String argument) {
			StringBuilder builder = new StringBuilder();
			builder.append(argument);
			for(int i = 0; i < 5; ++i) {
				try {
					Thread.sleep(1000);
				}
				catch(InterruptedException e) {
					// ignore
				}
				builder.append(i);
			}
			
			result = builder.toString();
		}
		
		@Override
		public String getResult() {
			return result;
		}
	}

## 运行结果
do other thing... 
get result... 
result: future simulate01234

从测试程序中可以发现，构造RealData耗时5秒钟，
若FutureSimulate处理其他业务逻辑花费2秒钟，其得到实际结果只阻塞3秒钟，
若处理其他业务逻辑花费5秒钟以上，则不会被阻塞。

## Java Future
Java内置Future模式中最核心的类为FutureTask，实现了Runnable和Future接口。
因此，可做为一个单独的线程运行。
run()方法中通过通用Callable.call()获得结果。
Callable对象来自于构造方法参数中的Callable对象，或者由构造方法参数Runnable和V对象构造一个Callable对象，

    public void run() {
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
        } finally {
            runner = null;
            int s = state;
            if (s >= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }

FutureTask.get()阻塞，直到获得结果或者抛出异常。

    public V get() throws InterruptedException, ExecutionException {
        int s = state;
        if (s <= COMPLETING)
            s = awaitDone(false, 0L);
        return report(s);
    }