# Java-Concurrent

## 线程

### 线程和进程

**进程**

进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。

**线程**

线程与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程共享进程的**堆**和**方法区**资源，但每个线程有自己的**程序计数器**、**虚拟机栈**和**本地方法栈**，所以系统在产生一个线程，或是在各个线程之间做切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。





### 线程的创建方式

| **实现方式**                         | **核心实现**                                                 | **优点**                                                     | **缺点**                                                     | **适用场景**                                                 |
| :----------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **1. 继承Thread类**                  | 自定义类继承 `Thread`，重写 `run()` 方法。                   | • 编写简单，代码量少。 • 可直接使用 `this` 访问当前线程。    | • 单继承限制，无法再继承其他父类。 • 线程任务与线程对象耦合度高。 | 简单、单一线程任务，且无需继承其他类时。                     |
| **2. 实现Runnable接口**              | 实现 `Runnable` 接口，重写 `run()` 方法。                    | • 避免了单继承限制，可继承其他类。 • 易于实现多个线程共享同一任务对象（数据）。 • 模型清晰，分离了线程和任务。 | • 编程相对复杂。 • 访问当前线程需调用 `Thread.currentThread()`。 | 多个线程需要处理同一份资源（共享数据）或需要继承其他类的场景。 |
| **3. 实现Callable接口 + FutureTask** | 实现 `Callable` 接口，重写 `call()` 方法（有返回值，可抛异常） | • 任务执行后有返回值。 • 可以抛出异常并处理。 • 支持泛型。 • 同样避免了单继承限制。 | • 编程更复杂。 • 获取返回值时，`get()` 方法会阻塞调用线程。  | 需要获取线程执行结果或处理非受检异常的任务。                 |
| **4. 使用线程池（Executor框架）**    | 实现 `Runnable` 或 `Callable` 接口定义任务。                 | • 重用线程，减少创建/销毁开销，性能高。 • 响应快，降低任务等待时间。 • 有效控制并发线程数量，避免资源耗尽。 • 便于统一管理和监控。 | • 增加程序复杂度，配置和调优有门槛。 • 配置不当（如线程池大小）可能导致死锁、资源耗尽等问题，排查较难。 | 高并发、任务数量众多或需要频繁执行短期异步任务的系统。       |



### 生命周期和状态

Java 线程在运行的生命周期中的指定时刻只可能处于下面 6 种不同状态的其中一个状态：

- NEW: 初始状态，线程被创建出来但没有被调用 `start()` 。
- RUNNABLE: 运行状态，线程被调用了 `start()`等待运行的状态。
- BLOCKED：阻塞状态，需要等待锁释放。
- WAITING：等待状态，表示该线程需要等待其他线程做出一些特定动作（通知或中断）。
- TIME_WAITING：超时等待状态，可以在指定的时间后自行返回而不是像 WAITING 那样一直等待。
- TERMINATED：终止状态，表示该线程已经运行完毕。

<div align="center"> <img src="https://oss.javaguide.cn/github/javaguide/java/concurrent/640.png" width="75%"/> </div><br>



### 线程上下文切换

线程在执行过程中会有自己的运行条件和状态（也称上下文），比如上文所说到过的程序计数器，栈信息等。当出现如下情况的时候，线程会从占用 CPU 状态中退出。

- 主动让出 CPU，比如调用了 `sleep()`, `wait()` 等。
- 时间片用完，因为操作系统要防止一个线程或者进程长时间占用 CPU 导致其他线程或者进程饿死。
- 调用了阻塞类型的系统中断，比如请求 IO，线程被阻塞。
- 被终止或结束运行

这其中前三种都会发生线程切换，线程切换意味着需要保存当前线程的上下文，留待线程下次占用 CPU 的时候恢复现场。并加载下一个将要占用 CPU 的线程上下文。这就是所谓的 **上下文切换**。

上下文切换是现代操作系统的基本功能，因其每次需要保存信息恢复信息，这将会占用 CPU，内存等系统资源进行处理，也就意味着效率会有一定损耗，如果频繁切换就会造成整体效率低下。





### sleep 和 wait的区别是什么？

| **特性** | `sleep()`                  | `wait()`                           |
| -------- | -------------------------- | ---------------------------------- |
| 所属类   | `Thread` 类（静态方法）    | `Object` 类（实例方法）            |
| 锁释放   | ❌                          | ✅                                  |
| 使用前提 | 任意位置调用               | 必须在同步块内（持有锁）           |
| 唤醒机制 | 超时自动恢复               | 需 `notify()`/`notifyAll()` 或超时 |
| 设计用途 | 暂停线程执行，不涉及锁协作 | 线程间协调，释放锁让其他线程工作   |

- **所属分类的不同**：sleep 是 `Thread` 类的静态方法，可以在任何地方直接通过 `Thread.sleep()` 调用，无需依赖对象实例。wait 是 `Object` 类的实例方法，这意味着必须通过对象实例来调用。
- **锁释放的情况**：`Thread.sleep()` 在调用时，线程会暂停执行指定的时间，但不会释放持有的对象锁。也就是说，在 `sleep` 期间，其他线程无法获得该线程持有的锁。`Object.wait()`：调用该方法时，线程会释放持有的对象锁，进入等待状态，直到其他线程调用相同对象的 `notify()` 或 `notifyAll()` 方法唤醒它
- **使用条件**：sleep 可在任意位置调用，无需事先获取锁。 wait 必须在同步块或同步方法内调用（即线程需持有该对象的锁），否则抛出 `IllegalMonitorStateException`。
- **唤醒机制**：sleep 休眠时间结束后，线程 自动恢复 到就绪状态，等待CPU调度。wait 需要其他线程调用相同对象的 `notify()` 或 `notifyAll()` 方法才能被唤醒。`notify()` 会随机唤醒一个在该对象上等待的线程，而 `notifyAll()` 会唤醒所有在该对象上等待的线程。



### sleep会释放cpu吗？

是的，调用 `Thread.sleep()` 时，线程会释放 CPU，但不会释放持有的锁。

**当线程调用** `sleep()` **后，会主动让出 CPU 时间片**，进入 `TIMED_WAITING` 状态。此时操作系统会触发调度，将 CPU 分配给其他处于就绪状态的线程。这样其他线程（无论是需要同一锁的线程还是不相关线程）便有机会执行。

`sleep()` 不会释放线程已持有的任何锁（如 `synchronized` 同步代码块或方法中获取的锁）。因此，如果有其他线程试图获取同一把锁，它们仍会被阻塞，直到原线程退出同步代码块。



### BLOCKED vs WAITING 状态区别

| 对比维度           | **BLOCKED（阻塞）**                                          | **WAITING（等待）**                                          |
| :----------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **触发方式**       | **被动触发** 线程因锁竞争失败，试图进入 `synchronized` 代码块/方法时，锁被其他线程持有。 | **主动触发** 线程主动调用以下方法（人为让出CPU）： •`Object.wait()` • `Thread.join()` • `LockSupport.park()` |
| **唤醒机制**       | **自动唤醒** 当持有锁的线程释放锁后，阻塞的线程会自动重新竞争锁（无需显式干预）。 | **显式唤醒** 必须由其他线程执行特定操作： • `Object.notify()` / `notifyAll()` • 被 `join()` 的线程执行完毕 • LockSupport.unpark()` |
| **核心区别**       | 等待**获取锁资源**                                           | 等待**其他线程执行特定动作**                                 |
| **是否参与锁竞争** | 是（一旦锁释放，立即参与竞争）                               | 否（唤醒后才会重新参与锁竞争或继续执行）                     |

**BLOCKED** 是线程**被动等待锁释放**（自动恢复）；**WAITING** 是线程**主动等待其他线程通知**（需显式唤醒）。



### 如何理解线程安全和不安全

线程安全和不安全是在多线程环境下对于同一份数据的访问是否能够保证其正确性和一致性的描述。

- 线程安全指的是在多线程环境下，对于同一份数据，不管有多少个线程同时访问，都能保证这份数据的正确性和一致性。
- 线程不安全则表示在多线程环境下，对于同一份数据，多个线程同时访问时可能会导致数据混乱、错误或者丢失。



### 并发与并行的区别

- **并发**：两个及两个以上的作业在同一 **时间段** 内执行。
- **并行**：两个及两个以上的作业在同一 **时刻** 执行。

最关键的点是：是否是 **同时** 执行。



### 同步和异步的区别

- **同步**：发出一个调用之后，在没有得到结果之前， 该调用就不可以返回，一直等待。
- **异步**：调用在发出之后，不用等待返回结果，该调用直接返回。



### 死锁

产生死锁的四个必要条件：

1. **互斥条件**：该资源任意一个时刻只由一个线程占用。
2. **请求与保持条件**：一个线程因请求资源而阻塞时，对已获得的资源保持不放。
3. **不剥夺条件**：线程已获得的资源在未使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
4. **循环等待条件**：若干线程之间形成一种头尾相接的循环等待资源关系。

**如何预防死锁？** 破坏死锁的产生的必要条件即可：

1. **破坏请求与保持条件**：一次性申请所有的资源。
2. **破坏不剥夺条件**：占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源。
3. **破坏循环等待条件**：靠按序申请资源来预防。按某一顺序申请资源，释放资源则反序释放。破坏循环等待条件。



## volatile关键字

**`volatile` 关键字保证变量的可见性**

<div align="center"> <img src="https://oss.javaguide.cn/github/javaguide/java/concurrent/jmm2.png" width="75%"/> </div>

**`volatile` 关键字防止 JVM 的指令重排序**

1. 单线程唤醒下不改变程序运行结果
2. 存在数据依赖关系的不能重排序

如果我们将变量声明为 **`volatile`** ，在对这个变量进行读写操作的时候，会通过插入特定的 **内存屏障** 的方式来禁止指令重排序。

**volatile 写操作的内存屏障插入策略**

**volatile 读操作的内存屏障插入策略**

**单例模式**

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {
        // private 访问修饰符将构造器隐藏，防止外部直接实例化
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
// instance = new Singleton();
// 1.为 instance 分配内存空间
// 2.初始化 instance
// 3.将 instance 指向分配的内存地址
// 如果不加volatile，步骤2和步骤3顺序可能颠倒
```

**`volatile` 关键字能保证变量的可见性，但不能保证对变量的操作是原子性的。**







## 乐观锁和悲观锁

**悲观锁**

悲观锁总是假设最坏的情况，认为共享资源每次被访问的时候就会出现问题(比如共享数据被修改)，所以每次在获取资源操作的时候都会上锁，这样其他线程想拿到这个资源就会阻塞直到锁被上一个持有者释放。也就是说，**共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程**。高并发的场景下，激烈的锁竞争会造成线程阻塞，大量阻塞线程会导致系统的上下文切换，增加系统的性能开销。并且，悲观锁还可能会存在死锁问题，影响代码的正常运行。

**乐观锁**

乐观锁总是假设最好的情况，认为共享资源每次被访问的时候不会出现问题，线程可以不停地执行，无需加锁也无需等待，只是在提交修改的时候去验证对应的资源（也就是数据）是否被其它线程修改了（具体方法可以使用版本号机制或 CAS 算法）。



### 如何实现乐观锁

**版本号机制**

一般是在数据表中加上一个数据版本号 `version` 字段，表示数据被修改的次数。当数据被修改时，`version` 值会加一。当线程 A 要更新数据值时，在读取数据的同时也会读取 `version` 值，在提交更新时，若刚才读取到的 version 值为当前数据库中的 `version` 值相等时才更新，否则重试更新操作，直到更新成功。

**CAS 算法**

CAS 的全称是 **Compare And Swap（比较与交换）** ，用于实现乐观锁，被广泛应用于各大框架中。CAS 的思想很简单，就是用一个预期值和要更新的变量值进行比较，两值相等才会进行更新。

CAS 是一个原子操作，底层依赖于一条 CPU 的原子指令。

> **原子操作** 即最小不可拆分的操作，也就是说操作一旦开始，就不能被打断，直到操作完成。



### CAS 算法存在的问题

**ABA 问题**

如果一个变量 V 初次读取的时候是 A 值，并且在准备赋值的时候检查到它仍然是 A 值，那我们就能说明它的值没有被其他线程修改过了吗？很明显是不能的，因为在这段时间它的值可能被改为其他值，然后又改回 A，那 CAS 操作就会误认为它从来没有被修改过。这个问题被称为 CAS 操作的 "ABA"问题。

ABA 问题的解决思路是在变量前面追加上**版本号或者时间戳**。

**自旋开销**

CAS 经常会用到自旋操作来进行重试，也就是不成功就一直循环执行直到成功。如果长时间不成功，会给 CPU 带来非常大的执行开销。

**只能保证一个共享变量的原子操作**

CAS 操作仅能对单个共享变量有效。当需要操作多个共享变量时，CAS 就显得无能为力。不过，从 JDK 1.5 开始，Java 提供了`AtomicReference`类，这使得我们能够保证引用对象之间的原子性。通过将多个变量封装在一个对象中，我们可以使用`AtomicReference`来执行 CAS 操作。



## synchronized关键字

`synchronized` 关键字的使用方式主要有下面 3 种：

1. 修饰实例方法
2. 修饰静态方法
3. 修饰代码块

### 底层原理

1. `synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。
2. 当执行 `monitorenter` 指令时，线程试图获取锁也就是获取 对象监视器 `monitor` 的持有权。
3. 在执行`monitorenter`时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。
4. 对象锁的拥有者线程才可以执行 `monitorexit` 指令来释放锁。在执行 `monitorexit` 指令后，将锁计数器设为 0，表明锁被释放，其他线程可以尝试获取锁。
5. 如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。



### synchronized的优化（JDK 1.6版本）

在jdk1.5版本（包含）之前，锁的状态只有两种状态：“无锁状态”和“重量级锁状态”，只要有线程访问共享资源对象，则锁直接成为重量级锁，jdk1.6版本后，对synchronized锁进行了优化，新加了“偏向锁”和“轻量级锁”，用来减少上下文的切换以提高性能，所以锁就有了4种状态。

锁的四种状态从低到高依次为：

1. **无锁状态**
2. **偏向锁**
3. **轻量级锁**
4. **重量级锁**

这四种状态会随着锁竞争的激烈程度**单向升级**（锁膨胀），不能降级。具体区别如下：

| 锁状态       | 设计初衷                   | 适用场景                           | 主要开销                                                |
| :----------- | :------------------------- | :--------------------------------- | :------------------------------------------------------ |
| **无锁**     | 没有线程竞争资源           | 单线程访问或无需加锁的代码         | 无                                                      |
| **偏向锁**   | 消除无竞争情况下的同步原语 | 始终只有一个线程反复获取锁         | 仅一次CAS（Compare-And-Swap）记录线程ID                 |
| **轻量级锁** | 将阻塞等待改为自旋等待     | 两个线程交替执行，或锁持有时间很短 | **自旋**消耗CPU（不阻塞，避免线程切换）                 |
| **重量级锁** | 引入操作系统内核态互斥量   | 多线程串行化、锁持有时间长         | 线程**阻塞**和**唤醒**（涉及用户态/内核态切换，开销大） |

**JDK 1.6 为 `synchronized` 引入了“偏向锁 → 轻量级锁 → 重量级锁”的膨胀路径，目的是让锁在不同竞争压力下选择最优策略，尽量减少线程阻塞和系统调用。**



### volatile 和 synchronized 关系

| 对比维度       | **volatile**                | **synchronized**                         | **关系本质**                                     |
| :------------- | :-------------------------- | :--------------------------------------- | :----------------------------------------------- |
| **核心功能**   | 保证**可见性** + **有序性** | 保证**可见性** + **有序性** + **原子性** | `synchronized` 功能更全面，`volatile` 是它的子集 |
| **实现机制**   | 内存屏障指令（无锁）        | 操作系统互斥锁（可能有锁膨胀）           | 机制不同，导致性能差异                           |
| **性能开销**   | **轻量级**（不阻塞线程）    | **重量级**（竞争时阻塞/唤醒）            | `volatile` 性能更优                              |
| **使用范围**   | 只能修饰**变量**            | 修饰**方法** / **代码块**                | `synchronized` 应用更灵活                        |
| **原子性保证** | ❌ 不能保证                  | ✅ 能保证                                 | **关键区分点**                                   |

**核心逻辑关系**

1. 性能层面：`volatile` 是轻量级方案

- `volatile` 不涉及锁竞争 → 无线程阻塞 → 无上下文切换 → **性能更优**
- `synchronized` 涉及锁升级（偏向→轻量→重量）→ 竞争激烈时进入内核态 → **开销更大**

2. 功能层面：`synchronized` 是重量级方案

- `synchronized` = `volatile`（可见性） + **原子性** + **互斥性**
- 当需要**复合操作**（如 `i++`）或**多个变量同时更新**时，`volatile` 无法保证线程安全，必须用 `synchronized`



## AQS

[AQS详解](https://javaguide.cn/java/concurrent/aqs.html)

AQS 的全称为 `AbstractQueuedSynchronizer` ，抽象队列同步器。这个类在 `java.util.concurrent.locks` 包下面。

AQS （`AbstractQueuedSynchronizer` ，抽象队列同步器）是从 JDK1.5 开始提供的 Java 并发核心组件。它提供了一个通用框架，用于实现各种同步器，例如 **可重入锁**（`ReentrantLock`）、**信号量**（`Semaphore`）和 **倒计时器**（`CountDownLatch`）。通过封装底层的线程同步机制，AQS 将复杂的线程管理逻辑隐藏起来，使开发者只需专注于具体的同步逻辑。

### 底层原理

AQS 维护一个 FIFO 的双向链表队列（CLH 变种）。当线程获取锁失败时，AQS 会将该线程封装成节点加入队尾。

> **CLH 锁** （Craig, Landin, and Hagersten locks） 

节点在尝试获取锁时，**会先判断前驱是否是头节点**，如果是，则尝试获取锁；否则（或获取失败）**可选择自旋几次**，然后**阻塞**（`park`）。

当头节点释放锁后，会**唤醒**它的下一个节点，被唤醒的节点再去尝试获取锁。

<div align="center"> <img src="https://oss.javaguide.cn/github/javaguide/java/concurrent/clh-queue-state.png" width="100%"/> </div>

AQS 使用 **int 成员变量 `state` 表示同步状态**，通过内置的 **线程等待队列** 来完成获取资源线程的排队工作。



## ReentrantLock

`ReentrantLock` 实现了 `Lock` 接口，是一个可重入且独占式的锁，和 `synchronized` 关键字类似。不过，`ReentrantLock` 更灵活、更强大，增加了轮询、超时、中断、公平锁和非公平锁等高级功能。

`ReentrantLock` 里面有一个内部类 `Sync`，`Sync` 继承 AQS（`AbstractQueuedSynchronizer`），添加锁和释放锁的大部分操作实际上都是在 `Sync` 中实现的。`Sync` 有公平锁 `FairSync` 和非公平锁 `NonfairSync` 两个子类。



### 公平锁和非公平锁有什么区别

- **公平锁** : 锁被释放之后，先申请的线程先得到锁。性能较差一些，因为公平锁为了保证时间上的绝对顺序，上下文切换更频繁。
- **非公平锁**：锁被释放之后，后申请的线程可能会先获取到锁，是随机或者按照其他优先级排序的。性能更好，但可能会导致某些线程永远无法获取到锁。

- **公平锁执行流程**：获取锁时，先将线程自己添加到等待队列的队尾并休眠，当某线程用完锁之后，会去唤醒等待队列中队首的线程尝试去获取锁，锁的使用顺序也就是队列中的先后顺序，在整个过程中，线程会从运行状态切换到休眠状态，再从休眠状态恢复成运行状态，但线程每次休眠和恢复都需要从用户态转换成内核态，而这个状态的转换是比较慢的，所以公平锁的执行速度会比较慢。
- **非公平锁执行流程**：当线程获取锁时，会先通过 CAS 尝试获取锁，如果获取成功就直接拥有锁，如果获取锁失败才会进入等待队列，等待下次尝试获取锁。这样做的好处是，获取锁不用遵循先到先得的规则，从而避免了线程休眠和恢复的操作，这样就加速了程序的执行效率。



### ReentrantLock 比 synchronized 都是可重入锁

**可重入锁** 也叫递归锁，指的是线程可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果是不可重入锁的话，就会造成死锁。

`synchronized` 是依赖于 JVM 实现的，前面我们也讲到了 虚拟机团队在 JDK1.6 为 `synchronized` 关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。

`ReentrantLock` 是 JDK 层面实现的（也就是 API 层面，需要 `lock()` 和 `unlock()` 方法配合 `try/finally` 语句块来完成）。



### ReentrantLock 比 synchronized 增加的功能

相比`synchronized`，`ReentrantLock`增加了一些高级功能。主要来说主要有三点：

- **等待可中断** : `ReentrantLock`提供了一种能够中断等待锁的线程的机制，通过 `lock.lockInterruptibly()` 来实现这个机制。也就是说当前线程在等待获取锁的过程中，如果其他线程中断当前线程「 `interrupt()` 」，当前线程就会抛出 `InterruptedException` 异常，可以捕捉该异常进行相应处理。
- **可实现公平锁** : `ReentrantLock`可以指定是公平锁还是非公平锁。而`synchronized`只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。`ReentrantLock`默认情况是非公平的，可以通过 `ReentrantLock`类的`ReentrantLock(boolean fair)`构造方法来指定是否是公平的。
- **通知机制更强大**：`ReentrantLock` 通过绑定多个 `Condition` 对象，可以实现分组唤醒和选择性通知。这解决了 `synchronized` 只能随机唤醒或全部唤醒的效率问题，为复杂的线程协作场景提供了强大的支持。
- **支持超时** ：`ReentrantLock` 提供了 `tryLock(timeout)` 的方法，可以指定等待获取锁的最长等待时间，如果超过了等待时间，就会获取锁失败，不会一直等待。



## ReentrantReadWriteLock







## Threadlocal

**`ThreadLocal` 类允许每个线程绑定自己的值**，可以将其形象地比喻为一个“存放数据的盒子”。每个线程都有自己独立的盒子，用于存储私有数据，确保不同线程之间的数据互不干扰。

**变量是放在了当前线程的 `ThreadLocalMap` 中，并不是存在 `ThreadLocal` 上，`ThreadLocal` 可以理解为只是`ThreadLocalMap`的封装，传递了变量值。** `ThrealLocal` 类中可以通过`Thread.currentThread()`获取到当前线程对象后，直接通过`getMap(Thread t)`可以访问到该线程的`ThreadLocalMap`对象。

**通俗比喻**

想象一个**办公柜**（每个线程都有自己的柜子）：

- **ThreadLocal 对象** = 一张**标签贴纸**（比如写着“用户ID”）
- **线程** = 一个**员工**（每个员工有自己的柜子）
- **ThreadLocalMap** = 员工**柜子里的小抽屉**
- **实际存储的值** = 放在抽屉里的**物品**

当你调用 `threadLocal.set(value)` 时：

1. 找到当前员工（线程）
2. 打开他的柜子
3. 找到贴着“用户ID”标签的那个抽屉
4. 把物品放进去

**关键点**：物品始终在员工的柜子里，而不是贴纸上。

每个`Thread`中都具备一个`ThreadLocalMap`，而`ThreadLocalMap`可以存储以`ThreadLocal`为 key ，Object 对象为 value 的键值对。





### ThreadLocal 内存泄露问题

`ThreadLocal` 内存泄漏的根本原因在于其内部实现机制。每个线程维护一个名为 `ThreadLocalMap` 的 map。 当你使用 `ThreadLocal` 存储值时，实际上是将值存储在当前线程的 `ThreadLocalMap` 中，其中 `ThreadLocal` 实例本身作为 key，而你要存储的值作为 value。

`ThreadLocal` 的 `set()` 方法源码如下：

```java
public void set(T value) {
    Thread t = Thread.currentThread(); // 获取当前线程
    ThreadLocalMap map = getMap(t);   // 获取当前线程的 ThreadLocalMap
    if (map != null) {
        map.set(this, value);         // 设置值
    } else {
        createMap(t, value);          // 创建新的 ThreadLocalMap
    }
}
```

`ThreadLocalMap` 的 `set()` 和 `createMap()` 方法中，并没有直接存储 `ThreadLocal` 对象本身，而是使用 `ThreadLocal` 的哈希值计算数组索引，最终存储于类型为`static class Entry extends WeakReference<ThreadLocal<?>>`的数组中。

```java
int i = key.threadLocalHashCode & (len-1);
```

`ThreadLocalMap` 的 `Entry` 定义如下：

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

`ThreadLocalMap` 的 `key` 和 `value` 引用机制：

- **key 是弱引用**：`ThreadLocalMap` 中的 key 是 `ThreadLocal` 的弱引用 (`WeakReference<ThreadLocal<?>>`)。 这意味着，如果 `ThreadLocal` 实例不再被任何强引用指向，垃圾回收器会在下次 GC 时回收该实例，导致 `ThreadLocalMap` 中对应的 key 变为 `null`。
- **value 是强引用**：即使 `key` 被 GC 回收，`value` 仍然被 `ThreadLocalMap.Entry` 强引用存在，无法被 GC 回收。

当 `ThreadLocal` 实例失去强引用后，其对应的 value 仍然存在于 `ThreadLocalMap` 中，因为 `Entry` 对象强引用了它。如果线程持续存活（例如线程池中的线程），`ThreadLocalMap` 也会一直存在，导致 key 为 `null` 的 entry 无法被垃圾回收，即会造成内存泄漏。

也就是说，内存泄漏的发生需要同时满足两个条件：

1. `ThreadLocal` 实例不再被强引用；
2. 线程持续存活，导致 `ThreadLocalMap` 长期存在。

虽然 `ThreadLocalMap` 在 `get()`, `set()` 和 `remove()` 操作时会尝试清理 key 为 null 的 entry，但这种清理机制是被动的，并不完全可靠。

**如何避免内存泄漏的发生？**

1. 在使用完 `ThreadLocal` 后，务必调用 `remove()` 方法。 这是最安全和最推荐的做法。 `remove()` 方法会从 `ThreadLocalMap` 中显式地移除对应的 entry，彻底解决内存泄漏的风险。 即使将 `ThreadLocal` 定义为 `static final`，也强烈建议在每次使用后调用 `remove()`。
2. 在线程池等线程复用的场景下，使用 `try-finally` 块可以确保即使发生异常，`remove()` 方法也一定会被执行。



### 为什么 Entry 的 key 要设计为弱引用？

我们先来看完整的引用链路。当一个线程使用 `ThreadLocal` 时，涉及以下引用关系：

```
强引用（栈/静态变量）──→ ThreadLocal 实例
                              ↑
Thread ──→ ThreadLocalMap ──→ Entry ─── key（WeakReference）──┘
                              │
                              └─── value（强引用）──→ 实际存储的对象
```

理解了这条引用链路，我们来对比两种设计方案：

**假设 key 使用强引用（实际没有采用）：**

当业务代码中的 `ThreadLocal` 引用被置为 `null`（例如方法执行结束、对象被回收），此时虽然业务代码已经不再需要这个 `ThreadLocal`，但由于 `ThreadLocalMap` 的 Entry 对 key 持有**强引用**，`ThreadLocal` 实例仍然无法被 GC 回收。只要线程不终止，这个 `ThreadLocal` 和它对应的 value 都会一直存在于内存中，造成 key 和 value **都无法回收**的内存泄漏。

**key 使用弱引用（实际采用的方案）：**

当业务代码中的 `ThreadLocal` 引用被置为 `null` 后，由于 Entry 的 key 是弱引用，`ThreadLocal` 实例在下次 GC 时会被回收，key 变为 `null`。此时虽然 value 仍然存在（强引用），但 `ThreadLocalMap` 在执行 `get()`、`set()`、`remove()` 等操作时，会主动探测并清理这些 key 为 `null` 的 "stale entry"（过期条目），从而释放 value 对象。

也就是说，**弱引用的设计是一种"兜底"防御机制**——即便开发者忘记调用 `remove()`，JVM 的 GC 配合 `ThreadLocalMap` 的自清理逻辑，仍然有机会回收泄漏的数据。而如果使用强引用，一旦忘记 `remove()`，就完全没有任何补救机会了。

> 需要注意的是，这种自清理机制是**被动触发**的（只在 `get`/`set`/`remove` 操作时顺便清理），并不能保证所有过期条目都被及时清理。因此，**弱引用只是降低了内存泄漏的风险，并没有彻底消除它**，手动调用 `remove()` 仍然是必须的。



## 线程池

### 优势

线程池就是管理一系列线程的资源池。当有任务要处理时，直接从线程池中获取线程来处理，处理完之后线程并不会立即被销毁，而是等待下一个任务。

1. **降低资源消耗**：线程池里的线程是可以重复利用的。一旦线程完成了某个任务，它不会立即销毁，而是回到池子里等待下一个任务。这就避免了频繁创建和销毁线程带来的开销。
2. **提高响应速度**：因为线程池里通常会维护一定数量的核心线程（或者说“常驻工人”），任务来了之后，可以直接交给这些已经存在的、空闲的线程去执行，省去了创建线程的时间，任务能够更快地得到处理。
3. **提高线程的可管理性**：线程池允许我们统一管理池中的线程。我们可以配置线程池的大小（核心线程数、最大线程数）、任务队列的类型和大小、拒绝策略等。这样就能控制并发线程的总量，防止资源耗尽，保证系统的稳定性。同时，线程池通常也提供了监控接口，方便我们了解线程池的运行状态（比如有多少活跃线程、多少任务在排队等），便于调优。



### 用到的设计模式

1. **工厂模式**，体现在线程池的创建过程中。Java 中`Executors`类提供的`newFixedThreadPool`、`newCachedThreadPool`等静态方法，本质上是工厂方法，它们封装了`ThreadPoolExecutor`复杂的参数配置，根据不同场景返回预配置的线程池实例，简化了用户创建线程池的操作。用户无需关心线程池内部参数的细节，只需根据需求选择对应的工厂方法即可。
2. **享元模式**，这是线程池的核心设计思想之一。线程池通过复用已创建的线程（尤其是核心线程）来执行多个任务，避免了频繁创建和销毁线程带来的资源消耗。线程作为 “共享资源”，在完成一个任务后不会被销毁，而是回到线程池等待新的任务，这种对线程资源的共享复用，正是享元模式的典型应用，有效提高了系统的资源利用率。
3. **生产者 - 消费者模式**，体现在任务的提交与执行流程中。用户通过`execute`或`submit`方法提交任务（生产者），任务被放入阻塞队列中；线程池中的工作线程（消费者）不断从队列中取出任务并执行。阻塞队列作为缓冲区，平衡了生产者和消费者的处理速度，避免了任务提交过快导致的线程资源不足，或线程空闲时的资源浪费，实现了任务的异步处理和解耦。
4. **模板方法模式**。`ThreadPoolExecutor`的`execute`方法定义了任务执行的整体流程框架：先尝试添加任务到核心线程，失败则加入队列，再失败则创建非核心线程，最后执行拒绝策略。这个流程是固定的模板，而具体的任务执行、线程创建等细节则通过重写`beforeExecute`、`afterExecute`等钩子方法进行扩展，用户可以在不改变核心流程的前提下自定义任务执行前后的操作。



### 创建线程池两种方式：

**通过 `Executors` 工厂类创建（快速但不推荐生产环境）**

| 方法                                      | 线程池类型      | 特点                                         | 适用场景                     |
| :---------------------------------------- | :-------------- | :------------------------------------------- | :--------------------------- |
| `Executors.newFixedThreadPool(int n)`     | 固定大小线程池  | 核心线程数 = 最大线程数                      | 任务数量可控，资源占用稳定   |
| `Executors.newCachedThreadPool()`         | 缓存线程池      | 核心=0，最大=Integer.MAX_VALUE，空闲60秒回收 | **短生命周期、大量临时任务** |
| `Executors.newSingleThreadExecutor()`     | 单线程池        | 核心=最大=1，保证任务顺序执行                | 需要顺序处理任务的队列       |
| `Executors.newScheduledThreadPool(int n)` | 定时/延迟线程池 | 支持延迟执行和周期性任务                     | 定时任务、延迟任务           |

缺点（阿里规约明确禁止生产环境使用）：

- `FixedThreadPool` 和 `SingleThreadPool` 使用无界队列 → 可能OOM
- `CachedThreadPool` 最大线程数过大 → 可能创建过多线程导致OOM

**通过 `ThreadPoolExecutor` 构造函数创建（推荐生产环境）**

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    int corePoolSize,           // 核心线程数
    int maximumPoolSize,        // 最大线程数
    long keepAliveTime,         // 空闲线程存活时间（默认只控制非核心线程，超出 corePoolSize 的部分）
    TimeUnit unit,              // 时间单位
    BlockingQueue<Runnable> workQueue,  // 任务队列
    ThreadFactory threadFactory,        // 线程工厂
    RejectedExecutionHandler handler    // 拒绝策略
);
```

**参数详解**

| 参数              | 含义                                       | 常见配置                                  |
| :---------------- | :----------------------------------------- | :---------------------------------------- |
| `corePoolSize`    | 核心线程数，即使空闲也会保留               | CPU密集型：CPU核数+1；IO密集型：2×CPU核数 |
| `maximumPoolSize` | 最大线程数（队列满后才会创建新线程到该值） | 通常为corePoolSize的2-4倍                 |
| `keepAliveTime`   | 超出corePoolSize的线程空闲存活时间         | 通常60秒                                  |
| `workQueue`       | 任务队列（见下表）                         | 根据是否需要排队选择                      |
| `handler`         | 队列满且线程数达最大时的拒绝策略           | 4种内置策略                               |

**常用队列选择**

| 队列类型                | 特点                         | 适用场景                         |
| :---------------------- | :--------------------------- | :------------------------------- |
| `ArrayBlockingQueue`    | 有界队列，需指定容量         | 任务数量可控，防止OOM            |
| `LinkedBlockingQueue`   | 可选择有界/无界（默认无界）  | 无界→任务过多会OOM               |
| `SynchronousQueue`      | 不存储任务，直接交给线程     | `newCachedThreadPool` 使用的队列 |
| `PriorityBlockingQueue` | 优先级队列，可设置任务优先级 | 需要任务按优先级执行             |

**内置拒绝策略**

| 策略                  | 行为                                  | 风险             |
| :-------------------- | :------------------------------------ | :--------------- |
| `AbortPolicy`（默认） | 直接抛出 `RejectedExecutionException` | 需要业务捕获处理 |
| `CallerRunsPolicy`    | 由调用者线程执行该任务                | 可能阻塞主线程   |
| `DiscardPolicy`       | 静默丢弃任务，不抛异常                | 任务会丢失       |
| `DiscardOldestPolicy` | 丢弃队列头部的任务，再提交当前任务    | 丢失旧任务       |



### 线程池的核心线程会被回收吗？

`ThreadPoolExecutor` 默认不会回收核心线程，即使它们已经空闲了。这是为了减少创建线程的开销，因为核心线程通常是要长期保持活跃的。但是，如果线程池是被用于周期性使用的场景，且频率不高（周期之间有明显的空闲时间），可以考虑将 `allowCoreThreadTimeOut(boolean value)` 方法的参数设置为 `true`，这样就会回收空闲（时间间隔由 `keepAliveTime` 指定）的核心线程了。



### CallerRunsPolicy 拒绝策略有什么风险？如何解决？

如果想要保证任何一个任务请求都要被执行的话，那选择 `CallerRunsPolicy` 拒绝策略更合适一些。不过，如果走到`CallerRunsPolicy`的任务是个非常耗时的任务，且处理提交任务的线程是主线程，可能会导致主线程阻塞，影响程序的正常运行。

调用者采用`CallerRunsPolicy`是希望所有的任务都能够被执行，暂时无法处理的任务又被保存在阻塞队列`BlockingQueue`中。这样的话，在内存允许的情况下，我们可以增加阻塞队列`BlockingQueue`的大小并调整堆内存以容纳更多的任务，确保任务能够被准确执行。

**自定义拒绝策略**

只需实现 `RejectedExecutionHandler` 接口，并重写 `rejectedExecution` 方法即可。

```java
public class CustomRejectedPolicy implements RejectedExecutionHandler {
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        // 实现你的自定义逻辑
    }
}
```



常见自定义拒绝策略示例

1. 日志记录 + 任务持久化（最实用）

```java
public class LogAndPersistPolicy implements RejectedExecutionHandler {
    private final BlockingQueue<Runnable> backupQueue = new LinkedBlockingQueue<>(10000);
    
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        // 1. 记录告警日志
        System.err.printf("[Rejected] Active: %d, Queue: %d, Pool: %d%n",
            executor.getActiveCount(),
            executor.getQueue().size(),
            executor.getPoolSize());
        
        // 2. 尝试放入备份队列（降级存储）
        if (!backupQueue.offer(r)) {
            // 备份队列也满了，写入文件或数据库
            saveToDisk(r);
        }
    }
    
    private void saveToDisk(Runnable r) {
        // 序列化任务到文件，待后续恢复
    }
}
```

2. 动态扩容（激进策略）

```java
public class DynamicExpandPolicy implements RejectedExecutionHandler {
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        // 触发拒绝时，动态增加最大线程数
        if (executor.getMaximumPoolSize() < 500) {
            executor.setMaximumPoolSize(executor.getMaximumPoolSize() + 10);
            System.out.println("线程池扩容到: " + executor.getMaximumPoolSize());
        }
        
        // 扩容后重新提交
        executor.execute(r);
    }
}
```

3. 优先级任务处理（业务分级）

```java
public class PriorityPolicy implements RejectedExecutionHandler {
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        if (r instanceof PriorityTask) {
            PriorityTask task = (PriorityTask) r;
            
            if (task.getLevel() == Level.CRITICAL) {
                // 关键任务：由调用线程执行（保险）
                r.run();
            } else if (task.getLevel() == Level.NORMAL) {
                // 普通任务：记录并丢弃
                log.warn("普通任务被丢弃: {}", task);
            } else {
                // 低优先级任务：直接丢弃
                log.debug("低优先级任务被丢弃");
            }
        } else {
            // 兜底：直接丢弃
            r.run();  // 或直接丢弃
        }
    }
}
```

4. 死信队列 + 异步重试
5. 监控告警 + 快速失败



### 线程池处理任务的流程

1. 如果当前运行的线程数小于核心线程数，那么就会新建一个线程来执行任务。
2. 如果当前运行的线程数等于或大于核心线程数，但是小于最大线程数，那么就把该任务放入到任务队列里等待执行。
3. 如果向任务队列投放任务失败（任务队列已经满了），但是当前运行的线程数是小于最大线程数的，就新建一个线程来执行任务。
4. 如果当前运行的线程数已经等同于最大线程数了，新建线程将会使当前运行的线程超出最大线程数，那么当前任务会被拒绝，拒绝策略会调用`RejectedExecutionHandler.rejectedExecution()`方法。

<div align="center"> <img src="https://oss.javaguide.cn/github/javaguide/java/concurrent/thread-pool-principle.png" width="100%"/> </div>



### 线程池中线程异常后，销毁还是复用？

- **使用`execute()`提交任务**：当任务通过`execute()`提交到线程池并在执行过程中抛出异常时，如果这个异常没有在任务内被捕获，那么该异常会导致当前线程终止，并且异常会被打印到控制台或日志文件中。线程池会检测到这种线程终止，并创建一个新线程来替换它，从而保持配置的线程数不变。
- **使用`submit()`提交任务**：对于通过`submit()`提交的任务，如果在任务执行中发生异常，这个异常不会直接打印出来。相反，异常会被封装在由`submit()`返回的`Future`对象中。当调用`Future.get()`方法时，可以捕获到一个`ExecutionException`。在这种情况下，线程不会因为异常而终止，它会继续存在于线程池中，准备执行后续的任务。

> **`execute()` 的异常会"烧死"线程，线程池不得不"换人"；`submit()` 的异常被 `Future` 这个"保护盒"包住了，线程根本不知道异常发生，所以安心复用。**



### 线程池中shutdown()，shutdownNow()区别

**`shutdown()` 是“温柔关闭”，不再接收新任务，但会等待已有任务执行完毕；`shutdownNow()` 是“暴力关闭”，试图中断正在执行的任务，并清空等待队列中的任务。**

* `shutdown`使用了以后会置状态为`SHUTDOWN`，正在执行的任务会继续执行下去，没有被执行的则中断。此时，则不能再往线程池中添加任何任务，否则将会抛出 `RejectedExecutionException `异常
* `shutdownNow `为`STOP`，并试图停止所有正在执行的线程，不再处理还在池队列中等待的任务，当然，它会返回那些未执行的任务。 它试图终止线程的方法是通过调用 `Thread.interrupt()` 方法来实现的，但是这种方法的作用有限，如果线程中没有`sleep` 、`wait`、`Condition`、定时锁等应用， `interrupt()`方法是无法中断当前的线程的。所以，`ShutdownNow()`并不代表线程池就一定立即就能退出，它可能必须要等待所有正在执行的任务都执行完成了才能退出。



### 提交给线程池中的任务能否被撤回

通过提交任务时返回的 `Future` 对象，可以尝试取消该任务。

`Future` 接口提供了 `cancel(boolean mayInterruptIfRunning)` 方法来实现取消操作。该方法的核心逻辑是：

- 如果任务尚未开始执行，会被直接取消，不会进入线程池运行。
- 如果任务已经开始执行，则由参数 `mayInterruptIfRunning` 决定：设为 `true` 时会尝试中断正在运行的线程；设为 `false` 时则不会中断，让任务继续执行完成。

简单来说：**未开始的任务可以无条件取消；已开始的任务需要看是否允许被中断。**

此外，`Future` 还提供了 `isCancelled()` 判断任务是否已被取消，以及 `isDone()` 判断任务是否已执行完毕（包括正常完成、异常或取消）。



## Future

将耗时任务提交给子线程异步执行，主线程可以继续处理其他事务，无需阻塞等待。待主线程工作完成后，再通过 `Future` 获取异步任务的结果，从而显著提升程序执行效率。

在 Java 中，`Future` 类只是一个泛型接口，位于 `java.util.concurrent` 包下，其中定义了 5 个方法，主要包括下面这 4 个功能：

- 取消任务；
- 判断任务是否被取消;
- 判断任务是否已经执行完成;
- 获取任务执行结果。



### Callable 和 Future 的关系

`Callable` 和 `Future` 的关系可以概括为：**`Callable` 是任务的“生产者”，`Future` 是结果的“凭证”和“控制器”**。

> **`Callable` 定义任务，`Future` 管理任务和执行结果。**

```java
// Callable：定义任务（有返回值的 Runnable）
Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 42;
};

// Future：获取任务的凭证，用于控制任务和获取结果
Future<Integer> future = executor.submit(task);

// 通过 Future 获取结果
Integer result = future.get();  // 阻塞直到任务完成
```

| 维度         | **Callable**             | **Future**                                  |
| :----------- | :----------------------- | :------------------------------------------ |
| **本质**     | 任务本身（定义要做什么） | 任务的凭证（用来控制任务）                  |
| **核心方法** | `call()`                 | `get()`、`cancel()`、`isDone()`             |
| **返回值**   | 有（泛型定义返回类型）   | 通过 `get()` 返回 Callable 的结果           |
| **异常处理** | 可以抛出 checked 异常    | 通过 `get()` 时的 `ExecutionException` 获取 |
| **谁创建**   | 开发者实现               | 线程池（`submit()` 方法返回）               |
| **能否复用** | 可以提交给多个线程池     | 每个 Future 对应一次提交                    |







## Semaphore

`Semaphore` 的中文意思是“信号量”，它的核心作用是**控制同时访问特定资源的线程数量**，实现限流。

可以把它理解成一个**管理“通行证”的计数器**：线程要执行操作，必须先向 `Semaphore` 申请到一张“通行证”（`acquire()` 方法），执行完后再归还（`release()` 方法）。如果通行证发完了，后来的线程就只能阻塞等待，直到有其他线程归还。

### 底层原理

`Semaphore` 是共享锁的一种实现，它默认构造 AQS 的 `state` 值为 `permits`，你可以将 `permits` 的值理解为许可证的数量，只有拿到许可证的线程才能执行。

调用`semaphore.acquire()` ，线程尝试获取许可证，如果 `state >= 0` 的话，则表示可以获取成功。如果获取成功的话，使用 CAS 操作去修改 `state` 的值 `state=state-1`。如果 `state<0` 的话，则表示许可证数量不足。此时会创建一个 Node 节点加入阻塞队列，挂起当前线程。

调用`semaphore.release();` ，线程尝试释放许可证，并使用 CAS 操作去修改 `state` 的值 `state=state+1`。释放许可证成功之后，同时会唤醒同步队列中的一个线程。被唤醒的线程会重新尝试去修改 `state` 的值 `state=state-1` ，如果 `state>=0` 则获取令牌成功，否则重新进入阻塞队列，挂起线程。

### 两个经典应用场景

1. 限流

用来保护一个脆弱的资源，防止一瞬间有过多的请求把它压垮。

**代码示例**：

```java
// 最多允许 5 个线程同时访问
Semaphore semaphore = new Semaphore(5);

for (int i = 0; i < 20; i++) {
    new Thread(() -> {
        try {
            semaphore.acquire();   // 获取许可，没有则等待
            System.out.println(Thread.currentThread().getName() + " 访问资源");
            Thread.sleep(1000);    // 模拟操作
            semaphore.release();   // 释放许可
        } catch (InterruptedException e) {}
    }).start();
}
```

执行效果：同一时刻最多只能看到 5 个线程在工作。

2. 控制资源池

管理一个有限的资源池（如数据库连接池）。线程需要用资源时，从池子里取；用完后归还，方便其他线程使用。



## CountDownLatch

`CountDownLatch` 允许 任意数量个线程阻塞在一个地方，直至`count`个线程的任务都执行完毕(`count`次countDown方法)。

`CountDownLatch` 是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当 `CountDownLatch` 使用完毕后，它不能再次被使用。

### 底层原理

`CountDownLatch` 是共享锁的一种实现,它默认构造 AQS 的 `state` 值为 `count`。当线程使用 `countDown()` 方法时,其实使用了`tryReleaseShared`方法以 CAS 的操作来减少 `state`,直至 `state` 为 0 。当调用 `await()` 方法的时候，如果 `state` 不为 0，那就证明任务还没有执行完毕，`await()` 方法就会一直阻塞，也就是说 `await()` 方法之后的语句不会被执行。直到`count` 个线程调用了`countDown()`使 state 值被减为 0，或者调用`await()`的线程被中断，该线程才会从阻塞中被唤醒，`await()` 方法之后的语句得到执行。



### 3个线程并发执行，1个线程等待这三个线程全部执行完再执行，怎么实现？

可以使用 `CountDownLatch` 来实现 3 个线程并发执行，另一个线程等待这三个线程全部执行完再执行的需求。以下是具体的实现步骤：

- 创建一个 `CountDownLatch` 对象，并将计数器初始化为 3，因为有 3 个线程需要等待。
- 创建 3 个并发执行的线程，在每个线程的任务结束时调用 `countDown` 方法将计数器减 1。
- 创建第 4 个线程，使用 `await` 方法等待计数器为 0，即等待其他 3 个线程完成任务。

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {
    public static void main(String[] args) {
        // 创建一个 CountDownLatch, 初始计数为 3
        CountDownLatch latch = new CountDownLatch(3);

        // 创建并启动 3 个并发线程
        for (int i = 0; i < 3; i++) {
            final int threadNumber = i + 1;
            new Thread(() -> {
                try {
                    System.out.println("Thread " + threadNumber + " is working.");
                    // 模拟线程执行任务
                    Thread.sleep((long) (Math.random() * 1000));
                    System.out.println("Thread " + threadNumber + " has finished.");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    // 任务完成后, 计数器减 1
                    latch.countDown();
                }
            }).start();
        }

        // 创建并启动第 4 个线程, 等待其他 3 个线程完成
        new Thread(() -> {
            try {
                System.out.println("Waiting for other threads to finish.");
                // 等待计数器为 0
                latch.await();
                System.out.println("All threads have finished, this thread starts to work.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

**代码解释**：

- 首先，创建了一个 `CountDownLatch` 对象 `latch`，并将其初始计数设置为 3。
- 然后，使用 `for` 循环创建并启动 3 个线程。每个线程会执行一些工作（这里使用 `Thread.sleep` 模拟），在工作完成后，会调用 `latch.countDown()` 方法，将 `latch` 的计数减 1。
- 最后，创建第 4 个线程。这个线程在开始时调用 `latch.await()` 方法，它会阻塞，直到 `latch` 的计数为 0，即前面 3 个线程都调用了 `countDown()` 方法。一旦计数为 0，该线程将继续执行后续任务。



## CyclicBarrier

