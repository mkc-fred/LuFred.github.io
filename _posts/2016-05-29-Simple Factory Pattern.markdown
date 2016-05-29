---
layout: post
title: "简单工厂模式"
subtitle: "每一个模式描述了一个在外面周围不断重复发生的问题，以及该问题的解决方案的核心。这样，你就能一次又一次地使用该方案而不必做重复劳动。"
subtitleAuthor: "——克里斯托弗·亚历山大"
date: 2016-05-13
author: LuJiangBo
category: Software design pattern
tags: 设计模式
finished: true
---

## 场景假设 
&emsp;&emsp;有这样一个应用场景，月底到了，公司需要给所有的员工按照他们的职位结算当月的薪资，现在公司里面分别有部门主管、部门经理、部门员工，这3大类职位，那财务部需要按照他们的职位分别通过公式去计算他们应得的工资。有一天，老板来视察发现大家都好辛苦，决定招聘几个鼓励师给大家打打气，于是部门又多了一个新职位叫做鼓励师，这时候财务部月度算薪资的时候该怎么办？



## 场景分析

&emsp;&emsp;按照上面的假设，这时候财务必须知道原先3个职位薪资的计算公式，然后再去问老板要鼓励师薪资的结算公式。那么问题又来了，如果财务看到部门主管的交通补贴这一项是1000块而自己才100块不满而把主管的也改成100块怎么办？又或者财务不小心把新加的鼓励师的薪资公式和部门经理的薪资结构互换了怎么办？如果你的项目中也有类似的问题，那这时候就该考虑是否可以用简单工厂模式了。

## 意图

&emsp;&emsp;由工厂类来取代产品类的实例化动作，根据传入的参数，动态的决定应该创建一个产品类。


## 适用性 
下列情况可以考虑使用简单工厂模式: 

* 工厂类负责创建的对象比较少  
* 客户端对于创建对象不关心，只知道传入工厂类的参数

## 结构 
此模式的结构图下图所示。  
	![uml结构图]({{ post.url| prepend: site.url  }}/content/images/201605/2016-05-29-SimpleFactory01.png) 

## 参与者
* Factory: (工厂角色)  
	－实现创建具体产品对象的操作。
* Product: (抽象产品角色)  
	－为一类对象定义共有的公共接口。  
* ConcreteProduct: (具体的产品对象)  
	－定义一个将被工厂对象创建的产品对象。


## 协作   

通常在运行时刻，通过调用工厂的创建方法，通过传入不同的参数来创建不同的产品对象。

## 代码实现
{% highlight c# %}  

//工厂类
public class EmployeeFactory
｛
    public static Employee CreateEmployee(string EmployeeType)
    {
        Employee employee=null;
        switch (EmployeeType)
        {
            case "主管":
                employee=new DirectorEmployee();
                break;
            case "经理":
                employee=new ManagerEmployee();
                break;
            case "普通员工":
                employee=new OrdinaryEmployee();
                break;
            
        }
        return employee;
    }
｝
//员工抽象类
public Employee
{
    public virtual double GetSalaryResult(){
        //...
    }
}
//主管对象
public DirectorEmployee:Employee
{
    public override double GetSalaryResult(){
        //...
    }
}
//经理对象
public ManagerEmployee:Employee
{
    public override double GetSalaryResult(){
        //...
    }
}
//普通员工对象
public OrdinaryEmployee:Employee
{
    public override double GetSalaryResult(){
        //...
    }
}  
{% endhighlight %}

## 场景回顾  
对于场景回顾中的2个问题，此刻我们可以很轻易的得到解决方案。  
1.财务知道了个职位的薪资构成如果擅自更改怎么办？  
&emsp;&emsp;通过简单工厂模式，此时的财务不需要知道每个职位的薪资计算公式，她只需要通过工厂类的*CreateEmployee*方法通过传入相应的职位名，就能得到特定职位的对象，再调用*GetSalaryResult*方法就能计算出响应的薪资。  

2.如果部门新加了鼓励师职位怎么办？  
&emsp;&emsp;针对这个问题，我们只要创建一个鼓励师类，让其集成*Employee*类，重写计算薪资的*GetSalaryResult*方法。然后在工厂类中新加一个鼓励师的case即可。

## 模式优点  
* 具体产品从客户端代码中分离出来，客户端免除了直接接触创建产品的责任，而仅仅是去消费产品；
* 新产品的添加只需要在工厂中新加一条case，而不会影响到以后产品；

## 模式缺点  
* 由于所有的产品都是通过一个工厂类来创建，一旦该工厂不能正常工作，则整个系统都会收到影响;
* 当产品种类越来越多时，简单工厂将会增加系统中类的个数，增加了系统点复杂度，同时也违背了开闭原则；
* 由于创建特定产品是通过工厂来判断，则在添加新产品时不得不修改工厂类的创建逻辑，在产品较多时，可能会造成工厂类的逻辑过于复杂，不利于后期维护；







