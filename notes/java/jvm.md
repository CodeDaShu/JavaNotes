
<!-- TOC -->
- [一、运行时内存区域](#一运行时内存区域)
  - [程序计数器](#程序计数器)
  - [栈](#栈)
  - [堆](#堆)
  - [方法区](#方法区)
  - [运行时常量池](#运行时常量池)
  - [直接内存](#直接内存)
  - [线程隔离和线程共享](#线程隔离和线程共享)
- [二、垃圾回收与算法](#二垃圾回收与算法)
- [三、引用类型](#三引用类型)
- [四、类加载机制](#四类加载机制)
- [五、内存溢出和内存泄漏](#五内存溢出和内存泄漏)
<!-- TOC -->


# 一、运行时内存区域

JVM 运行时数据区域大致可以分为：程序计数器、虚拟机栈、本地方法栈、堆区、元空间、运行时常量池、直接内存等区域；就是下面这个样子的：

![Java 运行时数据区域](https://github.com/CodeDaShu/JavaNotes/blob/master/img/JVM/JVM-1.jpg)

其中有些区域，随着 JDK 版本的升级不断调整，例如：

*   JDK 1.6，字符串常量池位于永久代的运行时常量池中；
*   JDK 1.7，字符串常量池从永久代剥离，放入了堆中；
*   JDK 1.8，元空间取代了永久代，并且放入了本地内存（Native memory）中。

## 程序计数器

## 栈

## 堆

## 方法区

## 运行时常量池

## 直接内存

## 线程隔离和线程共享

### 线程私有

![线程私有](https://github.com/CodeDaShu/JavaNotes/blob/master/img/JVM/JVM-private.jpg)

#### 1. 程序计数器

一个 CPU 在某个时间点，只能做一件事情，在多线程的情况下，CPU 运行时间被划分成若干个时间片，分配给各个线程执行；
程序计数器的作用就是记录当前线程执行的位置，当线程被切换回来的时候，能够找到该线程上次运行到哪儿了；所以程序计数器一定是线程隔离的。

#### 2. 虚拟机栈和本地方法栈

*   虚拟机栈：每个 Java 方法在执行的同时，会创建一个栈帧，用于存储局部变量表、操作数栈、常量池引用等信息；方法的调用过程，就是一个栈帧在 Java 虚拟机栈中入栈和出栈的过程；
*   本地方法栈：和虚拟机栈很类似，区别在于虚拟机栈为 Java 方法服务，本地方法栈为 Native 方法服务；其中 Native 方法可以看做用其它语言（C、C++ 或汇编语言等）编写的方法；
*   HotSpot 虚拟机就选择了将虚拟机栈和本地方法栈合并在了一起；

为了保证线程中的局部变量不被别的线程访问到，所以虚拟机栈和本地方法栈是线程隔离的。

### 线程公有

![线程公有](https://github.com/CodeDaShu/JavaNotes/blob/master/img/JVM/JVM-public.jpg)

#### 1. 堆区
对于堆栈的区别总结一句话：堆中存对象，栈中存基本数据类型和堆中对象的引用；一个对象的大小是可以动态变化的，而引用是固定大小的。
这么看就容易理解堆为什么是线程公有的了，省地儿啊。

#### 2. 元空间区/方法区 
方法区用于存放已被加载的类信息、常量、静态变量、即编译器编译后的代码等。
还有要注意的一点：方法区是 JVM 的规范，在 JDK 1.8 之前，方法区的实现是永久代；从 JDK 1.8 开始 JVM 移除了永久代，使用本地内存来存储元数据并称之为：元空间（Metaspace）。

#### 3. 运行时常量池 
Class 文件中的常量池，会在类加载后被放入这个区域。
另外在 JDK 1.7 之前，字符串常量池就在运行时常量池中，后来字符串常量池放入了堆中，而运行时常量池仍然在方法区（元空间区）中。
有兴趣的朋友可以自己测试一下，以死循环方式创建字符串常量，在 JDK 1.6 里会报永久代 OOM ；在 JDK 1.7 里会报堆区 OOM 。 

#### 4. 直接内存 
也叫做堆外内存，并不是虚拟机运行时数据区的一部分，也不是Java 虚拟机规范中定义的内存区域。
JDK 1.4 加入的 NIO 类，引入了一种基于通道 ( Channel ) 与缓冲区 ( Buffer ) 的 I/O 方式，它可以使用 native 函数库直接分配堆外内存，然后通过堆上的DirectByteBuffer对象对这块内存进行引用和操作。
简单来说，直接内存就是 JVM 内存之外有一块内存区域，我们通过堆上的一个对象可以操作它；具体等讲到 NIO 部分的时候，再回来加深理解。

# 二、垃圾回收与算法

# 三、引用类型

# 四、类加载机制

# 五、内存溢出和内存泄漏

JAVA中的内存溢出和内存泄露分别是什么，有什么联系和区别，让我们来看一看。

## 内存泄漏 & 内存溢出

1. 内存泄漏（memory leak ）

申请了内存用完了不释放，比如一共有 1024M 的内存，分配了 521M 的内存一直不回收，那么可以用的内存只有 521M 了，仿佛泄露掉了一部分；

通俗一点讲的话，内存泄漏就是【占着茅坑不拉shi】。


2. 内存溢出（out of memory）

申请内存时，没有足够的内存可以使用；

通俗一点儿讲，一个厕所就三个坑，有两个站着茅坑不走的，剩下最后一个表示压力很大，这就是内存泄漏，这时候一下子来了两个人，坑位（内存）就不够了，内存泄漏变成内存溢出了。

可见，内存泄漏和内存溢出的关系：内存泄露的增多，最终会导致内存溢出。

这是一个很有味道的例子。

![OOM](https://github.com/CodeDaShu/JavaNotes/blob/master/img/JVM/OOM.jpg)

如上图：

- 对象 X 引用对象 Y，X 的生命周期比 Y 的生命周期长；
- 那么当Y生命周期结束的时候，X依然引用着Y，这时候，垃圾回收期是不会回收对象Y的；
- 如果对象X还引用着生命周期比较短的A、B、C，对象A又引用着对象a、b、c，这样就可能造成大量无用的对象不能被回收，进而占据了内存资源，造成内存泄漏，直到内存溢出。

## 泄漏按照发生频率的分类

- 经常发生：发生内存泄露的代码会被多次执行，每次执行，泄露一块内存；
- 偶然发生：在某些特定情况下才会发生；
- 一次性：发生内存泄露的方法只会执行一次；
- 隐式泄露：一直占着内存不释放，直到执行结束；严格的说这个不算内存泄露，因为最终释放掉了，但是如果执行时间特别长，也可能会导致内存耗尽。

## 导致内存泄露的常见原因

1. 循环过多或死循环，产生大量对象；

2. 静态集合类引起内存泄漏，因为静态集合的生命周期和 JVM 一致，所以静态集合引用的对象不能被释放；下面这个例子中，list 是静态的，只要 JVM 不停，那么 obj 也一直不会释放。

```Java
public class OOM {
 static List list = new ArrayList();

 public void oomTests(){
  Object obj = new Object();

  list.add(obj);
 }
}
```

3. 单例模式，和静态集合导致内存泄露的原因类似，因为单例的静态特性，它的生命周期和 JVM 的生命周期一样长，所以如果单例对象如果持有外部对象的引用，那么这个外部对象也不会被回收，那么就会造成内存泄漏。

4. 数据连接、IO、Socket连接等等，它们必须显示释放（用代码 close 掉），否则不会被 GC 回收。

```Java
try {
 Connection conn = null;
 Class.forName("com.mysql.jdbc.Driver");
 conn = DriverManager.getConnection("url","", "");
 Statement stmt = conn.createStatement() ;
 ResultSet rs = stmt.executeQuery("....") ;
} catch (Exception e) {
 //异常日志
} finally {
 //1.关闭结果集 Statement
 //2.关闭声明的对象 ResultSet
 //3.关闭连接 Connection
}
```

5. 内部类的对象被长期持有，那么内部类对象所属的外部类对象也不会被回收。

6. Hash 值发生改变，比如下面中的这个类，它的 hashCode 会随着变量 x 的变化而变化：

```Java
public class ChangeHashCode {
 private int x ;

 @Override
 public int hashCode() {
  final int prime = 31;
  int result = 1;
  result = prime * result + x;
  return result;
 }

 @Override
 public boolean equals(Object obj) {
  if (this == obj)
   return true;
  if (obj == null)
   return false;
  if (getClass() != obj.getClass())
   return false;
  ChangeHashCode other = (ChangeHashCode) obj;
  if (x != other.x)
   return false;
  return true;
 }
 //省略 set 、get 方法
}
```

```Java
public class HashSetTests {
 public static void main(String[] args){
  HashSet<ChangeHashCode> hs = new HashSet<ChangeHashCode>();

  ChangeHashCode cc = new ChangeHashCode();
  cc.setX(10);//hashCode = 41
  hs.add(cc);

  cc.setX(20);//hashCode = 51
  System.out.println("hs.remove = " + hs.remove(cc));//false

  hs.add(cc);
  System.out.println("hs.size = " + hs.size());//size = 2

 }
}
```

可以看到，在测试方法中，当元素的 hashCode 发生改变之后，就再也找不到改变之前的那个元素了；

这也是 String 为什么被设置成了不可变类型，我们可以放心地把 String 存入 HashSet，或者把 String 当做 HashMap 的 key 值；

当我们想把自己定义的类保存到散列表的时候，需要保证对象的 hashCode 不可变。

7. 内存中加载数据量过大；之前项目在一次上线的时候，应用启动奇慢直到夯死，就是因为代码中会加载一个表中的数据到缓存（内存）中，测试环境只有几百条数据，但是生产环境有几百万的数据。
