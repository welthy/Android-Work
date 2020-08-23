[TOC]

### 1、创建Looper的方法有哪些？为什么一般不用prepareMainLooper()？一般什么时候会用到prepareMainLooper()？

**答：**

a、创建Looper的方法有2种：prepare()；prepareMainLooper()。

b、prepareMainLooper()方法一般是用于设置主线程的Looper，在ActivityThread.main中已经设置过。所以一般开发不需要设置。

c、目前了解只有应用创建的时候在ActivityThread中调用。

### 2、Looper中无限循环监听MessageQueue为什么没造成卡顿？

**答：**

MessageQueue中没有消息时，去取消息的话会进入nativePollOnce()中，该方法进入native中，由于epoll机制会释放CPU，并进入休眠等待状态，直到有消息时再被唤醒执行，所以不会造成卡顿。

### 3、1个线程对应1个handler吗？为什么

**答：**

不是，**1个线程可有多个handler。但1个线程对应1个looper**。这在Looper的prepare()操作中被限定了。

在Looper的prepare()方法中，就将其与ThreadLocal存储。key即为对应的线程，value就是存储的Looper。

如果已经存储过的话会抛出异常。

所以同一个线程可创建多个Handler，但所有Handler都对应同一个Looper。

### 4、简述Handler发送消息与获取消息流程。可流程图展示

无论是sendXXX()还是postXXX()最终都是走到sendMsgAtTime()，然后调用MessageQueue.enqueueMessage()方法，将消息入队。入队过程是根据制定好的规则入队，规则就是根据when参数决定谁排在前面，when越小则越靠前，若when相同则根据先执行先入队的原则入队。默认是插在**链表尾**

入队后，会唤醒MessageQueue的next()去取消息。MessageQueue的nativePollOnce()操作被唤醒然后从消息**链表头**取出消息并返回。取到消息后，通过消息的target（也就是发送消息的handler）dispatchMessage()方法将消息交给Handler，最后执行handler的handleMessage()方法去处理消息。

### 5、Message是如何实现消息池的？回收机制是怎么运作的？

通过obtian()方法从消息池中获取一条消息，若消息池中没有消息则新建一条消息；消息池是以命名为sPools的消息为首的链表，获取消息从链表头获取。

通过recycle()方法将消息放回到消息池中，若消息池中的消息数大于50则不放入消息池。回收也是重新放入链表头

### 6、Handler是如何实现不同线程间消息传递的？

handler将子线程消息放入关联的Looper的消息队列中，Looper再从消息队列中取出消息，完成线程间消息传递。

### 7、IdleHandler与Handler是什么关系？存在的意义是什么？

IdleHandler是Handler的一种特殊机制，用于在MessageQueue无消息或有消息但需要延迟执行的空闲期间内进行一些特殊操作。

如ActivityThread会进行gc回收等操作。一般是系统使用，目的是为了在MessageQueue空闲时可以执行一些其他操作。

### 8、IdleHandler的循环会不会进入死循环

不会。因为设置了标记位**pendingIdleHandlerCount**，该标记位只在next()第一次循环会被mIdleHandlers.size()即idleHandler数组的大小赋值，然后才会去遍历循环。遍历之后该标记位会被置0，这样就不会再进入被数组大小赋值的语句中，当该标记位<=0时，会执行continue继续遍历循环，而不会去执行遍历idleHandler数组的循环。

### 9、MessageQueue中的同步消息和异步消息代表什么？如何区分的？

异步消息是为Message设置了Asynchronous为true的消息。我们平常正常使用的都是同步消息。

异步消息会搭配异步屏障使用，在设置了消息屏障后，我们会取之后的第一条异步消息去处理。

### 10、postSyncBarrier()方法存在的意义么？

postSyncBarrier()方法就是插入一条不设置target的消息。这条消息就是屏障，在next()中取消息时会判断当前消息是不是一个屏障，然后会去取屏障后的一条异步消息，并选取它作为待处理消息。

在系统绘制体系的scheduleTraversals()中就会插入一条同步消息，然后再插入一条执行绘制的异步消息，这样可以加快绘制的执行速度。