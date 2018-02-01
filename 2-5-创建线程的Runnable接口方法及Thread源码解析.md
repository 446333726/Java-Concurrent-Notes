---
layout:

title: 2-5-创建线程的Runnable接口方法及Thread源码解析 

date: 2015-10-06

updated: 2015-10-06

tags:
- Java多线程
- Java多线程基础

categories: Java多线程基础

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---

# 创建线程的Runnable接口方法

>public interface Runnable

>Runnable 接口应该由那些打算通过某一线程执行其实例的类来实现。类必须定义一个称为 run 的无参数方法。 

>设计该接口的目的是为希望在活动时执行代码的对象提供一个公共协议。例如，Thread 类实现了 Runnable。激活的意思是说某个线程已启动并且尚未停止。 

>此外，Runnable 为非 Thread 子类的类提供了一种激活方式。通过实例化某个 Thread 实例并将自身作为运行目标，就可以运行实现 Runnable 的类而无需创建 Thread 的子类

>**大多数情况下，如果只想重写 run() 方法，而不重写其他 Thread 方法，那么应使用Runnable接口。这很重要，因为除非程序员打算修改或增强类的基本行为，否则不应为该类创建子类。** 

## 使用方法

* 定义类实现runnable接口(避免单继承的局限性)
* 覆盖接口中run方法，将线程任务代码定义到run方法中
* 创建thread类的对象(只有通过线程对线才能操作线程方法)
* 将runnable接口的子类对象作为参数传递给Thread类的构造函数
* 调用Thread类的Start方法开启线程

## 代码示例-1


```
/**
 * Created by Egg on 2016/10/21.
 *创建线程的第二种方式
 * 1. 定义类实现runnable接口(避免单继承的局限性)
 * 2. 覆盖接口中run方法，将线程任务代码定义到run方法中
 * 3. 创建thread类的对象(只有通过线程对线才能操作线程方法)
 * 4. 将runnable接口的子类对象作为参数传递给Thread类的构造函数
 * 5. 调用Thread类的Start方法开启线程
 *
 * 第二种方式实现runnable接口避免了单继承的局限性，所以较为常用
 *
 *       public Thread(Runnable target) {
         init(null, target, "Thread-" + nextThreadNum(), 0);
         }
         @Override
         public void run() {
             if (target != null) {
             target.run();
             }
         }
 */
class Demo implements Runnable
{
    private String name;
    Demo(String name)
    {
        this.name = name;
    }
    public void run()
    {
        for (int i = 1 ; i < 20 ; i ++)
        {
            System.out.println(name + i);
        }
    }

}
public class ThreadDemo
{
    public static void main(String[] args) {
        /*
        * 创建Runnable子类的对象
        * 而非线程对象
        * 此处runnable对象是Thread类的一个参数
        * */
        Demo d1 = new Demo("AAA");
        Demo d2 = new Demo("BBB");
        Demo d3 = new Demo("CCC");
        Demo d4 = new Demo("DDD");
        Thread t1 = new Thread(d1);
        Thread t2 = new Thread(d2);
        Thread t3 = new Thread(d3);
        Thread t4 = new Thread(d4);
        t2.start();
        d1.run();

        t3.start();
        t4.run();
    }
}
```

## 使用Runnable接口的好处

>假设Student类继承了Person,但现在有一个需求,Student中有一段CODE需要多线程执行,然而Java只支持单继承.

>可以让Person也继承Thread,然而这会让所有的Person的子类也继承Thread所以并不是一种好的办法.

>所以我们让Student类实现Runnable接口

![image](http://clsaajavaimgbed-10042610.cos.myqcloud.com/DemoMul2.png)

>Runnable接口避免了单继承的局限性,所以较为常用.

>实现runnable接口的方式更加面向对象,线程分为两部分,一部分是线程对象,一部分是线程任务.继承Thread类会让线程对象和线程任务耦合在一起.一旦创建Thread类的子对象,既是线程对象又有线程任务,实现runnable接口将线程任务单独分离出来封装为对象,对线程任务和线程对象解耦.

## 源码解析

* 定义类实现runnable接口(避免单继承的局限性)
* 覆盖接口中run方法，将线程任务代码定义到run方法中
* 创建thread类的对象(只有通过线程对线才能操作线程方法)
* 将runnable接口的子类对象作为参数传递给Thread类的构造函数(提供run方法)
* 调用Thread类的Start方法开启线程


```
    /**
     * If this thread was constructed using a separate
     * <code>Runnable</code> run object, then that
     * <code>Runnable</code> object's <code>run</code> method is called;
     * otherwise, this method does nothing and returns.
     * <p>
     * Subclasses of <code>Thread</code> should override this method.
     *
     * @see     #start()
     * @see     #stop()
     * @see     #Thread(ThreadGroup, Runnable, String)
     */
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }

```

>可以看到JDK源码中,Thread对象在调用run方法时会判断target如果target不等于null那么就会调用target的run方法,那么target是什么呢?


```
    /* What will be run. */
    private Runnable target;
    
     /**
     * Allocates a new {@code Thread} object. This constructor has the same
     * effect as {@linkplain #Thread(ThreadGroup,Runnable,String) Thread}
     * {@code (null, target, gname)}, where {@code gname} is a newly generated
     * name. Automatically generated names are of the form
     * {@code "Thread-"+}<i>n</i>, where <i>n</i> is an integer.
     *
     * @param  target
     *         the object whose {@code run} method is invoked when this thread
     *         is started. If {@code null}, this classes {@code run} method does
     *         nothing.
     */
    public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }
    
    /**
     * Initializes a Thread with the current AccessControlContext.
     * @see #init(ThreadGroup,Runnable,String,long,AccessControlContext)
     */
    private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize) {
        init(g, target, name, stackSize, null);
    }

```

>原来target是Thread类的一个Runnable类型的成员变量,其在构造函数中被初始化