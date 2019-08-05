---
title: 同步异步阻塞非阻塞
date: 2019-08-02 11:22:28
tags: java
categories: java
---

### 同步和异步

同步：一个事件或者任务的执行，会使整个流程暂时等待，也就是说如果有多个任务要执行，必须要逐个进行。
异步：一个事件或者任务的执行，不会使整个流程暂时等待，也就是说如果有多个任务要执行，可以并发去执行。
**同步和异步的关键在于一个事件或者任务的执行是否会导致整个流程暂时等待。也就是任务是否是逐个完成的**


```java
public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(2);

        IOTest test = new IOTest();
        System.out.println("开始同步学习java");
        test.synDo();
        System.out.println("学完java学习nodeJS");
        test.synDo();
        System.out.println("java-node都同步学完了");
        System.out.println("------------------------------");
        System.out.println("开始学习java");
        test.asyDo(countDownLatch);
        System.out.println("学java的同时学习nodeJS");
        test.asyDo(countDownLatch);
        countDownLatch.await();
        System.out.println("java-node都异步学完了");

    }

    public void synDo() throws InterruptedException {
        Thread.sleep(5000);
    }

    public void asyDo(CountDownLatch countDownLatch) throws InterruptedException {

        Thread oneThread = new Thread(new MyThread(countDownLatch));
        oneThread.start();
    }

    class MyThread implements Runnable {

        private CountDownLatch countDownLatch;

        public MyThread(CountDownLatch countDownLatch) {
            this.countDownLatch = countDownLatch;
        }

        @Override
        public void run() {
            try {
                Thread.sleep(10000);

            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                countDownLatch.countDown();
            }
        }
    }

```

当在同步方式的时候，需要先学习完java然后再学习nodeJS，需要一个流程的结束再进行下一个流程。
当采用异步方式的时候，下一个流程不需要等待上一个流程的结束就可以执行


### 阻塞和非阻塞

阻塞:在某个事件或者任务执行的过程中，它发出了一个请求，但是由于该操作能够执行所需要的条件还没有达到，就会一直等待在那里，直到条件满足。
非阻塞：在某个事件或者任务执行的过程中，它发出了一个请求，如果请求条件不满足，会返回一个标志信息告知不满足，不需要一直在那里等待。

**阻塞和非阻塞的关键在于，当发出请求时，如果不满足请求时是一直等待还是不需要等待。**


### 阻塞IO和非阻塞IO

IO操作通常包括：对硬盘的读写、对socket的读写以及外围设备的读写
一个完整的IO请求读写操作包括
（1）查看数据是否就绪；
（2）进行数据拷贝；

当线程发出一个IO请求操作的时候，内核会查看所需要读写的数据是否已经准备就绪，
阻塞IO:如果数据没有准备就绪，就一直等待，直到数据准备就绪；
非阻塞IO:如果没有准备就绪会返回一个标志信息，不需要等待，等到数据准备就绪以后，内核会把数据拷贝到线程中去。
阻塞IO和非阻塞IO关键在于数据没有准备好时是否会等待；

阻塞I/O的一个特点是调用之后一定要等到系统内核层面完成所有操作后，调用才结束。已读取磁盘上的一段文件为例，系统内核
在完成磁盘寻道，读取数据，复制数据到内存中之后，这个调用才结束
阻塞IO造成CPU等待IO,浪费等待时间，CPU的处理能力不能得到充分利用
非阻塞IO与阻塞IO的区别就是 会立即返回，不会等待执行结果。非阻塞IO返回之后CPU可以做一些其他的事情，这可以提高性能。
但是由于完整的I/O并没有完成，立即返回的并不是业务层期望的数据，而仅仅是当前的调用的状态，为了获取结果需要不断轮训，而轮训也会造成cpu的浪费





### 四、同步IO和异步IO

同步IO:当一个线程请求进行IO操作时，当IO操作完成之前，该线程会被阻塞；
当线程发送IO请求操作时，数据没有准备就绪，就会使用线程或者内核不停地去询问数据有没有准备好，当数据准备就绪后将数据从内核拷贝到线程。
异步IO:当一个线程请求进行IO操作时，当IO操作完成之前，该线程会不会被阻塞；
异步IO只有IO操作请求是线程发出的，查看数据是否准备和对数据的拷贝都是通过内核来完成的，发送通知告知线程IO操作已经完成，在异步IO中，不会对线程产生任何阻塞。
同步IO和异步IO都是的关键区别在于数据拷贝阶段是由线程完成还是内核完成。所以异步IO必须要有底层操作系统的支持


### 五、常用的轮训技术

read: 最原始，性能最低的，通过重复调用来检查I/O的状态来读取完整的数据，cpu会一直耗在上面。类似while循环

select： 对read的改进，通过文件描述符上的事件状态来判断。select轮询具有一个较弱的限制，那就是由于采用了一个1024长度的数组来存储状态，所以它最多可以同时检查1024个文件描述符
 linux上有时候会报打开文件过多的错误，就是因为这个限制
 
poll: 对select方式的改进，采用链表的方式避免数组长度的限制，其次他能避免不需要的检查


epoll: 该方案是linux下效率最高的IO事件通知机制




