在生产者/消费者模式中使用锁和semaphores。，这个模块是关于同步的。现在我们在Java语言中有两种同步的方式，一种是可以用在多个方面的synchronized关键字，另一种是可以用在失败声明上的volatile关键字。这两个关键字与所谓的内在锁定有关。还有显式锁定，这就是我们首先要看到的主要基于锁接口的使用。然后，我们将看到，使用这个锁接口和条件接口，我们可以以一种不同的、更强大的方式实现等待/通知模式。然后，我们将看到什么是semaphores。Semaphores在Java中并不是一个新的概念。它是一个来自操作系统的概念，在其他语言中也有实现。我们将看到semaphores的Java风味。

### Lock
#### 本质锁和同步有什么问题？

让我们来介绍一下本征锁和显式锁。如果我们想同步一个方法，例如，这里是Person类的init方法。最好的方法是创建一个key对象，它可以是任何对象，并在这个方法中放置一个同步块。这个key的同步代码可以防止一个以上的线程同时执行这个被保护的代码块。这在Java中被称为同步模式，它是众所周知的。现在，如果有几个线程试图在init方法中执行由synchronized关键字保护的代码块，会发生什么？其中一个线程将被允许进入该代码块，其他线程将不得不等待轮到自己执行同一代码块。这就是Java中的基本同步模式。现在，如果一个线程在代码块内被阻塞，会发生什么呢？我指的是阻塞，我指的是可能被错误地阻塞。也就是说，init方法中存在某种错误，会阻止线程退出这个受保护的代码块。好吧，事实证明，所有其他的线程也被封锁了。没有其他线程会被允许进入这个代码块。所有等待进入这个代码块的线程也被封锁了，JDK和JVM都没有办法释放它们。因此，当这种情况发生时，大多数时候，解决这个问题的唯一方法是重新启动JVM，也就是关闭应用程序，然后再重新加载。当然，这是我们要不惜一切代价避免的。

```java
public class Person{ 
    private final Objec tkey =new Object();
    public String init() {
        synchronized(key) {
            // do some stuff
        }
    }
}
```

#### 用Lock接口介绍API锁定
Lock Pattern 正是为了给这种情况带来一个解决方案。事实上，Lock Pattern带来了一个更丰富的API来处理这种情况。我们不需要写这样的代码，即创建一个密钥对象并将这个密钥对象传递给一个同步的代码块，而是要写这样的代码。我们创建一个Lock接口的实例，JDK提供了一个名为ReentrantLock的实现类，在代码的try finally块中，我们调用这个Lock对象的lock方法，并在finally部分调用unlock方法，从而保证在退出这个代码块时，无论lock调用之后发生什么，都会调用这个unlock方法。Lock是一个由ReentrantLock实现的接口。它是2004年Java 5中引入的Java.util.并发API的一部分。它提供了完全相同的保证，即明确的执行和读写排序，也就是可见性，发生在操作之间的链接之前，就像synchronize模式一样。而且它还提供了更多的功能。为什么呢？因为它不是一个语言基元，也就是同步块的情况，而是一个API，在这个API上，我们可以有很多方法。

```java
Lock lock=new ReentrantLock();
try{
    lock.lock();
// do some stuff
} finally {
    lock.unlock();
}
```

现在让我们仔细看看这两种模式。一方面，我们有同步模式。我们创建任何对象的实例来使用在这个对象上定义的监视器，这个对象可以根据我们的需要用来保护许多代码块。在这个锁模式中，我们创建一个Lock接口的Lock对象实例，最大的区别是在这个锁对象上我们有锁接口的方法，通过使用这些方法，我们将有更多的模式和更多的功能来守护代码块和处理锁的获取。
本质锁和同步有什么问题？
让我们来介绍一下本征锁和显式锁。如果我们想同步一个方法，例如，这里是Person类的init方法。最好的方法是创建一个key对象，它可以是任何对象，并在这个方法中放置一个同步块。这个key的同步代码可以防止一个以上的线程同时执行这个被保护的代码块。这在Java中被称为同步模式，它是众所周知的。现在，如果有几个线程试图在init方法中执行由synchronized关键字保护的代码块，会发生什么？其中一个线程将被允许进入该代码块，其他线程将不得不等待轮到自己执行同一代码块。这就是Java中的基本同步模式。现在，如果一个线程在代码块内被阻塞，会发生什么呢？我指的是阻塞，我指的是可能被错误地阻塞。也就是说，init方法中存在某种错误，会阻止线程退出这个受保护的代码块。好吧，事实证明，所有其他的线程也被封锁了。没有其他线程会被允许进入这个代码块。所有等待进入这个代码块的线程也被封锁了，JDK和JVM都没有办法释放它们。因此，当这种情况发生时，大多数时候，解决这个问题的唯一方法是重新启动JVM，也就是关闭应用程序，然后再重新加载。当然，这是我们要不惜一切代价避免的。

### 锁定模式 
#### 可中断的锁获取
那么，这种锁模式给我们带来了什么？首先，它带来了可中断的锁获取。让我们来看看这个例子。在这里，我们有一个基本的模式，它的锁方法调用将保护锁和解锁调用之间的代码块。取而代之的是，我们可以调用lock.Interruptibly，其语义与锁调用相同，即调用此方法的线程将被阻塞，直到被保护的代码块可以被执行。现在，如果另一个线程有一个对它的引用，它可以调用线程上的中断方法和lockInterruptibly方法，而不是让它执行通过自己中断异常的代码，从而读取等待线程。这在同步模式下是不可能的。这可能是昂贵的。从纯粹的实现角度来看，这可能很难实现，但这是可能的。它可以作为一种功能。

**Interruptible** lock acquisitio
```java

Lock lock=new ReentrantLock();
try{
    lock.lockInterruptibly();
// do some stuff
} finally {
    lock.unlock();
}

//This is the interruptible pattern
// The thread will wait until it can enter the guarded block of code
// But another thread can interrupt it by calling its
// interrupt()method
// This can be costly, or hard to achieve though…
```
#### 定时锁获取
我们可以做的第二件事叫做定时锁获取。它现在是什么意思呢？它意味着我们可以不调用锁方法，而是调用tryLock方法，这时如果一个线程已经在执行被保护的代码块，调用的tryLock将立即返回false。因此，我们的线程将不会被阻塞，而是不会进入被保护的代码块，并能立即做其他事情。请注意，我们也可以给这个tryLock方法传递一个超时，例如，这里我们的线程将等待1秒。如果在这个超时之后，被保护的代码块仍然不可用，它将执行else代码块。
**Timed** lock acquisition
```java
Lock lock=new ReentrantLock();
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try{
        // guarded block of code
    } finally{
        lock.unlock();
    }
} else { ... }
```

#### 公平锁获取
我们的第三个功能叫做公平锁获取。它是如何工作的呢？假设我们有几个线程在等待一个特定的锁。无论是内在的还是显性的锁，第一个进入被保护的代码块的线程都是随机选择的。这是synchronized关键字和Lock对象的默认行为。公平性意味着第一个进入等待线的人也将是第一个进入受保护代码块的人。让我们在一个例子上看到这一点。默认情况下，以正常方式构建的ReentrantLock是不公平的，也就是说，如果有两个线程在等待获取这个锁，我们并不能事先知道哪个线程会先执行被保护的代码块。现在，如果我们在构建这个锁对象时传递true Boolean，这个锁对象就会变成一个公平的ReentrantLock或者一个公平的锁。这意味着，如果有两个线程在等待获得这个锁，那么第一个到达被保护的代码块的线程将是第一个进入等待列表的。实现这一点的成本很高，所以默认情况下不会激活公平性，只有在你绝对需要的时候才应该使用。
**Fair** lock acquisition
```java
    Lock lock=new ReentrantLock(true); // fairtry{
lock.lock();
// guarded block of code
} finally{
    lock.unlock();
}
```
!!! Important
        A ReentrantLockbuilt in the normal way is non-fair
        And one can also pass a
        booleanto it
        True: this lock is fair, false: this lock is non-fair
        A fair lock is costly…


正如我们所看到的，使用这个锁接口给了我们的代码和我们的应用程序一些回旋的余地。
+  一个锁可以被打断。这是可能的，这很难实现，但已经做到了，而且在我们的应用中，这的确是很昂贵的。但是如果我们肯定需要它，那么它就可以使用。
+ 这个Lock对象的获取也可以在一定时间内被阻断。我可以问这个Lock对象，并告诉嘿，如果在从现在开始的一次，这个锁是不可用的，那么我宁愿离开并做其他事情。
+ 最后，这种获取可以是公平的，以先到先得的方式让线程进入，对于可中断的锁也是如此。这是一个昂贵的激活功能，所以只有在你绝对需要它的时候才使用它。


### 生产者/消费者模式  

### 等待/通知的同步实现

现在让我们看看如何使用这个锁接口来实现生产者/消费者模式。使用内在锁实现的经典方法可能是使用等待和通知模式。Wait和notify是Object类中的方法，应该在同步代码块中调用。很明显，由于显式锁在同步代码块中不起作用，所以wait/notify模式根本就不能工作。所以我们需要另一种模式，这就是我们现在要看到的。让我们快速了解一下实现这一模式的经典方法。这里是生产者的代码，这里是消费者的代码。顺便说一下，如果你不熟悉这种模式，Pluralsight上的一门课程，叫做《将并发和多线程应用于常见的Java模式》。这两个类是围绕同步块组织的。如果缓冲区满了，那么生产线程必须等待，而为了被唤醒，消费者线程一旦从缓冲区中移除一个元素，就必须在同一个块对象上调用notify或notifyAll。这将产生唤醒生产线程的效果，使其能够继续工作。而另一边也是如此。如果缓冲区是空的，消费者线程当然不能消费任何元素，所以它必须等待，而要被唤醒，它必须得到生产线程的通知。这种模式的一个主要注意事项是，一个线程处于这种等待状态。一旦它调用了这个等待方法，它就被阻塞了，我们没有办法打断它。因此，如果没有线程在调用notify或notifyAll，那么这个线程就没有机会被唤醒了。中断它的唯一方法是重新启动应用程序，以重新启动Java机器本身。

```java

// 01
Object lock =new Object();
class Producer {
    public void produce() {
    synchronized(lock) {
        while(isFull(buffer)){
            lock.wait();
        }
            
        buffer[count++] = 1;
        lock.notifyAll();
    }
}

class Consumer{
public void consume() {
    synchronized(lock) {
        while(isEmpty(buffer))
            lock.wait();
        buffer[--count] = 0;
        lock.notifyAll();
        }
    }
}
```

#### 生产者/消费者模式。带条件的锁定实现
让我们通过使用这个锁模式来解决这个问题和其他问题。这里是生产者。我们刚刚用这个新的锁模式翻译了同步块，我们对消费者的代码有同样的组织。现在在这个受保护的代码块里面，我们仍然有相同的结构。当缓冲区满了的时候，我们必须让这个线程处于等待状态，而在消费端，当一个元素从缓冲区被消费掉的时候，消费者必须通知生产者缓冲区里还有空间。如何做到这一点呢？嗯，它是用一个新的条件类型的对象来完成的，并通过调用新的条件方法从这个Lock对象中创建。这里的第一个条件被称为notFull。我们将在生产方调用notFull.await，在消费方调用notFull.signal。这个await方法相当于wait/notify模式中的wait方法，这个signal方法相当于notify方法。为了用另一种方式来做，我们可以创建第二个条件对象，这次叫做notEmpty。如果缓冲区是空的，如果消费者想消费一个对象，我们就调用notEmpty.await，一旦生产者向缓冲区添加了一个对象，就调用notEmpty.signal。所以这就是我们最后的完整模式。它看起来像等待/通知模式，但没有使用同步块。

```java
package org.paumard.locks;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ProcucerConsumerWithLocks {

	public static void main(String[] args) throws InterruptedException {

		List<Integer> buffer = new ArrayList<>();

		Lock lock = new ReentrantLock();
		Condition isEmpty = lock.newCondition();
		Condition isFull = lock.newCondition();

		class Consumer implements Callable<String> {

			public String call() throws InterruptedException, TimeoutException {
				int count = 0;
				while (count++ < 50) {
					try {
						lock.lock();
						while (isEmpty(buffer)) {
							// wait
							if (!isEmpty.await(10, TimeUnit.MILLISECONDS)) {
								throw new TimeoutException("Consumer time out");
							}
						}
						buffer.remove(buffer.size() - 1);
						// signal
						isFull.signalAll();
					} finally {
						lock.unlock();
					}
				}
				return "Consumed " + (count - 1);
			}
		}

		class Producer implements Callable<String> {

			public String call() throws InterruptedException {
				int count = 0;
				while (count++ < 50) {
					try {
						lock.lock();
						// int i = 10/0;
						while (isFull(buffer)) {
							// wait
							isFull.await();
						}
						buffer.add(1);
						// signal
						isEmpty.signalAll();
					} finally {
						lock.unlock();
					}
				}
				return "Produced " + (count - 1);
			}
		}

		List<Producer> producers = new ArrayList<>();
		for (int i = 0; i < 4; i++) {
			producers.add(new Producer());
		}

		List<Consumer> consumers = new ArrayList<>();
		for (int i = 0; i < 4; i++) {
			consumers.add(new Consumer());
		}
		
		System.out.println("Producers and Consumers launched");
		
		List<Callable<String>> producersAndConsumers = new ArrayList<>();
		producersAndConsumers.addAll(producers);
		producersAndConsumers.addAll(consumers);

		ExecutorService executorService = Executors.newFixedThreadPool(8);
		try {
			List<Future<String>> futures = executorService.invokeAll(producersAndConsumers);

			futures.forEach(
					future -> {
						try {
							System.out.println(future.get());
						} catch (InterruptedException | ExecutionException e) {
							System.out.println("Exception: " + e.getMessage());
						}
					});

		} finally {
			executorService.shutdown();
			System.out.println("Executor service shut down");
		}

	}

	public static boolean isEmpty(List<Integer> buffer) {
		return buffer.size() == 0;
	}

	public static boolean isFull(List<Integer> buffer) {
		return buffer.size() == 10;
	}
}

```

### 条件对象。可中断性和公平性
这个条件对象是我们在这里介绍的新对象。在这个模式中，它被用来停放和唤醒线程。它是由Lock对象建立的，一个Lock对象可以有任意数量的Condition对象与之相连。现在我们需要小心一点，因为这个条件对象和所有的Java对象一样扩展了对象类，所以它有一个等待和一个通知方法。这些方法不应该被当作 await 和 signal。事实上，如果我们尝试使用这些方法，它们将不会起作用，因为我们不在一个同步的代码块中。这种模式给我们带来了什么？事实上，它带来了很多东西。await调用是阻塞的，但是，这也是与对象类的wait调用不同的地方，它可以被中断。我们可以中断在这个await调用中被阻塞的线程。而在来自对象类的等待方法上则不是这样的。事实上，这个`await`方法有五个版本，我们使用的普通await方法，三个await方法需要超时，可以用时间单位表示，例如2秒，或者纳秒，还有一个`awaitUntil`，需要一个日期作为参数，当然是未来的某个时间。如果我们不希望这个 await 调用被中断，我们也可以调用 `awaitUninterruptibly`。这将防止一个线程中断这个方法的调用。所以这个API给了我们一些方法来防止等待线程的阻塞与Condition API。在上一个节点中，我们看到有可能创建公平锁，而公平锁会产生公平条件。也就是说，如果几个线程同时调用这个等待方法，它们将以相同的顺序被唤醒。

!!! Notes
    The await() call is blocking, can be interrupted  
    There are fiveversions for await:
        - await()  
        - await(time, timeUnit)  
        - awaitNanos(nanosTimeout)  
        - awaitUntil(date)  
        - awaitUninterruptibly()  
    These are ways to prevent the blockingo f waiting threads with the Condition API  
    A fair Lock generates fair Condition  

我们现在可以总结一下关于锁和条件的这部分内容了。锁定和条件是等待/通知模式的另一种实现。它为建立更好的并发系统提供了回旋余地，提供了一种控制中断的方法，控制并发锁和锁获取的超时，并为我们的系统提供了公平性

### ReadWriteLock
#### 读写锁模式简介

现在让我们来谈谈读/写锁的问题。在某些情况下，不是说在大多数情况下，我们需要的是独占写。这就是我想做的，就是保护要修改一个变量、一个集合或一个地图的代码块，但我想允许对这个变量或这个集合或地图进行并行读取。而这并不是普通锁的工作方式。也就是说，如果我保护要修改这个变量的代码块和要读取它的代码块，我将有排他性的写和排他性的读。这就是读/写锁的作用，这就是我们现在要看到的。读写锁是一个接口，只有两个方法。第一个方法是readLock来获得一个读锁，第二个方法非常简单，是writeLock来获得一个写锁。这个readLock和这个writeLock都是lock的实例，也就是我们刚才看到的Lock接口。现在的规则是这样的。只有一个线程可以持有writeLock。当writeLock被持有时，没有人可以持有readLock。当然，有多少个线程都可以持有readLock。这意味着，如果我用writeLock守护一个代码块，这个代码块的执行将是排他的。而如果我用readLock来守护另一个代码块，那么这个代码块将被我需要的多个线程所使用。


#### 用ReadWriteLock实现一个高效的并发缓存
让我们在一个例子上看看。ReadWriteLock是一个接口，ReentrantReadWriteLock是由JDK提供的实现类。从这个readWriteLock对象中，我用readLock和writeLock方法创建了两个锁，readLock和writeLock。由于我从同一个readWriteLock中得到了这两个锁，所以它们形成了一对读和写的锁。这一点非常重要。这两个锁可以用来创建一个线程安全的缓冲区。一个缓存可以用一个基本的HashMap来实现。这里我们有一个Long和User的Map，它可以是一个数据库的缓存，long是用户的主键。读取这个缓存是由readLock来保护的，其模式与我们看到的使用基本Lock对象的模式相同。知道了这个readLock对象的语义，我们就知道任何数量的线程都可以在同一时间读取这个缓存。现在这是由writeLock保护的对地图的修改，再次使用相同的模式，但这次这个writeLocking保护了对缓存的修改，并将防止并发读取可能读到的损坏的值。这也可以用ConcurrentHashMap来实现。我们将在最后一个模块中看到。

```java
ReadWriteLock readWritelock= new ReentrantReadWriteLock();
Lock readLock= readWritelock.readLock();
Lock writeLock= readWritelock.writeLock();

Map<Long, User> cache= new HashMap<>();
// read
try{
    readLock.lock();
    return cache.get(key);
} finally{
    readLock.unlock();
}

// write
try{
    writeLock.lock();
    cache.put(key,value);
} finally{
    writeLock.unlock();
}
```

让我们对这个读/写锁的概念做一个简单的总结。它在一个单一的ReadWriteLock对象上工作，用来获得一个写锁和一个读锁。非常重要的一点是，这对读写锁必须由同一个ReadWriteLock对象创建。写操作是排斥其他写和读的，所以当一个线程在修改，例如，我们刚刚创建的缓存对象时，没有其他线程可以修改它，也没有其他线程可以从它那里读取，但读操作是自由的。它们可以并行地进行。因此，它允许极好的吞吐量，尤其是当我们有很多读和很少写的时候，这通常是我们创建缓存时的假设。

###  Semaphore Pattern
现在让我们看看本模块中关于信号的最后一部分。在并发编程中，semaphores是一个著名的概念。它并不是来自于Java。它来自于早期的独特的操作系统。它看起来像一个锁，事实上，它是某种锁，但它不允许在被保护的代码块中只有一个线程，而是允许多个线程，事实上，一个信号量的建立和一些许可，这个许可的数量是这个代码块中允许的线程数量。让我们在一个例子上看看。我们有一个Semaphore类。当我们在Java中创建一个信号量时，我们必须确定信号量的许可数量，然后有一个获取方法和一个释放方法来获取一个许可，并允许在一个受保护的代码块中释放这个许可。以正常方式，也就是默认方式建立的信号量是不公平的。这意味着，如果有线程在等待许可，它们将在受保护的代码块中被随机地接受。当然，这种获取方式是**阻塞**的，直到有许可证可用。所以在我们的例子中，只有5个线程被允许同时执行被守护的代码块。可以让一个信号量变得公平。如果我在构造这个对象时传递true Boolean和第二个参数，每个对象都会创建一个公平的semaphore。而且，我还可以一次要求一个以上的许可。这里的Acquire 2将请求2个许可证，如果只有1个可用，执行这段代码的线程将不得不等待第二个1被释放。当然，如果我们要求获得两个许可证，我们也应该释放两个许可证。

```java
Semaphore semaphore=new Semaphore(5,true); // fair
try{
    semaphore.acquire(2);
// guarded block of code
} finally{
    semaphore.release(2);
}
// A Semaphorecan be fair
// The acquire()can ask for more than one permit
// Then the release()call must release them all
```
#### 可中断性和定时的许可获取
这个API是建立在与锁API相同的想法上的，所以我也可以在semaphore API上处理中断性和超时。如果我在一个在获取调用中被阻塞的线程上调用中断方法，这个线程就会立刻抛出一个InterruptedException。如果我不想要这种行为，那么我可以在这个信号对象上调用acquisitionUninterruptibly方法，在这种情况下，这个线程不会被中断。释放目的的唯一方法是调用其释放方法。现在如果我中断这个线程，在这个方法调用的时刻，它不会做任何事情。但如果有一个许可证变得可用，这个线程将不被允许进入被保护的代码块。它反而会抛出这个InterruptedException。再一次，遵循与锁接口相同的想法，可以使获取立即进行。TryAcquire将查看是否有许可证，如果没有，它将失败，写成false，我将能够执行一些其他代码而不是被保护的代码块。我也可以给这个tryAcquire方法调用传递一个超时，这样这个方法在这个超时后就会写成false。
```java
Semaphore semaphore=new Semaphore(5);
try{
    semaphore.acquireUninterruptibly();
// guarded block of code
} finally{
    semaphore.release();
}
// By default if a waiting thread is interrupted it will throw an InterruptedException
// Uninterruptibility means that the thread cannot be interrupted
// It can be only be freed by calling its release() method
```

```java
Semaphore semaphore=new Semaphore(5);
try{
    if(semaphore.tryAcquire(1, TimeUnit.SECONDS))
        // guarded block of code
    else
    // I could not enter the guarded code
} finally{
    semaphore.release();
}
```

!!! Notes
    One can also set a timeout before failing to acquire a permit
    This pattern can also request more than one permit

我们刚刚看到了如何使用semaphore对象的模式，但这还不是全部。我们在这个Semaphore对象上还有一些特定的方法，这些方法在经典的Lock对象上是不存在的。这些方法使得semaphore不仅仅是一个有多个许可的锁。事实上，我们有方法来处理许可和等待的线程。首先，我们可以在semaphore被创建后减少许可的数量。不可能增加这个许可证的数量。我们也有方法来检查这个semaphore上是否有等待线程。是否有等待的线程？有多少个线程在等待？而且我们还可以获得等待线程的集合，这在Lock对象上是不可能的。所以我们现在可以在Semaphore对象上快速收尾这部分内容。Semaphore是建立在许可证的数量上的。这些许可可以通过不同的方式获得，并且必须由线程释放。这部分API基本上与Lock API相同，唯一的区别是，一个锁只有一个许可，而一个semaphore有多个。但除此之外，还可以查询semaphore中等待线程的数量。是否有等待的线程？有多少个？而且我们还可以获得线程的参考信息。

### 代码实践
略（见上）