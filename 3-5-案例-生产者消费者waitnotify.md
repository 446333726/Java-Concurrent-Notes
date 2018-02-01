---
layout:

title: 3-4-案例-死锁

date: 2015-10-11

updated: 2015-10-11

tags:
- Java多线程
- Java多线程基础
- Java多线程案例

categories: Java多线程基础

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---


# 　生产者＆消费者问题

* 多线程中最为常见的应用案例
* 生产者消费者问题
* 生产和消费同时执行，需要多线程
* 但是执行的任务却不相同，处理的资源却是相同的；线程间通信．

 * 描述一下资源
 * 描述一下生产者，因为其具备自己的任务
 * 描述一下消费者，因为其具备自己的任务

## 代码示例-1

### 代码


```

/*描述资源*/
/*
* 属性：商品名称和编号
* 行为：对名称赋值，获取商品
* */
class Resource{
    private String name;
    private int count = 1;



    public String getName() {
        return name;
    }

    public  void setName(String name) {
        this.name = name + count;
        count++;
        System.out.println(Thread.currentThread().getName() + " ...生产者..." + this.name);
    }

    public  void out()
    {
        System.out.println(Thread.currentThread().getName() + " ...消费者..." + this.name);
    }
}

/*描述生产者*/
class Producer implements Runnable{

    private Resource resource;

    public Producer(Resource resource) {
        this.resource = resource;
    }

    public void run() {
        while (true)
        {
            resource.setName("面包");
        }
    }
}
/*描述消费者*/
class Consumer implements Runnable{
    private Resource resource;

    public Consumer(Resource resource) {
        this.resource = resource;
    }

    public void run() {
        while (true)
        {
            resource.out();
        }
    }
}
public class ProduceConsum {
    public static void main(String[] args) {
        Resource resource = new Resource();

        Producer producer = new Producer(resource);
        Consumer consumer = new Consumer(resource);

        Thread t1  = new Thread(producer);
        Thread t2 = new Thread(consumer);
        t1.start();
        t2.start();
    }
}

```

### 结果片段
```
Thread-0 ...生产者...面包173867
Thread-0 ...生产者...面包173868
Thread-0 ...生产者...面包173869
Thread-0 ...生产者...面包173870
Thread-1 ...消费者...面包173869
Thread-1 ...消费者...面包173871
Thread-1 ...消费者...面包173871
```

### 分析 

>数据错误需加入同步代码块
* 未加同步前问题1：运行结果错误：已经被生产很早期的商品，才被消费到
* 问题已解决 不会再消费到之前很早期的商品。但出现了问题2
* 问题2：出现了连续生产却没有消费，同时对同一个商品进行多次消费
* 希望生产一个商品就被消费掉，然后再去生产下一个

## 代码示例-加同步


```
/*描述资源*/
/*
* 属性：商品名称和编号
* 行为：对名称赋值，获取商品
* */
class Resource{
    private String name;
    private int count = 1;



    public String getName() {
        return name;
    }

    public synchronized void setName(String name) {
        this.name = name + count;
        count++;
        System.out.println(Thread.currentThread().getName() + " ...生产者..." + this.name);
    }

    public synchronized void out()
    {
        System.out.println(Thread.currentThread().getName() + " ...消费者..." + this.name);
    }
}

/*描述生产者*/
class Producer implements Runnable{

    private Resource resource;

    public Producer(Resource resource) {
        this.resource = resource;
    }

    public void run() {
        while (true)
        {
            resource.setName("面包");
        }
    }
}
/*描述消费者*/
class Consumer implements Runnable{
    private Resource resource;

    public Consumer(Resource resource) {
        this.resource = resource;
    }

    public void run() {
        while (true)
        {
            resource.out();
        }
    }
}
public class ProduceConsum {
    public static void main(String[] args) {
        Resource resource = new Resource();

        Producer producer = new Producer(resource);
        Consumer consumer = new Consumer(resource);

        Thread t1  = new Thread(producer);
        Thread t2 = new Thread(consumer);
        t1.start();
        t2.start();
    }
}

```


### 结果片段

```
Thread-0 ...生产者...面包51722
Thread-0 ...生产者...面包51723
Thread-0 ...生产者...面包51724
Thread-0 ...生产者...面包51725
Thread-0 ...生产者...面包51726
Thread-0 ...生产者...面包51727
Thread-0 ...生产者...面包51728
Thread-0 ...生产者...面包51729
Thread-0 ...生产者...面包51730
Thread-1 ...消费者...面包51730
Thread-1 ...消费者...面包51730
Thread-1 ...消费者...面包51730
Thread-1 ...消费者...面包51730
Thread-1 ...消费者...面包51730
Thread-1 ...消费者...面包51730
```

### 问题分析

* 问题2：出现了连续生产却没有消费，同时对同一个商品进行多次消费
* 希望生产一个商品就被消费掉，然后再去生产下一个
* 生产者生么时候应该生产呢？消费者生么时候应该消费呢？
* 当盘子中没有面包时，就生产，如果有了面包就不要生产
* 当盘子中已有面包时，就消费，如果没了面包就不消费

## 代码示例-wait/notify

```

/*描述资源*/
/*
* 属性：商品名称和编号
* 行为：对名称赋值，获取商品
* */
class Resource{
    private String name;
    private int count = 1;
    private boolean flag = false;


    public String getName() {
        return name;
    }

    public synchronized void setName(String name) {
        if (flag)
        {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        this.name = name + count;
        count++;
        System.out.println(Thread.currentThread().getName() + " ...生产者..." + this.name);
        flag = true;
        notify();
    }

    public synchronized void out()  {
        if (!flag)
        {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println(Thread.currentThread().getName() + " ...消费者..." + this.name);
        flag = false;
        notify();
    }
}

/*描述生产者*/
class Producer implements Runnable{

    private Resource resource;

    public Producer(Resource resource) {
        this.resource = resource;
    }

    public void run() {
        while (true)
        {
            resource.setName("面包");
        }
    }
}

/*描述消费者*/
class Consumer implements Runnable{
    private Resource resource;

    public Consumer(Resource resource) {
        this.resource = resource;
    }

    public void run() {
        while (true)
        {
            resource.out();
        }
    }
}

public class ProduceConsum {
    public static void main(String[] args) {
        Resource resource = new Resource();

        Producer producer = new Producer(resource);
        Consumer consumer = new Consumer(resource);

        Thread t1  = new Thread(producer);
        Thread t2 = new Thread(consumer);
        t1.start();
        t2.start();
    }
}

```

### 结果片段


```
Thread-0 ...生产者...面包90079
Thread-1 ...消费者...面包90079
Thread-0 ...生产者...面包90080
Thread-1 ...消费者...面包90080
Thread-0 ...生产者...面包90081
Thread-1 ...消费者...面包90081
Thread-0 ...生产者...面包90082
Thread-1 ...消费者...面包90082
Thread-0 ...生产者...面包90083
Thread-1 ...消费者...面包90083
Thread-0 ...生产者...面包90084
Thread-1 ...消费者...面包90084
Thread-0 ...生产者...面包90085
Thread-1 ...消费者...面包90085
Thread-0 ...生产者...面包90086
Thread-1 ...消费者...面包90086
Thread-0 ...生产者...面包90087
Thread-1 ...消费者...面包90087
Thread-0 ...生产者...面包90088
Thread-1 ...消费者...面包90088
Thread-0 ...生产者...面包90089
Thread-1 ...消费者...面包90089
Thread-0 ...生产者...面包90090
```

### 分析

*  生产者生产了商品后应该告诉消费者来消费。这时的生产者应该处于等待状态
*  消费者消费了商品后，应该告诉生产者，这时候消费者应该等待
*  等待wait:会让线程处于等待，将线程临时存储到线程池中。
*  唤醒notify notifyAll:唤醒在此锁上等待的单个线程。如果所有线程都在此对象上等待，那么就会唤醒任意一个等待的线程

*  记住：这些方法必须使用在同步中，因为必须要标识wait，notify等方法所属的锁，同一个锁上的notify只能唤醒该锁上的被wait的线程

*  为什么这些方法定义在object中？
*  因为这些方法必须标识所属的锁，而锁可以是任意对象，任意对象可以调用的方法必然是Object中的方法。

## wait&notify

>这些方法必须是在同步中使用,因为当前线程必须拥有此对象监视器(必须要标识wait,notify所属的锁),同一个锁上的notify,只能唤醒该锁上的被wait的线程.

```
wait:会让线程处于等待状态,其实就是将线程临时存储到线程池中.
public final void wait() throws InterruptedException
在其他线程调用此对象的 notify() 方法或 notifyAll() 方法前，导致当前线程等待。换句话说，此方法的行为就好像它仅执行 wait(0) 调用一样。 
当前线程必须拥有此对象监视器,该线程发布对此监视器的所有权并等待，直到其他线程通过调用 notify 方法，或 notifyAll 方法通知在此对象的监视器上等待的线程醒来。然后该线程将等到重新获得对监视器的所有权后才能继续执行。 

对于某一个参数的版本，实现中断和虚假唤醒是可能的，而且此方法应始终在循环中使用： 

synchronized (obj) {
    while (<condition does not hold>)
    obj.wait();
    ... // Perform action appropriate to condition
     }
 此方法只应由作为此对象监视器的所有者的线程来调用。有关线程能够成为监视器所有者的方法的描述，请参阅 notify 方法。 
 
notify 会唤醒任意一个等待的线程
public final void notify()唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。线程通过调用其中一个 wait 方法，在对象的监视器上等待。 
直到当前线程放弃此对象上的锁定，才能继续执行被唤醒的线程。被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争；例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。 

此方法只应由作为此对象监视器的所有者的线程来调用。通过以下三种方法之一，线程可以成为此对象监视器的所有者： 

通过执行此对象的同步实例方法。 
通过执行在此对象上进行同步的 synchronized 语句的正文。 
对于 Class 类型的对象，可以通过执行该类的同步静态方法。 
一次只能有一个线程拥有对象的监视器。 

notifyAll 会唤醒线程池中所有等待的线程.
public final void notifyAll()唤醒在此对象监视器上等待的所有线程。线程通过调用其中一个 wait 方法，在对象的监视器上等待。 
直到当前线程放弃此对象上的锁定，才能继续执行被唤醒的线程。被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争；例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。 

此方法只应由作为此对象监视器的所有者的线程来调用。有关线程能够成为监视器所有者的方法的描述，请参阅 notify 方法。 

```

## 代码示例-多生产多消费的问题

###　代码

>其他不变仅仅增加两个线程

```
public class ProduceConsum {
    public static void main(String[] args) {
        Resource resource = new Resource();

        Producer producer = new Producer(resource);
        Consumer consumer = new Consumer(resource);

        Thread t1  = new Thread(producer);
        Thread t2  = new Thread(producer);
        Thread t3  = new Thread(consumer);
        Thread t4 = new Thread(consumer);
        t1.start();
        t2.start();
        t3.start();
        t4.start();
    }
}
```



###　结果片段


```
Thread-1 ...生产者...面包129360
Thread-3 ...消费者...面包129360
Thread-2 ...消费者...面包129360
Thread-1 ...生产者...面包129361
Thread-3 ...消费者...面包129361
Thread-2 ...消费者...面包129361
Thread-1 ...生产者...面包129362
Thread-3 ...消费者...面包129362
Thread-2 ...消费者...面包129362
Thread-1 ...生产者...面包129363
Thread-3 ...消费者...面包129363
Thread-2 ...消费者...面包129363
Thread-1 ...生产者...面包129364
Thread-3 ...消费者...面包129364
```

###　分析

>生产了商品没有被消费,同一个商品被消费多次.

>出现原因:假设t1t2为生产者 t3t4为消费者 t1生产->t1等待->t2等待->t3消费->t3等待->t4等->t1生产->t2被唤醒(此时从wait开始不再判断标记) t1生产的第二个商品未被消费.

>被唤醒的线程未判断标记,造成了问题的产生
>解决:让被唤醒的线程必须判断标记.使用while,把if转为while判断

>while判断后死锁,

>原因:生产方唤醒了线程池中生产方
>解决:希望本方要唤醒对方.(把notify换位notifyALl)

```
class Resource
{
    private String name;
    private int count = 1;

    //定义标记。
    private boolean flag = false;

    //1,提供设置的方法。
    public synchronized void set(String name)//
    {

        if(flag)
            try{this.wait();}catch(InterruptedException e){}// t1等  t2等
        //给成员变量赋值并加上编号。
        this.name = name + count;//商品1  商品2  商品3
        //编号自增。
        count++;//2 3  4
        //打印生产了哪个商品。
        System.out.println(Thread.currentThread().getName()+"......生产者...."+this.name);//生产 商品1  生产商品2  生产商品3

        //将标记改为true。
        flag = true;
        //唤醒消费者。
        this.notify();
    }
    public synchronized void out()//
    {
        if(!flag)
            try{this.wait();}catch(InterruptedException e){}//t3等  //t4等
        System.out.println(Thread.currentThread().getName()+"....消费者...."+this.name);//消费 商品1
        //将标记该为false。
        flag = false;
        //唤醒生产者。
        this.notify();
    }
}
```

## 代码示例-多生产多消费解决

### 代码


```

class Resource
{
    private String name;
    private int count = 1;

    //定义标记。
    private boolean flag = false;

    //1,提供设置的方法。
    public synchronized void set(String name)//
    {

        while(flag)
            try{this.wait();}catch(InterruptedException e){}// t1等  t2等
        //给成员变量赋值并加上编号。
        this.name = name + count;//商品1  商品2  商品3
        //编号自增。
        count++;//2 3  4
        //打印生产了哪个商品。
        System.out.println(Thread.currentThread().getName()+"......生产者...."+this.name);//生产 商品1  生产商品2  生产商品3

        //将标记改为true。
        flag = true;
        //唤醒消费者。
        this.notifyAll();
    }
    public synchronized void out()//
    {
        while(!flag)
            try{this.wait();}catch(InterruptedException e){}//t3等  //t4等
        System.out.println(Thread.currentThread().getName()+"....消费者...."+this.name);//消费 商品1
        //将标记该为false。
        flag = false;
        //唤醒生产者。
        this.notifyAll();
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




class ThreadDemo10
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
### 分析

>效率变低,因为唤醒了所有的锁,然而只有一个消费者可以消费.

