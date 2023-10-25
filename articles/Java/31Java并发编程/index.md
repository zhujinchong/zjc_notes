## 什么JUC

java并发编程是对java.util.concurrent下面的工具类的使用。

比如：

Thread只是一个普通的线程类，Runnable没有返回值，效率相对Callable使用率较低。而Callable正是java.util.concurrent下面的类。等等还有很多。

## Lock锁

### synchronized用法

```
public class Test {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();
        for (int i = 1; i <= 20; i++) {
            final int temp = i;
            new Thread(() -> {
                ticket.sale();
            }, "Thread-" + temp).start();
        }

    }
}

class Ticket {
    private int number = 10;
    public synchronized void sale() {
        if (number > 0) {
            number -= 1;
            System.out.println(Thread.currentThread().getName() + ": " + number);
        }
    }
}
```

### Lock用法

> 参数可以传true/false：
>
> true默认：非公平锁：可以插队
>
> false：公平锁：先来后到

```
class Ticket {
    private int number = 10;
    Lock lock = new ReentrantLock();
    public void sale() {
        lock.lock();
        try {
            if (number > 0) {
                number -= 1;
                System.out.println(Thread.currentThread().getName() + ": " + number);
            }
        } catch (Exception e) {

        } finally {
            lock.unlock();
        }
    }
}
```

### synchronized vs Lock

区别：

1. synchronized是内置关键字，Lock是内置java类
2. synchronized无法获取锁的状态，Lock可以判断是否获取到了锁
3. synchronized自动释放锁，Lock需要手动释放锁
4. synchronized会阻塞其他线程，Lock锁不一定会等待下去
5. synchronized是非公平的，Lock可以设置是否公平，默认非公平锁。
6. synchronized适合少量代码同步问题，Lock适合锁大量的同步代码

## 生产者消费者

生产者消费者问题是线程之间通信问题：

AB操作同一个变量, A+1, B-1

### synchronized版

```
public class Test {
    public static void main(String[] args) throws InterruptedException {
        Ticket ticket = new Ticket();
        new Thread(() -> { for (int i = 1; i <= 10; i++) {
            try {
                ticket.increment();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }},"A").start();

        new Thread(() -> { for (int i = 1; i <= 10; i++) {
            try {
                ticket.decrement();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }},"B").start();

    }
}

class Ticket {
    private int number = 0;

    public synchronized void increment() throws InterruptedException {
        while (number != 0) {  // 不能用if, 会导致虚假唤醒
            this.wait(); // 等待
        }
        number++;
        System.out.println(Thread.currentThread().getName() + ": " + number);
        this.notifyAll(); // 通知
    }

    public synchronized void decrement() throws InterruptedException {
        while (number == 0) {
            this.wait(); // 等待
        }
        number--;
        System.out.println(Thread.currentThread().getName() + ": " + number);
        this.notifyAll(); // 通知
    }
}
```

### Lock版

```
class Ticket {
    private int number = 0;
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    public void increment() {
        lock.lock();
        try {
            // 业务代码
            while (number != 0) {
                condition.await(); // 等待
            }
            number++;
            System.out.println(Thread.currentThread().getName() + ": " + number);
            condition.signalAll(); // 通知
        } catch (Exception e) {

        } finally {
            lock.unlock();
        }
    }

    public void decrement() {
        lock.lock();
        try {
            while (number == 0) {
                condition.await(); // 等待
            }
            number--;
            System.out.println(Thread.currentThread().getName() + ": " + number);
            condition.signalAll(); // 通知
        } catch (Exception e) {

        } finally {
            lock.unlock();
        }
    }
}
```

### Condition精准通知和唤醒

定义多个condition，实现精准通知

```
// 1...
while (xx != 1) {
	condition.await(); // 等待
}
xx = 2
condition2.signal(); // 通知2


// 2...
while (xx != 2){
	condition2.await();
}
xx = 1
condition.signal(); // 通知1
```



## synchronized的8锁现象

8锁现象是锁出现的几种情况，本节虽然没有全部列举，但以下代码能让你了解清楚锁。

锁一般加到方法上：普通方法、加锁的普通方法、加锁的静态方法

1. synchronized加锁的对象是方法的调用者。因此，加锁的普通方法 锁在实例对象；加锁的静态方法 锁在class模板。
2. 普通方法不会受限于上面两种锁，互不干扰。
3. 实例对象锁 和 Class锁 两者互不烦扰。

基本代码如下：

```
class Phone {
    public synchronized void sendSms() {
        TimeUnit.SECONDS.sleep(4); // 这里需要捕获异常（代码略）
        System.out.println("发短信");
    }

    public synchronized void call() {
        TimeUnit.SECONDS.sleep(1); // 这里需要捕获异常（代码略）
        System.out.println("打电话");
    }

    public void hello() {
        System.out.println("hello");
    }
    
    public synchronized static void hello2() {
    	System.out.println("static hello");
    }
}
```

当调用

```
Phone phone = new Phone();
new Thread(() -> phone.sendSms()).start();
new Thread(() -> phone.call()).start();
new Thread(() -> phone.hello()).start();
```

输出顺序：hello 发短信 打电话

原因：

1. synchronized加锁的对象是他的调用者，在这里就是实例对象。所以发短信 先于 打电话
2. hello()方法没有锁，不受锁限制。所以会先执行。

当调用

```
Phone phone1 = new Phone();
Phone phone2 = new Phone();
new Thread(() -> phone1.sendSms()).start();
new Thread(() -> phone2.call()).start();
```

输出顺序：打电话 发短信

原因：两个实例对象是两把锁，两者不会干扰。所以 打电话 先于 发短信

当调用

```
Phone phone = new Phone();
new Thread(() -> phone.sendSms()).start();
new Thread(() -> Phone.hello2()).start();
```

输出顺序：hello2 发短信

原因：Class锁 和 实例对象锁 互不干扰。





## 集合类不安全

并发下 ArrayList 不安全，可能会报错`java.util.ConcurrentModificationException`

```
List<Double> list = new ArrayList<>();
for (int i = 0; i < 100; i++) {
    new Thread(() -> {
        list.add(Math.random());
        System.out.println(list);
    }, String.valueOf(i)).start();
}
```

解决方案：

```
1. 使用线程安全的集合类，如Vector，底层是synchronized
List<Double> list = new Vector<>();

2. 将集合变成线程安全的
List<Double> list = Collections.synchronizedList(new ArrayList<>());

3. CopyOnWrite 来自 java.util.concurrent 包，底层是Lock锁
// 类似的有CopyOnWriteArraySet ConcurrentHashMap
List<Double> list = new CopyOnWriteArrayList<>();
```

## 多线程常用辅助类

### CoutDownLatch

减法计数器：当计数器复位时，才能被`await()`，再继续执行

```
CountDownLatch countDownLatch = new CountDownLatch(3);
for (int i = 1; i <= 10; i++) {
    new Thread(() -> {
        countDownLatch.countDown(); // 计数器 -1，执行完 +1
        System.out.println(Thread.currentThread().getName());
    }, "thread-" + String.valueOf(i)).start();
}
countDownLatch.await(); // 等待计数器复位，然后再向下执行
System.out.println("成功");
```

### CyclicBarrier

加法计数器：当线程数量达到时，再继续执行。

```
CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> System.out.println("召唤神龙成功"));
for (int i = 1; i <= 7; i++) {
    new Thread(() -> {
        System.out.println(Thread.currentThread().getName());
        try {
            System.out.println("等待...");
            cyclicBarrier.await();
            System.out.println("成功");
        } catch (Exception e) {
            System.out.println("失败");
        }
    }, "thread-" + String.valueOf(i)).start();
}
```

### Semaphore

Semaphore的意思是信号量

```
Semaphore semaphore = new Semaphore(3);
for (int i = 1; i <= 7; i++) {
    new Thread(() -> {
        try {
            semaphore.acquire();  // 得到信号量
            System.out.println(Thread.currentThread().getName() + "得到信号量");
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            semaphore.release(); // 释放信号量
            System.out.println(Thread.currentThread().getName() + "释放信号量");

        }
        System.out.println(Thread.currentThread().getName());
    }, "thread-" + String.valueOf(i)).start();
}
```

### ReadWriteLock

ReadWriteLock是一个接口，只有一个实现类ReentrantReadWriteLock。

作用：多个线程可以同时读，但在写时会加锁。

```
public class Test {
    public static void main(String[] args) {
        MyCacheLock myCache = new MyCacheLock();
        for (int i = 1; i <= 5; i++) {
            final int temp = i;
            new Thread(() -> {
                myCache.put("Thread-" + temp, temp);
            }, "Thread-" + temp).start();
        }
        for (int i = 1; i <= 5; i++) {
            final int temp = i;
            new Thread(() -> {
                myCache.get("Thread-" + temp);
            }, "Thread-" + temp).start();
        }

    }
}

class MyCacheLock {
    private volatile Map<String, Object> map = new HashMap<>();
    private ReadWriteLock lock = new ReentrantReadWriteLock();

    // 写的时候只有一个线程
    public void put(String key, Object val) {
        lock.writeLock().lock();
        System.out.println(Thread.currentThread().getName() + "写入：" + key);
        map.put(key, val);
        System.out.println(Thread.currentThread().getName() + "写入OK");
        lock.writeLock().unlock();
    }

    // 读的时候可以多个线程，读为什么还有加锁？为了防止脏读
    public void get(String key) {
        lock.readLock().lock();
        System.out.println(Thread.currentThread().getName() + "读取：" + key);
        Object o = map.get(key);
        System.out.println(Thread.currentThread().getName() + "读取OK");
        lock.readLock().unlock();
    }
}
```

## volatile

volatile是java虚拟机提供的轻量级同步机制，特点：

可见性：变量的修改对所有线程可见

非原子性：虽然所有线程可见，但是并不能保证操作的原子性

禁止指令重排：程序编译时，编码顺序不会被重排


### 可见性

子线程拿到主线程的变量的副本，当变量在主线程改变时，子线程并不知道。

```
public class Test {
    public static int number = 0;
    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            while (number == 0) {
                
            }
        }).start();
        TimeUnit.SECONDS.sleep(1);
        number = 1;
        System.out.println("end");
    }
}
```

解决办法：加关键字volatile

```
public volatile static int number = 0;
```

### 非原子性

volatile关键字，并不能保证下面代码结果正确

```
public class Test {
    public volatile static int number = 0;
    public static void add() {
        number++;
    }
    public static void main(String[] args) throws InterruptedException {
        for (int i = 1; i <= 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 10000; j++) {
                    add();
                }
            }).start();
        }
        while (Thread.activeCount() > 2) {
            Thread.yield();
        }
        System.out.println(number);
    }
}
```

怎么办呢？

第一种方法：加synchronized关键字

```
public synchronized static void add() {
    number++;
}
```

第二种方法：使用java.util.concurrent.atomic下面的类（底层原理是CAS）

```
public static AtomicInteger number = new AtomicInteger();
public static void add() {
    number.getAndIncrement(); // +1
}
```

## Callable

Callable接口类似于Runnable，都是用于线程的可执行类。但是Runnable没有返回结果，也不能抛出异常。

Callable接口如何和Thread类联系上？

* 通过Runnable接口的实现类`FutureTask<T>( Callable )`

线程用法如下

```
new Thread(new Runnable() {
    @Override
    public void run() {
		// 
    }
}).start();
```

FutureTask是Runnable接口的实现类，且FutureTask 有参函数 `FutureTask( Callable )`

所以，线程可以这么用：

```
FutureTask<String> stringFutureTask = new FutureTask<>(() -> "haha");
new Thread(stringFutureTask).start();
String s = stringFutureTask.get();
```

## 异步回调

> 这里主要学习一个类

没有返回值的异步回调

```
// 发起一个异步请求
CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
    System.out.println("runAsync");;
});
System.out.println("111");
// 异步请求的结果
completableFuture.get();
```

有返回值的异步回调

```
// 发起一个异步请求
CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
    System.out.println("hi");
    return "haha";
});
System.out.println("111");
// 异步请求的结果
completableFuture.whenComplete((t, u) -> {
    System.out.println(t); // t是成功的结果
    System.out.println(u); // u是异常信息
});
```

## 彻底玩转单例模式

饿汉式：假设类中有许多属性，容易占内存。

```
class Hungry {
    private Hungry() {
    }
    private final static Hungry HUNGRY = new Hungry();
    public static Hungry getInstance() {
        return HUNGRY;
    }
}
```

懒汉式：并发下有问题

```
class Lazy {
    private Lazy() {
    }
    private static Lazy LAZY;
    public static Lazy getInstance() {
        if (LAZY == null) {
            LAZY = new Lazy();
        }
        return LAZY;
    }
}
```

DCL懒汉式：双重锁，并发下没有问题

```
class Lazy {
    private Lazy() {
    }
    private volatile static Lazy LAZY;
    public static Lazy getInstance() {
        if (LAZY == null) {
            synchronized (Lazy.class) {
                if (LAZY == null) {
                    LAZY = new Lazy(); // 不是一个原子操作，所以需要在属性前加 volatile
                    /**
                     * 为什么不是原子操作，底层会被编译为：
                     * 1. 分配内存空间
                     * 2. 执行构造方法，初始化对象
                     * 3. 把这个对象指向这个空间
                     * 正常是123，如果操作 132 多线程是容易出错。因此要禁止指令重拍，加volatile关键字
                     */
                }
            }
        }
        return LAZY;
    }
}
```

但是单例模式不安全（反射可以破解，但是反射不能破解枚举类型）

## CAS

> java没办法操作内存，但是java可以调用 c++, 通过c++操作内存。即native方法

CAS：compare and set，比较当前工作内存中的值和主内存中的值，如果是期望值，则操作！如果不是期望值，则一致循环（自旋锁）！

缺点：

1. 循环会耗时
2. 一次性只能保证一个共享变量的原子性
3. ABA问题：线程1将A->B->A，但是线程2并不知道A改变过！

解决ABA问题：（原子操作+版本号/时间戳）

* 可以去搜`AtomicStampedReference`类的原理和使用。

AtomicInteger可以原子操作，就是使用了CAS，看下面代码：

```
// CAS操作如下 
AtomicInteger atomicInteger = new AtomicInteger();
atomicInteger.getAndIncrement(); // +1
System.out.println(atomicInteger.get());

// getAndIncrement() 的源代码如下:
// 这也是比较并交换的源码：
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
    	v = getIntVolatile(o, offset);
    } while (!weakCompareAndSetInt(o, offset, v, v + delta));
    return v;
}
```

## 各种锁的理解

### 公平锁、非公平锁

公平锁：非常公平，不可以插队！

非公平锁：不公平，可以插队！（像超市小商品优先结算）

### 可重入锁

又叫递归锁（嵌套锁）

以下是synchronized版，也有lock版

```
public class Test {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(() -> phone.sms(), "A").start();
        new Thread(() -> phone.sms(), "B").start();
    }
}

class Phone {
    public synchronized void sms() {
        System.out.println(Thread.currentThread().getName() + ": sms");
        call();
    }
    public synchronized void call() {
        System.out.println(Thread.currentThread().getName() + ": call");
    }
}
```

### 自旋锁

创建一个锁（里面用到了自旋锁，自旋锁是一个循环的方法）

```
class SpinlockDemo {
    // 默认值是null
    AtomicReference<Thread> atomic = new AtomicReference<>();

    // 加锁
    public void myLock() {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + "-> mylock");
        // 自旋锁
        while (!atomic.compareAndSet(null, thread)) {
        }
    }

    // 解锁
    public void myUnLock() {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + "-> myUnLock");
        atomic.compareAndSet(thread, null);
    }
}
```

测试：

```
public class Test {
    public static void main(String[] args) throws InterruptedException {
        SpinlockDemo lock = new SpinlockDemo();
        new Thread(() -> {
            lock.myLock();
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.myUnLock();
            }
        }, "T1").start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(() -> {
            lock.myLock();
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.myUnLock();
            }
        }, "T2").start();
    }
}
```



### 死锁排查

死锁代码

```
public class Test {
    public static void main(String[] args) {
        new Thread(new MyThread("A", "B"), "T1").start();
        new Thread(new MyThread("B", "A"), "T2").start();
    }
}

class MyThread implements Runnable {
    private String lockA;
    private String lockB;

    public MyThread(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }

    @SneakyThrows
    @Override
    public void run() {
        synchronized (lockA) {
            System.out.println(Thread.currentThread().getName() + lockA);
            TimeUnit.SECONDS.sleep(2);
            synchronized (lockB) {
                System.out.println(Thread.currentThread().getName() + lockB);
            }
        }
    }
}
```

死锁解决：

1. 打开cmd窗口，输入jps -l，查看代码进程号
2. 再输入`jstack 进程号`查看堆栈信息，发现有死锁。

