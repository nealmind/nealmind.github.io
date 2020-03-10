---
layout: post
title: Java面试题（基础篇）
tags: 面试题
author: neal

---
* content
{:toc}

## Java面试题总结

**Java基础**

* **JAVA中的几种基本数据类型是什么，各自占用多少字节。**

8种基本数据类型，4个整型，两个浮点类型，一个字符类型，一个布尔类型；

整型：

byte：一个字节，8位；

short： 2个字节，16位；

int： 4个字节，32位；

long： 8个字节，64位；









浮点类型：

float： 4个字节，32位；

double： 8个字节，64位；

char： 2个字节，16位；boolean；

* **String类能被继承吗？为什么？**

不能，String类是final类型的不能被继承；String类被设计为Immutable，同样的还有Integer和Long等；

* **String， StringBuffer ，StringBuilder的区别。**

String类是不可变的，StringBuffer提供可变的字符串拼接，StringBuilder和StringBuffer功能类似；

StringBuffer是线程安全的类，其append方法用synchronized修饰，并且维护toStringCache字段保存每次toString时char数组中的值；

StringBuilder是线程不安全的类；

字符串的拼接相当于StringBuffer的append；

* **ArrayList和LinkedList有什么区别。**

ArrayList是基于数组实现的，基于下标访问，随机访问的复杂度是O(1);

LinkedList是基于双向链表实现的，每个节点都维护一个prev和next字段指向前后节点，随机访问复杂度是O(n)；

在插入数据时，ArrayList需要检查数组容量，必要时进行扩容（原来容积的1/2，最大为Integer.MAX_VALUE-8）;

另外插入时需要进行数组copy；LinkedList直接修改前后节点的指针即可；

（推荐使用Array List，James Gosling在Stack Overflow提到自己很少用LinkedList >_<）

* **讲讲类的实例化顺序，比如父类静态数据，构造函数，字段，子类静态数据，构造函数，字
  段，当new的时候，他们的执行顺序。**

当new一个类时，初始化顺序为：静态变量>静态代码块>普通变量；

同时，在初始化一个类时，如果父类没有初始化，优先初始化父类；

静态代码块及静态变量只初始化一次；

* **用过哪些Map类，都有什么区别，HashMap是线程安全的吗,并发下使用的Map是什么，他们
  内部原理分别是什么，比如存储方式，hashcode，扩容，默认容量等。**

HashMap, HashTable, LinkedHashMap, TreeMap, ConcurrentHashMap等；

区别：HashMap多线程下不保证线程不安全，并且key和value允许使用null；LinkedHashMap继承了HashMap，在HashMap基础上维护了一个双向链表保存HashMap的Node节点顺序。

ConcurrentHashMap和HashTable可以保证多线程环境下线程安全；区别是实现方式不同，HashTable是使用synchronized锁定整个hash表来实现读，写同步，ConcurrentHashMap是基于synchronized和cas实现同步（1.8），且锁粒度上要更细，只锁定头节点，并且读操作没有锁定；

* **JAVA8的ConcurrentHashMap为什么放弃了分段锁，有什么问题吗，如果你来设计，你如何
  设计。**

使用了CAS+synchronized实现同步，由于使用了数组+链表/红黑树的数据结构，在锁定粒度上能够更细化，只要锁定链表头节点/红黑书根节点就可以；另外synchronized在JVM层面在不断优化，使用synchronized可以保持优化能够同步可控；

* **有没有有顺序的Map实现类，如果有，他们是怎么保证有序的。**

TreeMap和LinkedHashMap是可以保证有序的Map实现类；区别是TreeMap实现了SortedMap，可以根据key值进行排序，默认是按key升序排列；

LinkedHashMap维护了一个双向链表，可以根据数据的插入和获取进行排序（根据accessOrder的值判断，默认为false），最近被访问/操作的数据放在链表尾部；

* **抽象类和接口的区别，类可以继承多个类么，接口可以继承多个接口么,类可以实现多个接口
  么。**

- 抽象类和接口都不能直接实例化，必须通过实现了所有接口方法/抽象方法的实现类类进行实例化；
- 抽象类是继承，接口是实现；Java允许单继承，多实现；
- 抽象类可以有构造函数，抽象方法只声明不实现；接口不能有构造函数，可以借助default关键字默认实现方法；

* **继承和聚合的区别在哪。**

- 继承通过extends关键字，继承另一个类的功能，可以添加相应扩展；类与类，接口与接口之间可以继承；
- 聚合是两个独立的类之间的整体与部分的关系，即has-a关系；
- 继承的特点是子类自动继承父类接口，缺点是子类与父类耦合性较强，破坏了封装性；聚合正好相反；

* **IO模型有哪些，讲讲你理解的nio ，他和bio，aio的区别是啥，谈谈reactor模型。**

常见的IO模型有：

blocking(阻塞IO)；non-blocking(非阻塞IO)；asynchronous(异步IO)；multiplexing(多路复用IO)；

一般的IO请求会经过几个阶段：

- 第一阶段为准备数据，把数据copy到内核缓冲空间
- 第二阶段为把内核空间的数据copy到应用进程的缓冲空间；

**bio**是在数据准备的两个阶段都阻塞进程，直到内核完成copy数据到应用进程缓冲空间并返回结果后才唤起进程进行相应操作；

**多路复用io**又叫信号驱动io，是基于select，poll，epoll实现的。当用户进程调用了select函数，整个进程就会被阻塞，同时内核进程会轮询监视所有调用了select的进程socket，当任何一个socket准备好数据了，select就会返回，同时用户进程再调用read请求从内核copy数据到用户进程空间，之后继续阻塞，直到copy数据结束并返回，用户进程接下来执行相应的操作；

  多路复用io会阻塞两次，copy数据到内核缓冲区，内核copy数据到应用进程缓冲区都会阻塞，好处是一个进程可以同时处理多个网络连接的请求；

**nio** 是指用户进程调用read请求数据，如果数据还没有准备好，会马上返回一个error，用户轮询请求，直到得到数据准备完成的返回结果，和多路复用io不同的是由用户进程负责轮询；

**aio** 是指用户进程发起read请求数据后，会立即得到返回，之后用户进程继续执行其他操作不必等到数据完全准备好，当数据copy完成是，会收到一个signal信号，之后再进行数据相关的操作；

- **反射的原理，反射创建类实例的三种方式是什么。**

反射简单来说就是在运行时获取类的属性和方法并进行赋值或调用；

一般来说想要使用一个类，需要先进行初始化，即

```
A a = new A();
a.methodA();
```

要使用一个类，就需要获取该类的Class对象；

通过反射有三种获取方式：

```
//1,通过Class.forName("类全限定名")获取
Class c1 = Class.forName("com.xxx.A");
//2.通过.class获取
Class c2 = A.clas;
//3.通过类对象.getClass()获取
A a = new A();
Class c3 = a.getClass();

//通过类对象操作类属性或方法有两种方式
//1.通过newInstance()方法直接实例化
Class clazz = Class.forName("com.xxx.A");
A a = (A)clazz.newInstance();
//2.通过构造函数实例化
Constructor cons = clazz.getConstructor();
A a = (A)cons.newInstance();

//获取类的属性及方法
Method[] declared = clazz.getDeclaredMethods();		//获取自定义方法

```

拿到了类对象，就可以调用类的方法或操作属性值；

- **反射中，Class.forName和ClassLoader区别 。**

Class.forName()和ClassLoader都会将.class文件加载到jvm中，区别是前者加载后还会对类进行解释，执行类的static代码块，后者只有在调用了newInstance()后才会执行static代码块；

- **动态代理的几种实现方式，分别说出相应优缺点；**

动态代理：是使用反射和字节码的技术，在运行期创建指定接口或类的子类，以及其实例对象的技术，通过这个技术可以无侵入的为代码进行增强;

动态代理有JDK动态代理和CGLIB动态代理；

JDK动态代理：是使用反射实现，不需要引用第三方库，在运行时生成代理类，效率较高；缺点是只能针对接口(生成的代理类继承了Proxy，由于Java是单继承，想要跟被代理类建立联系，只能针对接口进行代理，即，生成的代理类：$Proxy extends Proxy implements TargetInterface)；

CGLIB：是基于ASM字节码生成库，通过继承的方式对字节码进行修改和动态生成，无论目标对象有没有实现接口都可以；缺点是无法处理final修饰的类；

通常来说，基于接口的一般使用JDK动态代理，基于类的使用CGLIB(类不能用final修饰)；

JDK动态代理基于反射，所以生成效率较快；cglib基于字节码技术，所以执行效率较快；

- **动态代理与cglib实现的区别。**

区别同上；

- **为什么CGlib方式可以对接口实现代理。**

CGLIB使用ASM字节码技术生成新的子类，基于继承；

- **final的用途。**

final修饰的类不能够被继承，final修饰的方法不能被重写，final修饰的变量引用不可变；

- **写出三种单例模式实现 。**

单线程下使用，懒加载：

```
/**
 * @date 2019/12/23 21:29
 * 
 * 线程不安全，懒加载
 */
public class LazySingleton {
    private static LazySingleton instance;

    private LazySingleton(){
    }

    public static LazySingleton getInstance(){
        if(instance==null){
            instance=new LazySingleton();
        }
        return instance;
    }
}

```

二、静态内部类形式：懒加载，线程安全

```
/**
 * @date 2019/12/23 21:37
 * 静态内部类模式，线程安全，懒加载（推荐）
 */
public class InnerClassSingleton {

    private InnerClassSingleton() {
    }

    public static InnerClassSingleton getInstance() {
        return SingletonHolder.INSTANCE;
    }

    private static class SingletonHolder {
        private static final InnerClassSingleton INSTANCE = new 	InnerClassSingleton();
    }
}

```

三、基于static关键字初始化，线程安全，非懒加载

```
/**
 * @date 2019/12/23 21:12
 * 基于static关键字初始化，线程安全，非懒加载
 */
public class Singleton {
    private static Singleton instance = new Singleton();

    private Singleton() {
        System.err.println("create a new instance.");
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

四、Double-Check-Locking 双重锁检查模式，线程安全，懒加载

```
/**
 * @author Created by neal.zhang
 * @date 2020/2/14 17:54
 * 双重锁检查，jdk1.5以上适用，基于synchronized和volatile
 * 线程安全，懒加载，有人认为anti-pattern
 */
public class DoubleCheckSingleton {

    private static volatile DoubleCheckSingleton instance;

    private DoubleCheckSingleton() {
    }

    public static DoubleCheckSingleton getInstance() {
        if (instance == null) {
            synchronized (DoubleCheckSingleton.class) {
                if (instance == null) {
                    instance = new DoubleCheckSingleton();
                }
            }

        }
        return instance;
    }
}
```

五、枚举，天然immutable，天然多线程安全；

其他的各种不推荐；

- **如何在父类中为子类自动完成所有的hashcode和equals实现？这么做有何优劣。**

子类不实现，默认调用父类方法；

缺点是不精确，有需要比较子类的场景则不适用；

- **请结合OO设计理念，谈谈访问修饰符public、private、protected、default在应用设
  计中的作用。**

- `public` 所有类都可以访问
- `protected` 子类可以访问
- `private` 只有自己可以访问
- `default` 默认访问模式，同一包下可以访问

- **深拷贝和浅拷贝区别。**

浅拷贝是指只拷贝对象本身和包含的基本类型属性，而不拷贝引用类型属性；

深拷贝是指不仅拷贝对象本身，还拷贝其包含的所有引用指向的对象；

- **数组和链表数据结构描述，各自的时间复杂度。**

- 数组是长度分配后固定的连续的内存空间，每个元素占用空间相等，可通过下标进行快速访问，时间复杂度O (1) ；在中间插入/删除数据需要移动大量元素，时间复杂度O(n)；
- 链表是长度不固定的非连续存储的内存空间，各个元素通过指针链接，插入/删除只需要改变前后节点的指针指向，不需要移动大量元素，时间复杂度O(1)；访问时需要可能需要遍历多个节点才能够找到需要的数据，时间复杂度O(n)；

- **error和exception的区别，CheckedException，RuntimeException的区别。**

- Error一般为虚拟机非正常运行，无法捕获和处理，出现Error程序应停止运行；
- Exception一般为程序可以处理的异常，可以捕获处理并恢复，不需要停止运行；

CheckedException和RuntimeException都是Exception的子类：

* CheckedException是指在编译期如果不做处理（throws或try-catch），无法通过编译的异常；

- RuntimeException是指运行时异常，可以通过编译，一般为程序逻辑错误，应该在编码阶段尽量避免；

- **请列出5个运行时异常。**

NullPointerException（空指针）,ArrayOutOfBoundsException（数组越界）,ClassCastException（类型转换）,ArithmeticException（运算异常）,DateTimeException（日期异常）；

- **在自己的代码中，如果创建一个java.lang.String类，这个类是否可以被类加载器加
  载？为什么。**

不能，java.*的类jvm决定了必须由Bootstrap类加载器进行加载，防止用户自定义的类替换java核心api定义的类，避免安全隐患；用户自定义的类加载器是AppClassLoader，由于jvm类加载器采用双亲委派机制进行加载，会先尝试由父加载器（ExtClassLoader）进行加载，以此类推直到Bootstrap类加载器，由于已经加载过api中定义的String，直接返回；

- **说一说你对java.lang.Object对象中hashCode和equals方法的理解。在什么场景下需
  要重新实现这两个方法。**

hashCode返回一个整数值，一般用于hash table中进行快速确定对象的存储位置；如果两个不同的对象hashCode相等，就存在了hash冲突；

equals用==来比较，比较的是两个对象的内存地址；

当需要将对象存储到哈希表中，出于业务需求，在比较某些属性相同后就判断为同一个对象，则需要重写hashCode和equals方法，因为哈希表一般会首先根据hash值进行判断是否存在，用equals方法判断是否hash冲突；

