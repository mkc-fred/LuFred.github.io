---
layout: post
title: "单例模式"
subtitle: "单例模式."
date: 2016-05-13
author: LuJiangBo
category: Software design pattern
tags: 设计模式
finished: true
---

## 作用   

>&emsp;&emsp;绕过常规实例的构造器，提供一种机制保证一个类只有一个实例。  

## 意图

>&emsp;&emsp;保证一个类仅有一个实例，并提供一个该实例的全局访问点

## 缺点  
>* 一般不要支持ICloneable结果，因为这可能会导致多个对象实例
>* 一般不要支持序列化，这样可能会导致多个实例的创建
>* 只考虑类对象创建的管理，没有考虑对象销毁的管理

## 实现方式
{% highlight c# %}
//No.1 隐藏默认构造函数，抛出静态方法，实例话静态字段
//不支持多线程
public class Singleton
{
	private static Singleton instance;
	
	private Singleton(){}
	
	public static Singleton Instance{
		get{
			if(instance==null){
				instance=new Singleton();
			}
			return instance;
		}
	} 
}

//No.2 静态字段只有在姿态构造函数中初始化一次，所以只会new一次
//不支持多线程
public class Singleton
{
	private static Singleton instance=new Singleton();
	
	private Singleton(){}
	
	public static Singleton Instance{
		get{
			return instance;
		}
	} 
}

//No.3  volatile+look组合 多线程单例
//volatile:防治编译后的指令序列与代码顺序不一致
public class Singleton
{
	private static volatile Singleton instance=null;
	
	private static object lookHelper=new object();
	private Singleton(){}
	
	public static Singleton Instance{
		get{
			if(instance==null){
				look(lookHelper){
					if(instance==null){
						instance=new Singleton();
					}
				}
			}
			return instance;
		}
	} 
}
//No.4 readonly关键字
//内联初始化，即声明的时候直接初始化
//Instance在静态构造函数中初始化
//类似于No.2方法
//缺点：不支持参数化                                                              
public class Singleton
{
	private static readonly Singleton Instance=new Singleton();
	
	private Singleton(){}
	
}
{% endhighlight %}