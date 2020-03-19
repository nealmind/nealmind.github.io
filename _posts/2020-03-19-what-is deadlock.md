---
layout: post
title: 死锁
tags: Java基础
author: neal
---
* content
{:toc}

## 锁顺序死锁
例如有两个方法分别持有锁`L`,`R`。A线程获得L锁之后申请R锁，同时B线程获得了R锁申请L锁，
由于两个两个线程调用顺序不一致导致死锁；
这种情况需要在编码时避免多个线程以不同的顺序请求相同的锁；





## 动态的锁顺序死锁
例如
```java
public voidd transferMoney(Account fromAccount, Account toAccount, BigDecimal amount){
	synchronized(fromAccount){
		synchronized(toAcccount){
			fromAccount.debit(toAcccount);
			toAcccount.credit(fromAccount);
		}
	}
}
```
看似每个线程都是按照相同的顺序获得锁，其实锁顺序是动态变化的，如果一个线程由A向B转账，
同时另一个线程由B向A转账，就会出现顺序死锁。
可以采用计算两个对象的hashCode，判断hashCode值的大小，始终保证大的/小的在前保证顺序；
如果两个恰好相等又不属于同一个对象，可定义一个额外的对象锁进行锁定从而保证锁顺序；

## 协作对象间的死锁
一般出现在持有锁A的时候调用某个外部方法，如果外部方法恰好持有锁B并且会请求锁A，
就会出现死锁。
```java
/**
 * CooperatingDeadlock
 * <p/>
 * Lock-ordering deadlock between cooperating objects
 *
 * @author Brian Goetz and Tim Peierls
 */
@Evaluated(">_<")
public class CooperatingDeadlock {
    // Warning: deadlock-prone!
    class Taxi {
        @GuardedBy("this")
        private Point location, destination;
        private final Dispatcher dispatcher;

        public Taxi(Dispatcher dispatcher) {
            this.dispatcher = dispatcher;
        }

        public synchronized Point getLocation() {
            return location;
        }

        public synchronized void setLocation(Point location) {
            this.location = location;
			//持有锁Taxi并调用外部方法notifyAvailable()申请锁Dispatcher
            if (location.equals( destination ))
                dispatcher.notifyAvailable( this );
        }

        public synchronized Point getDestination() {
            return destination;
        }

        public synchronized void setDestination(Point destination) {
            this.destination = destination;
        }
    }

    class Dispatcher {
        @GuardedBy("this")
        private final Set< Taxi > taxis;
        @GuardedBy("this")
        private final Set< Taxi > availableTaxis;

        public Dispatcher() {
            taxis = new HashSet< Taxi >();
            availableTaxis = new HashSet< Taxi >();
        }

        public synchronized void notifyAvailable(Taxi taxi) {
            availableTaxis.add( taxi );
        }

        public synchronized Image getImage() {
            Image image = new Image();
			//持有锁Dispatcher并调用外部方法t.getLocation()申请锁Taxi
            for (Taxi t : taxis)
                image.drawMarker( t.getLocation() );
            return image;
        }
    }

    class Image {
        public void drawMarker(Point p) {
        }
    }

    interface Point {

    }
}
```
解决办法是将同步方法改为同步代码块，避免在持有锁时调用外部方法

---
引用：《Java并发编程实战》





