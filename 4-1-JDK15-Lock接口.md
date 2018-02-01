---
layout:

title: 4-1-JDK1.5-Lock接口

date: 2015-10-12

updated: 2015-10-12

tags:
- Java多线程
- Java多线程基础
- Java多线程JDK1.5

categories: Java多线程基础

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---

# java.util.concurrent.locks 

>JDK1.5以后提供了多生产多消费的解决方案.

>Lock接口:比同步有更多的操作.Lock():获取锁,unlock:释放锁;提供了一个更加面向对象的锁,在该锁中提供了更多的显式的锁操作.替代同步

>在将旧锁替换为新锁后,那么锁上的监视器方法(wait,notify,notifyALL),也应该替换成新锁的监视器方法.而JDK1.5中国将这些原有的监视器方法封装到了一个Condition中

>其实Condition对象的出现其实就是替代了Object中的监视器方法

![image](http://clsaajavaimgbed-10042610.cos.myqcloud.com/QQ%E6%B5%8F%E8%A7%88%E5%99%A8%E6%88%AA%E5%B1%8F%E6%9C%AA%E5%91%BD%E5%90%8D.png)

## Lock

>Lock 实现提供了比使用 synchronized 方法和语句可获得的更广泛的锁定操作。此实现允许更灵活的结构，可以具有差别很大的属性，可以支持多个相关的 Condition 对象。 

>锁是控制多个线程对共享资源进行访问的工具。通常，锁提供了对共享资源的独占访问。一次只能有一个线程获得锁，对共享资源的所有访问都需要首先获得锁。不过，某些锁可能允许对共享资源并发访问，如 ReadWriteLock 的读取锁。 

>synchronized 方法或语句的使用提供了对与每个对象相关的隐式监视器锁的访问，但却强制所有锁获取和释放均要出现在一个块结构中：当获取了多个锁时，它们必须以相反的顺序释放，且必须在与所有锁被获取时相同的词法范围内释放所有锁。 

>虽然 synchronized 方法和语句的范围机制使得使用监视器锁编程方便了很多，而且还帮助避免了很多涉及到锁的常见编程错误，但有时也需要以更为灵活的方式使用锁。例如，某些遍历并发访问的数据结果的算法要求使用 "hand-over-hand" 或 "chain locking"：获取节点 A 的锁，然后再获取节点 B 的锁，然后释放 A 并获取 C，然后释放 B 并获取 D，依此类推。Lock 接口的实现允许锁在不同的作用范围内获取和释放，并允许以任何顺序获取和释放多个锁，从而支持使用这种技术。 

>**随着灵活性的增加，也带来了更多的责任。不使用块结构锁就失去了使用 synchronized 方法和语句时会出现的锁自动释放功能。在大多数情况下，应该使用以下语句**

     Lock l = ...; 
     l.lock();
     try {
         // access the resource protected by this lock
     } finally {
         l.unlock();
     }
>锁定和取消锁定出现在不同作用范围中时，必须谨慎地确保保持锁定时所执行的所有代码用 try-finally 或 try-catch 加以保护，以确保在必要时释放锁。 
Lock 实现提供了使用 synchronized 方法和语句所没有的其他功能，包括提供了一个非块结构的获取锁尝试 (tryLock())、一个获取可中断锁的尝试 (lockInterruptibly()) 和一个获取超时失效锁的尝试 (tryLock(long, TimeUnit))。 

>Lock 类还可以提供与隐式监视器锁完全不同的行为和语义，如保证排序、非重入用法或死锁检测。如果某个实现提供了这样特殊的语义，则该实现必须对这些语义加以记录。 

>注意，Lock 实例只是普通的对象，其本身可以在 synchronized 语句中作为目标使用。获取 Lock 实例的监视器锁与调用该实例的任何 lock() 方法没有特别的关系。为了避免混淆，建议除了在其自身的实现中之外，决不要以这种方式使用 Lock 实例。 

>除非另有说明，否则为任何参数传递 null 值都将导致抛出 NullPointerException。 

>内存同步
所有 Lock 实现都必须 实施与内置监视器锁提供的相同内存同步语义，如 The Java Language Specification, Third Edition (17.4 Memory Model) 中所描述的: 

>成功的 lock 操作与成功的 Lock 操作具有同样的内存同步效应。 
成功的 unlock 操作与成功的 Unlock 操作具有同样的内存同步效应。 
不成功的锁定与取消锁定操作以及重入锁定/取消锁定操作都不需要任何内存同步效果。 
>实现注意事项
三种形式的锁获取（可中断、不可中断和定时）在其性能特征、排序保证或其他实现质量上可能会有所不同。而且，对于给定的 Lock 类，可能没有中断正在进行的 锁获取的能力。因此，并不要求实现为所有三种形式的锁获取定义相同的保证或语义，也不要求其支持中断正在进行的锁获取。实现必需清楚地对每个锁定方法所提供的语义和保证进行记录。还必须遵守此接口中定义的中断语义，以便为锁获取中断提供支持：完全支持中断，或仅在进入方法时支持中断。 

>由于中断通常意味着取消，而通常又很少进行中断检查，因此，相对于普通方法返回而言，实现可能更喜欢响应某个中断。即使出现在另一个操作后的中断可能会释放线程锁时也是如此。实现应记录此行为。 

## Condition

>public interface ConditionCondition 将 Object 监视器方法（wait、notify 和 notifyAll）分解成截然不同的对象，以便通过将这些对象与任意 Lock 实现组合使用，为每个对象提供多个等待 set（wait-set）。其中，Lock 替代了 synchronized 方法和语句的使用，Condition 替代了 Object 监视器方法的使用。 

>条件（也称为条件队列 或条件变量）为线程提供了一个含义，以便在某个状态条件现在可能为 true 的另一个线程通知它之前，一直挂起该线程（即让其“等待”）。因为访问此共享状态信息发生在不同的线程中，所以它必须受保护，因此要将某种形式的锁与该条件相关联。等待提供一个条件的主要属性是：以原子方式 释放相关的锁，并挂起当前线程，就像 Object.wait 做的那样。 

>Condition 实例实质上被绑定到一个锁上。要为特定 Lock 实例获得 Condition 实例，请使用其 newCondition() 方法。 

>作为一个示例，假定有一个绑定的缓冲区，它支持 put 和 take 方法。如果试图在空的缓冲区上执行 take 操作，则在某一个项变得可用之前，线程将一直阻塞；如果试图在满的缓冲区上执行 put 操作，则在有空间变得可用之前，线程将一直阻塞。我们喜欢在单独的等待 set 中保存 put 线程和 take 线程，这样就可以在缓冲区中的项或空间变得可用时利用最佳规划，一次只通知一个线程。可以使用两个 Condition 实例来做到这一点。 


```
class BoundedBuffer {
   final Lock lock = new ReentrantLock();
   final Condition notFull  = lock.newCondition(); 
   final Condition notEmpty = lock.newCondition(); 

   final Object[] items = new Object[100];
   int putptr, takeptr, count;

   public void put(Object x) throws InterruptedException {
     lock.lock();
     try {
       while (count == items.length) 
         notFull.await();
       items[putptr] = x; 
       if (++putptr == items.length) putptr = 0;
       ++count;
       notEmpty.signal();
     } finally {
       lock.unlock();
     }
   }

   public Object take() throws InterruptedException {
     lock.lock();
     try {
       while (count == 0) 
         notEmpty.await();
       Object x = items[takeptr]; 
       if (++takeptr == items.length) takeptr = 0;
       --count;
       notFull.signal();
       return x;
     } finally {
       lock.unlock();
     }
   } 
 }
```

 （ArrayBlockingQueue 类提供了这项功能，因此没有理由去实现这个示例类。） 
>Condition 实现可以提供不同于 Object 监视器方法的行为和语义，比如受保证的通知排序，或者在执行通知时不需要保持一个锁。如果某个实现提供了这样特殊的语义，则该实现必须记录这些语义。 

>注意，Condition 实例只是一些普通的对象，它们自身可以用作 synchronized 语句中的目标，并且可以调用自己的 wait 和 notification 监视器方法。获取 Condition 实例的监视器锁或者使用其监视器方法，与获取和该 Condition 相关的 Lock 或使用其 waiting 和 signalling 方法没有什么特定的关系。为了避免混淆，建议除了在其自身的实现中之外，切勿以这种方式使用 Condition 实例。 

>除非另行说明，否则为任何参数传递 null 值将导致抛出 NullPointerException。 

>实现注意事项
在等待 Condition 时，允许发生“虚假唤醒”，这通常作为对基础平台语义的让步。对于大多数应用程序，这带来的实际影响很小，因为 Condition 应该总是在一个循环中被等待，并测试正被等待的状态声明。某个实现可以随意移除可能的虚假唤醒，但建议应用程序程序员总是假定这些虚假唤醒可能发生，因此总是在一个循环中等待。 

>三种形式的条件等待（可中断、不可中断和超时）在一些平台上的实现以及它们的性能特征可能会有所不同。尤其是它可能很难提供这些特性和维护特定语义，比如排序保证。更进一步地说，中断线程实际挂起的能力在所有平台上并不是总是可行的。 

>因此，并不要求某个实现为所有三种形式的等待定义完全相同的保证或语义，也不要求其支持中断线程的实际挂起。 

>要求实现清楚地记录每个等待方法提供的语义和保证，在某个实现不支持中断线程的挂起时，它必须遵从此接口中定义的中断语义。 

>由于中断通常意味着取消，而又通常很少进行中断检查，因此实现可以先于普通方法的返回来对中断进行响应。即使出现在另一个操作后的中断可能会释放线程锁时也是如此。实现应记录此行为。 

##　多生产多消费的效率问题

> 多生产多消费问题的效率问题产生原因是:生产方会唤醒生产方,如果生产方只唤醒消费方则可以避免多唤醒问题

>利用JDK1.5的LOCK和Condition可以在一个锁上挂多组监视器,则可以在生产方法中使用消费者监视器,而在消费方法中使用生产者的监视器(本方唤醒对方)

![image](http://clsaajavaimgbed-10042610.cos.myqcloud.com/%E6%97%A7%E9%94%81%E5%92%8C%E6%96%B0%E9%94%81%E7%9A%84%E5%8C%BA%E5%88%AB.JPG)


### 代码示例-多生产多消费解决效率问题


```

class Resource
{
    private String name;
    private int count = 1;

    //定义一个锁对象。
    private final Lock lock = new ReentrantLock();
    //获取锁上的Condition对象。为了解决本方唤醒对方的问题。可以一个锁创建两个监视器对象。

    private Condition produce = lock.newCondition();//负责生产。
    private Condition consume = lock.newCondition();//负责消费。

    //定义标记。
    private boolean flag = false;

    //1,提供设置的方法。
    public  void set(String name)//
    {
        //获取锁。
        lock.lock();
        try{

            while(flag)
                try{produce.await();}catch(InterruptedException e){}// t1等  t2等
            this.name = name + count;//商品1  商品2  商品3
            count++;//2 3  4
            System.out.println(Thread.currentThread().getName()+"......生产者...."+this.name);//生产 商品1  生产商品2  生产商品3

            //将标记改为true。
            flag = true;
            //执行的消费者的唤醒。唤醒一个消费者就哦了。
            consume.signal();
        }finally{

            lock.unlock();//一定要执行。
        }
    }
    public  void out()//
    {

        lock.lock();
        try{
            while(!flag)
                try{consume.await();}catch(InterruptedException e){}//t3等  //t4等
            System.out.println(Thread.currentThread().getName()+"....消费者...."+this.name);//消费 商品1
            //将标记该为false。
            flag = false;
            //
            produce.signal();
        }
        finally{
            lock.unlock();//这句话一定要执行
        }
    }
}

//2,描述生产者。
class Producer implements Runnable
{
    private Resource r ;
    // 生产者一初始化就要有资源，需要将资源传递到构造函数中。
    Producer(Resource r)
    {
        this.r = r;
    }
    public void run()
    {
        while(true)
        {
            r.set("面包");
        }
    }
}

//3,描述消费者。
class Consumer implements Runnable
{
    private Resource r ;
    // 消费者一初始化就要有资源，需要将资源传递到构造函数中。
    Consumer(Resource r)
    {
        this.r = r;
    }
    public void run()
    {
        while(true)
        {
            r.out();
        }
    }
}




class ThreadDemo11
{
    public static void main(String[] args)
    {
        //1,创建资源对象。
        Resource r = new Resource();

        //2,创建线程任务。
        Producer pro = new Producer(r);
        Consumer con = new Consumer(r);

        //3,创建线程。
        Thread t1 = new Thread(pro);
        Thread t2 = new Thread(pro);
        Thread t3 = new Thread(con);
        Thread t4 = new Thread(con);

        t1.start();
        t2.start();
        t3.start();
        t4.start();
    }
}

```

## 生产多个商品问题

![image](http://clsaajavaimgbed-10042610.cos.myqcloud.com/%E5%A4%9A%E7%94%9F%E4%BA%A7%E5%A4%9A%E6%B6%88%E8%B4%B9%E5%AF%B9%E5%AE%B9%E5%99%A8%E6%93%8D%E4%BD%9C.JPG)


```
class BoundedBuffer {
    final Lock lock = new ReentrantLock();//锁
    final Condition notFull  = lock.newCondition(); //生产
    final Condition notEmpty = lock.newCondition(); //消费

    final Object[] items = new Object[100];//存储商品的容器。
    int putptr/*生产者使用的角标*/, takeptr/*消费者使用的角标*/, count/*计数器*/;

    /*生产者使用的方法，往数组中存储商品*/
    public void put(Object x) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) //判断计数器是否已到数组长度。满了。
                notFull.await();//生产就等待。

            items[putptr] = x; //按照角标将商品存储到数组中

            if (++putptr == items.length) //如果存储的角标到了数组的长度，就将角标归零。
                putptr = 0;
            ++count;//计数器自增。
            notEmpty.signal();//唤醒一个消费者
        } finally {
            lock.unlock();
        }
    }

    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) //如果计数器为0，说明没有商品，消费者等待。
                notEmpty.await();
            Object x = items[takeptr]; //从数组中通过消费者角标获取商品。

            if (++takeptr == items.length) //如果消费的角标等于了数组的长度，将角标归零。
                takeptr = 0;
            --count;//计数器自减。
            notFull.signal();//唤醒生产者。
            return x;
        } finally {
            lock.unlock();
        }
    }
}
 
```

