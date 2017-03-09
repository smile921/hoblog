Looper Handler MessageQueue
==

<br/>

## 1. 异步消息处理线程


### 1.1 区别

**普通线程：** run() 方法内的代码后线程就结束了。  
**异步消息处理线程：** 线程会进入一个无限循环体，**循环一次就从内部的消息队列中取出一个消息，并回调该消息对应的处理方法**。然后继续循环下去，如果消息队列没有消息了，线程会停止；否则只有有新的消息，就继续循环。  


### 1.2 需求场景      

>  异步消息处理线程其本质上也是一个线程，只不过是被设计成了在一个无限循环体下，不断读取消息的场景。并且有消息则循环，无消息暂停的模式。

**需求场景：**      
- **1.** 任务需要常驻。比如说用于处理用户交互的任务。
- **2.** 任务需要根据外部传递的消息而执行不同的操作。


### 1.3 简单实现

<img src="http://ww2.sinaimg.cn/large/006lPEc9jw1f3il291ls9j31kw16c78d.jpg" width="640x"/>  

- **1.** 每个异步线程内部包含一个消息队列（MessageQueue)，队列中的消息一般采用排队机制
即先到达的消息会先得到处理。

- **2.** 线程的执行体中使用 while ( true ) 进行无限循环，循环体中从消息队列中取出消息，并且根
据消息的来源，回调其对应的消息处理函数。

- **3.** 其他外部线程可以向本线程的消息队列中发送消息，消息队列内部的读/写操作必须进行加锁
即消息队列不能同时进行读/ 写操作。


<br/>

## 2. Android 中的异步消息处理线程

<img src="http://ww4.sinaimg.cn/large/006lPEc9jw1f3il7024jdj31kw16odm7.jpg" width="640x"/>   

在线程内部有一个或多个 Handler 对象，外部程序通过该 Handler 对象向线程发送异步消息，消
息经由 Handler 传递到 MessageQueue 对象中。线程内部只能包含一个 MessageQueue 对象，线
程主执行函数中从 MessageQueue 中读取消息，并回调 Handler 对象中的回调函数 handleMessage()。   


<br/>

## 3. 线程局部存储

**变量的常见作用域：**

- **方法内部变量：** 作用域就是该方法内，每次调用该方法时，变量都会重新回到初始值。

- **类内部的变量：** 其作用域是该类所产生的对象，只要对象没有销毁，则对象内部的变量值
一直保持。

- **类内部静态变量：** 其作用域是整个过程，只要在该进程中，则该变量的值就一直保持，无论使用该类构造过多少个对象，该变量只有一个赋值，并一直保持。进程中哪个线程引用该变量，其值总是相同的，因为在编译器内部为静态变量分配了单独的内存空间。

**线程局部存储：**
> 同一个线程引用变量值相同，不同线程引用则变量值不相同。


<br/>

## 4. Looper

**Looper 的作用：**

- **1.** 调用静态方法 `prepare()` 为该线程创建一个 `消息队列` 。

- **2.** 调用 `loop()`， 线程进入无限循环的异步消息处理流程，从消息队列内读取消息，并处理消息。


### 4.1 prepare()

```java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
...
...
...
public static void prepare() {
    prepare(true);
}

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

创建一个 Looper 并将，Looper 存储在 ThreadLocal 内。所以，**相同线程访问的 Looper 是一样的，不同线程访问的 Looper 是不一样的。**

### 4.2 Looper 构造方法

由上面的 `new Looper(quitAllowed)` 方法。可以看出，调用了 Looper 的构造方法。

```java
private Looper(boolean quitAllowed) {  
    mQueue = new MessageQueue(quitAllowed);  
    mThread = Thread.currentThread();  
}
```

创建一个 `MessageQueue` 。

### 4.3 loop()

```java
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

        // This must be in a local variable, in case a UI event sets the logger
        Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }

        msg.target.dispatchMessage(msg);

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // Make sure that during the course of dispatching the
        // identity of the thread wasn't corrupted.
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }

        msg.recycleUnchecked();
    }
}
...
...
...
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
```

**主要操作：**

- **1.** `myLooper()` 拿到当前线程存储的 `Looper` 对象。

- **2.** 拿到 `MessageQueue` 后，进入到 `for (;;)` 无限循环内。

- **3.** 再逐个通过 `queue.next()` 取出 `Message`。

- **4.** 通过 `msg.target.dispatchMessage(msg)` 不难发现 `msg.target` 是一个Handler，执行了该 `msg` 的处理。

- **5.** 最后，`msg.recycleUnchecked();` 对消息的资源进行回收。对象占用的系统资源。因为 `Message` 类内部使用了一个数据池去保存 `Message` 对象，从而避免不停地创建和删除  `Message` 类对象。因此，每次处理完该消息后，需要将该 `Message` 对象表明为空闲，以便 `Message` 对象可以被重用。


<br/>

## 5. MessageQueue

> 消息队列采用排队方式对消息进行处理，就是先到的消息会先得到处理，但是如果消息本身指定
了被处理的时刻，则必须等到该时刻才能处理消息。消息在 MessageQueue 中使用 Message 类表
示，队列中的消息以链表的结构进行存储，Message 对象内部包含一个 next 变量，该变量指向下一
个消息。

**MessageQueue 的主要方法：**

- **取出消息：** `next()` 。

- **添加消息：** `enqueueMessage(Message msg, long when)` 。

### 5.1 next()
```java
Message next() {
    // Return here if the message loop has already quit and been disposed.
    // This can happen if the application tries to restart a looper after quit
    // which is not supported.
    final long ptr = mPtr;
    if (ptr == 0) {
        return null;
    }

    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }

        nativePollOnce(ptr, nextPollTimeoutMillis);

        synchronized (this) {
            // Try to retrieve the next message.  Return if found.
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                // Stalled by a barrier.  Find the next asynchronous message in the queue.
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                if (now < msg.when) {
                    // Next message is not ready.  Set a timeout to wake up when it is ready.
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // Got a message.
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                    msg.markInUse();
                    return msg;
                }
            } else {
                // No more messages.
                nextPollTimeoutMillis = -1;
            }

            // Process the quit message now that all pending messages have been handled.
            if (mQuitting) {
                dispose();
                return null;
            }

            // If first time idle, then get the number of idlers to run.
            // Idle handles only run if the queue is empty or if the first message
            // in the queue (possibly a barrier) is due to be handled in the future.
            if (pendingIdleHandlerCount < 0
                    && (mMessages == null || now < mMessages.when)) {
                pendingIdleHandlerCount = mIdleHandlers.size();
            }
            if (pendingIdleHandlerCount <= 0) {
                // No idle handlers to run.  Loop and wait some more.
                mBlocked = true;
                continue;
            }

            if (mPendingIdleHandlers == null) {
                mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
            }
            mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
        }

        // Run the idle handlers.
        // We only ever reach this code block during the first iteration.
        for (int i = 0; i < pendingIdleHandlerCount; i++) {
            final IdleHandler idler = mPendingIdleHandlers[i];
            mPendingIdleHandlers[i] = null; // release the reference to the handler

            boolean keep = false;
            try {
                keep = idler.queueIdle();
            } catch (Throwable t) {
                Log.wtf(TAG, "IdleHandler threw exception", t);
            }

            if (!keep) {
                synchronized (this) {
                    mIdleHandlers.remove(idler);
                }
            }
        }

        // Reset the idle handler count to 0 so we do not run them again.
        pendingIdleHandlerCount = 0;

        // While calling an idle handler, a new message could have been delivered
        // so go back and look again for a pending message without waiting.
        nextPollTimeoutMillis = 0;
    }
}
```

 - **1.** `nativePollOnce(ptr, nextPollTimeoutMillis)`, 这是一个 **JNI** 函数，其作用是从消息队列中取出一个消息。`MessageQueue` 类内部**本身并没有保存消息队列，真正的消息队列数据保存在 JNI 中的 C 代码中**，**在 C 环境中创建了一个 NativeMessageQueue 数据对象**，这就是 `nativePollOnce()` 第一个参数的意义。它是一个 `int` 型变量，在C 环境中，该变量将被强制转换为一个 `NativeMessageQueue` 对象。在 C 环境中，如果消息队列中没有消息，将导致当前线程被挂起（ wait ) ;如果消息队列中有消息，则 C 代码中将把该消息赋值给 Java 环境中的 mMessages 变量。

- **2.** 在 `synchronized(this)` 块中，判断消息所指定的执行时间是否到了。如果到了，就返回该消息，并将 `mMessages` 变量置空；如果时间还没有到，则继续等待时间到了。

- **3.** 如果 `mMessages` 为空，则说明 C 环境中的消息队列没有可执行的消息了， 因此，执行 `mPendingldleHandlers` 列表中的 **"空闲回调函数"**。可以向 `MessageQueue` 中注册一些 **"空闲回调函数"**，从而当线程中没有消息可处理时去执行这些 **"空闲代码"**。


### 5.1 enqueueMessage(Message msg, long when)

```java
boolean enqueueMessage(Message msg, long when) {
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }

    synchronized (this) {
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            msg.recycle();
            return false;
        }

        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            // New head, wake up the event queue if blocked.
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            // Inserted within the middle of the queue.  Usually we don't have to wake
            // up the event queue unless there is a barrier at the head of the queue
            // and the message is the earliest asynchronous message in the queue.
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }

        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
```

- **1.** 通过 `mMessages = msg` 将参数 `msg` 赋值给 `mMessages`。

- **2.** 调用 `nativeWake(mPtr)`。JNI 函数，其内部会将 `mMessages` 消息添加到 C 环境中
的消息队列中，并且如果消息线程正处于挂起(wait)状态，则唤醒该线程。   


<br/>

## 6. Handler

在 `Looper` 的 `loop()` 的无限循环读取消息过程中，MessageQueue 虽然提供了读写消息的方法，但是却没有直接去调用，而是通过 msg.target（Hadnler）去处理消息。

> 一般使用 Handler 类 MessageQueue 中发送消息，并重载 Handler 类的 handleMessage() 方法添加消息处理代码。


### 6.1 Handler 构造方法

**Handler 对象只能添加到有 MessageQueue 的线程中，否则会发生异常。**

**虽然 Handler 的构造方法很多，但是都是基于这两个构造方法去实现的。**
```java

...

public Handler(Callback callback, boolean async) {
    if (FIND_POTENTIAL_LEAKS) {
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
        }
    }

    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}

...

public Handler(Looper looper, Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

- **1.** 第一个构造方法，没有在构造参数里指定 `Looper`。所以，通过 `Looper.myLooper()` 去拿**当前线程里存储在 ThreadLocal 内的 Looper**。（ Ps：我们知道主线程创建时会默认初始化好一个 `Looper`，并且调用其 `prepare()` `loop()`。这样，参照上面讲的 `Looper` 源码，已经初始化好一个 Looper 存放在主线程的静态 ThreadLocal 内以及一个 `MessageQueue`。）这里如果是子线程，没有手动 `Looper.prepare()` 和 `Looper.loop()` 会抛出异常，因为这里肯定拿不到 `MessageQueue` (一个 Looper 对应一个 MessageQueue) 。

- **2.** 构造参数指定了 Looper，就必然有 `MessageQueue`。

总结这里：**其实 Handler 强制性要拿一个 Looper，也只是为了一个 MessageQueue**。一个 Looper 的初始化，伴随着一个 MessageQueue 的诞生 ( Ps:见上述 Looper 源码分析 )。因为，`Handler` 的主要工作，还是操作 `MessageQueue` 。

### 6.3 Handler 发消息

**虽然 Handler 的 发消息的方法很多，但是都是基于这两个 核心发消息方法 去实现的。**
```java

...

public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}

...

public final boolean sendMessageAtFrontOfQueue(Message msg) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, 0);
}
```

可以看源码得知，所有的发消息方法都最终会调用这两个方法之一，这两个方法都会调用 `MessageQueue` 的方法去写一个消息进入消息队列内，最终的消息还是被添加到 C 环境中去。


### 6.4 Handler 处理消息

```java

...

public void dispatchMessage(Message msg) {
    // 1. 让 Handler.post(Runnable r) 处理消息。
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            // 2. 初始化 Handler 的时候，指定的 Callback 处理消息。
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        // 3. 让重写的 handleMessage(Message msg) 去处理消息。
        handleMessage(msg);
    }
}

...

private static void handleCallback(Message message) {
    message.callback.run();
}

...

public final boolean post(Runnable r) {
    return sendMessageDelayed(getPostMessage(r), 0);
}

...

private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}
```

其实在 `Looper.loop()` 源码里，已经看到过 `msg.target.dispatchMessage(msg)`，拿到 `Hander` （ `msg.target` ）去处理了消息。


**dispatchMessage 内的三种处理方式：**

- **1.** `handleCallback(msg)` ：是在 `post(Runnable r)` 和 `getPostMessage(Runnable r)` 之后，`getPostMessage(Runnable r)` 方法会 `m.callback = r` 将 `Runnable` 赋值给 `Message` 的 callback 属性。 再结合 `handleCallback(msg)` 的源码，会直接调用`Message` 的 callback 属性 （ Runnable ） 。

- **2.** `mCallback.handleMessage(msg)` ：这里的 `mCallback` ,就是 实例化 Handler 的时候需要传入的 `callback`。这也是 Handler 的一种使用方法之一 ( Ps: 在实例化的时候，传入构造参数 Handler.Callback 去处理消息。)。

- **3.** `handleMessage(msg)` ：让重写的 handleMessage(Message msg) 去处理消息。
