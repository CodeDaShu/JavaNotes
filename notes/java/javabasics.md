<!-- TOC -->
- [一、数据类型](#一数据类型)
  - [基本数据类型](#基本数据类型)
  - [类型转换](#类型转换)
  - [缓存池机制](#缓存池机制)
- [二、String](#二String)
  - [String 的不可变性](#String的不可变性)
- [三、关键字](#三关键字)
  - [static](#static)
  - [final](#final)
- [四、继承](#四继承)
- [五、异常](#五异常)    
  - [异常的分类](#异常的分类)
  - [异常的处理方式](#异常的处理方式)
- [六、反射](#六反射)
- [七、注解](#七注解)
  - [注解的概念及作用](#注解的概念及作用)
  - [元注解](#元注解)
  - [内置注解](#内置注解)
  - [注解的使用场景](#注解的使用场景)
- [八、内部类](#八内部类)
  - [静态内部类](#静态内部类)
  - [成员内部类](#成员内部类)
  - [匿名内部类](#匿名内部类)
  - [局部内部类](#局部内部类)
  - [为什么使用内部类](#为什么使用内部类)
- [九、泛型](#九泛型)
  - [泛型的概念及好处](#泛型的概念及好处)
  - [泛型的使用](#泛型的使用)
  - [泛型类](#泛型类)
  - [泛型方法](#泛型方法)
  - [类型通配符](#类型通配符)
- [十、序列化](#十序列化)
  - [Java序列化的概念及作用](#Java序列化的概念及作用)
  - [序列化实现的方式](#序列化实现的方式)
  - [序列化版本](#序列化版本)
<!-- TOC -->

# 一、基本数据类型

## 基本数据类型

Java 中一共有 8 种基本数据类型：

![Java 基本类型](https://github.com/CodeDaShu/JavaNotes/blob/master/img/Java/java_type.jpg)

其中 boolean 比较特殊，在 java 规范中给出了 boolean 当做 int 处理（4 byte），boolean 数组用 byte 数组实现（1 byte）的定义，具体还要看虚拟机实现是否按照规范实现。

装箱与拆箱：基本类型与其对应的包装类型之间自动进行转换。

```Java
Integer x = 1; // 装箱：基本类型转包装类型，1 是 int 类型，调用了 Integer.valueOf(1)
int y = x;   // 拆箱：包装类型转基本类型，调用了 x.intValue()
```
## 类型转换

### 自动类型转换
由低字节向高字节自动转换；黑线表示无数据丢失，红线表示可能发生精度丢失。

![Java 类型转换](https://github.com/CodeDaShu/JavaNotes/blob/master/img/Java/java_type_transform.jpg)

### 强制数据转换
由高字节向低字节转换，存在精度损失的风险，需要在代码中强制转换。

```Java
int n = (int)56.56
```
### 类型提升
操作不同数据类型，会自动向字节更大的数据类型提升。

- 所有的byte,short,char型的值将被提升为int型；
- 有一个操作数是long型，计算结果是long型；
- 有一个操作数是float型，计算结果是float型；
- 有一个操作数是double型，计算结果是double型。

```Java
char ch = 'a';
int n = 10 ;
n = ch + n ; //107；转成了 int

long l = 100000000L ;
l = l + n ;  //100000107；转成了 long
```

### 隐式类型转换
让我们看看这几行代码：

```Java
char ch = 'a';
ch = ch + 1 ;
ch ++ ;
```
**ch = ch + 1 ：** 因为 1 是 int 类型，ch + 1 会转成更高范围的 int ，所以这里编译会报错，cannot convert from int to char
**ch ++ ：** 会正常编译执行，结果是 'b'，因为这里有个隐式类型转换，相当于 ch = (short) (ch + 1)

## 缓存池机制

关于 Java 字符串 String 有一道很基础的面试题，相信很多人都遇到过，就是 String s = "a" 和 String s = new String("a") 的区别是什么？相信大家都能回答上来。
那么你知道这三者有什么区别么？

```Java
Integer i = new Integer(1) ;
Integer i = Integer.valueOf(1) ;
Integer i = 1 ;
```

### new Integer(1) 与 Integer.valueOf(1)
- new Integer(1) ：会新建一个对象；
- Integer.valueOf(1) ：使用对象池中的对象，如果多次调用，会取得同一个对象的引用。

### 缓存池的概念
为了提高性能，Java 在 1.5 以后针对八种基本类型的包装类，提供了和 String 类一样的缓存池机制；让我们看一下 Integer.valueOf(int i) 的源码，就很容易理解了：

```Java
public final class Integer extends Number implements Comparable<Integer> {
  public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
}
```

- Integer.valueOf() 中有个内部类 IntegerCache，类似于一个常量数组，也叫缓存池，它维护了一个 Integer 数组 cache，长度为（128+127+1）=256，意味着 Integer 缓存池的大小默认为 -128 ~ 127 ；
- Integer类中还有一个静态代码块，默认创建了数值【-128-127】的 Integer 缓存数据；所以当 Integer.valueOf(1) 的时候，会直接在该在对象池找到该值的引用；
- 在 jdk 1.8 中，在启动 JVM 的时候，可以通过配置来指定这个缓存池的大小。

### Integer i = 1 与 Integer.valueOf(1)

```Java
Integer i = 1 ;
```

等号左边是 Integer 类型，等号右边是 int 类型 ，这种写法叫做装箱（基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成），而装箱操作是通过 Integer.valueOf(1) 完成的，所以：

Integer i = 1 等同于 Integer.valueOf(1)

### 其他基本类型对应的缓存池
- Boolean：true , false
- Short, Int, Long：-128 ~ 127
- Byte, Character : \u0000 到 \u007F，也就是 0 ~ 127

# 二、String

## String 的不可变性
String 被声明为 final，是不可变的，它也不可被继承。

### 通过源码了解 String 的不可变性

在 Java 8 中，String 是使用 char 数组实现的。

```Java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
   /** The value is used for character storage. */
   private final char value[];

   /** Cache the hash code for the string */
   private int hash; // Default to 0
}
```

到了 Java 9 ，String 类的实现改用 byte 数组实现，同时使用 coder 来标识使用了哪种编码。

```Java
public final class String
   implements java.io.Serializable, Comparable<String>, CharSequence {
          /** The value is used for 	character storage. */
          private final byte[] value;
    /** The identifier of the encoding used to encode the bytes in {@code value}. */
          private final byte coder;
}
```

我们可以看到，value 数组被声明为 final，并且 String 内部也没有改变 value 数组的方法，因此 String 是不可变的。

### 不可变的优势

**1. 线程安全**

因为 String 的不可变性，所以在多线程的环境下，可以被安全使用。

**2. hash 值不需要重复计算**

一个 String 被创建后，那么它的 hash 值只需要计算一次；比如使用 String 做为 HashMap 中的 key ，那么不需要重复计算 hash 值。

**3. String Pool：**

一个 String 被创建后，被可以放入 String Pool 中，就可以被其他地方所引用；因为 String 是不可变的，所以才能实现 String Pool 。

### String, StringBuffer, StringBuilder

**1. 不可变性**
- String 不可变
- StringBuffer 和 StringBuilder 可变

**2. 线程安全**
- String 不可变，因此是线程安全的
- StringBuilder 不是线程安全的
- StringBuffer 是线程安全的，内部使用 synchronized 进行同步


# 三、关键字

## static

### 静态方法

被 static 修饰的方法被称作静态方法，它可以不依赖于任何对象进行访问（通过类名访问），在静态方法中不能访问非静态的变量和方法。
静态方法在类加载的时候就存在了，静态方法不能是抽象方法。

### 静态变量

被 static 修饰的变量被称作静态变量，它在内存中只存在一份，被所有对象共享。

```Java
public static int n = 1;
```

JVM 类加载到准备阶段的时候，会初始化静态变量，也就是赋值成默认值，这里 n 会赋值为 0 ；当 JVM 加载进行到初始化阶段的时候，再讲静态变量赋值成指定的值。
```Java
public static final int n = 1;
```

如果同时被 static 和 final 修饰，那么叫做静态常量，JVM 类加载到准备阶段的时候，就会赋值成指定的值。

### 静态代码块

静态代码块可以放在类中的任何地方，一个类中可以有多个静态代码块；在类加载的时候按照顺序执行静态代码块，并且只会执行一次。

### 静态内部类

[匿名内部类](#匿名内部类)

### 初始化顺序

- 父类（静态变量/代码块）
- 子类（静态变量/代码块）
- 父类（实例变量/代码块）
- 父类（构造方法）
- 子类（实例变量/代码块）
- 子类（构造方法）

静态变量和静态代码块的初始化顺序，由它们在代码中的顺序决定。

### 静态导包

通过静态导包，在使用静态变量和方法的时候，不需要在带 [ClassNam.]，可以简化代码。

```Java
import static com.dashu.javanotes.sta.Contants.*;

public class StaticImport {
	public void say() {
		System.out.println(Contants.MAN);
	}

	public void speak() {
		System.out.println(MAN);//静态导包
	}
}
```

## final

### 修饰类

当 final 修饰一个类的时候，这个类不能被继承，并且类中所有的属性和方法默认是 final 的。

### 修饰方法

当 final 修饰一个方法的时候，这个方法不能被重写；private 私有方法隐式地被认为是 final 的。

使用 final 修饰的方法，执行效率可能会稍微高一些：

当我们在 A 方法中调用 B 方法时， JVM 会先保存 A 方法当前的数据和执行地址，然后跳到 B 方法所在的内存地址执行，执行完后再返回 A 方法原来的位置；如果 B 方法被 final 修饰，那么 JVM 就可能会进行内联优化。

比如：

```Java
public int addition(int x, int y) {
 return x + y;
}

addition.(1, 2) + addition.(3, 4) ;
```

就可以被优化成

```Java
1 + 2 + 3 + 4 ;
```

对了，上文中说 [ 那么 JVM 就可能会进行内联优化 ] ，为什么是 **“可能”**？因为我们只能告诉 JVM 这里可以做内联优化，但是最终 JVM 还是要自己决定是否做内联优化。


### 修饰变量

如果变量被 final 修饰，它就成了常量；

final 修饰基本类型变量的之后，赋值之后无法改变：

```Java
final int i = 1 ;
i = 2 ; // 编译报错：The final local variable i cannot be assigned.
```

final 修饰引用类型，可以使其在初始化之后，不能再指向其他对象，但是它指向的内容是可以改变的：

```Java
final User user1 = new User("Suku","M","30");
User user2 = new User("Alina","F","20");
user1 = user2; //报错：The final local variable user1 cannot be assigned.
user1.setGender("F");  //可以修改
```

# 四、继承


# 五、异常


如果 Java 方法不能按照正常的流程执行，那么可以通过另外一种途径退出：抛出一个封装了错误信息的对象，这个就是 Java 的异常；当发生异常时，后面的代码无法继续执行，而是由异常处理器继续执行。

![异常](https://github.com/CodeDaShu/JavaNotes/blob/master/img/Java/exception.jpg)

## 异常的分类

Throwable 是所有异常的超类，下一级可以分为 Error 和 Exception ：

1. Error

Error 是指 Java 运行时系统内部的错误，或者说它代表了 JVM 本身的错误，通常都是比较严重的错误， 比如内存溢出, 虚拟机错误等等；

Error 通常和硬件或 JVM 有关，和程序本身无关，所以不能被代码捕获和处理。

2. Exception

我们经常说的异常时指 Exception，又可以分成 RuntimeException 和 CheckedException。

RuntimeException：运行时异常，这类异常在编译期间不强制代码捕捉，但是可能在在 JVM 运行期间抛出异常；出现此类异常，通常是代码的问题，所以需要修改程序避免这类异常。常见的运行时异常，比如：NullPointerException、ClassCastException 等等。

CheckedException：检查异常，这种异常发生在编译阶段，Java 编译器会强制代码去捕获和处理此类异常；比如：ClassNotFoundException、IllegalAccessException 等等。


## 异常的处理方法

1.捕获异常使用 try...catch 语句，把可能发生异常的代码放到 try {...} 中，然后使用 catch 捕获对应的异常；

2.我们也可以在代码块中使用 Throw 向上级代码抛出异常；

3.在方法中使用 throws 关键字，向上级代码抛出异常；

## Throw 和 throws 的区别

1.Throw 在方法内，后面跟着异常对象；而 throws 是用在方法上，后面跟异常类；

2.Throw 会抛出具体的异常对象，当执行到 Throw 的时候，方法内的代码也就执行结束了；throws 用来声明异常，提醒调用方这个方法可能会出现这种异常，请做好处理的准备，但是不一定会真的出现异常。

3.Throw 和 throws 都是消极的处理异常的方法，因为它们只是抛出异常，而不会真正处理异常。

## Exception 的一些建议

1.不要试图通过异常来控制程序流程，比如开发一个接口，正确的做法是对入参进行非空验证，当参数为空的时候返回“参数不允许为空”，而不应该捕捉到空指针的时候返回错误提示。

2.仅捕获有必要的代码，尽量不要用一个 try...catch 包住大段甚至整个方法内所有的代码，因为这样会影响 JVM 对代码进行优化，从而带来额外的性能开销。

3.很多程序员喜欢 catch(Exception e)，其实应该尽可能地精确地指出是什么异常。

4.不要忽略异常，捕捉到异常之后千万不能什么也不做，要么在 catch{...} 中输出异常信息，要么通过 Throw 或 throws 抛出异常，让上层代码处理。

5.尽量不要在 catch{...} 中输出异常后，又向上层代码抛出异常，因为这样会输出多条异常信息，而且它们还是相同的，这样可能会产生误导。

6.不要在 finally{...} 中写 return，因为 try{...} 在执行 return 之前执行 finally{...} ，如果 finally{...} 中有 return，那么将不再执行 try{...} 中的return。


# 六、反射

在学习 Java 反射之前，先让我们看看这几个概念。

## 解释型语言和编译型语言

**解释型语言：** 不需要编译，在运行的时候逐行翻译解释；修改代码时可以直接修改，可以快速部署，不过性能上会比编译型语言稍差；比如 JavaScript、Python ；

**编译型语言：** 需要通过编译器将源代码编译成机器码才能执行；编译之后如果需要修改代码，在执行之前就需要重新编译。比如 C 语言；

Java 严格来说也是编译型语言，但又介于编译型和解释型之间；Java 不直接生成机器码而是生成中间码：编译期间，是将源码交给编译器生成 class 文件（字节码），这个过程中只做了翻译的工作，并没有把代码放入内存运行；当进入运行期，字节码才被 Java 虚拟机加载、解释成机器语言并运行。

![Java 编译-运行](https://github.com/CodeDaShu/JavaNotes/blob/master/img/Java/java_compile_run.jpg)

## 动态语言和静态语言

**动态语言：** 是指程序在运行时可以改变自身结构，在运行时确定数据类型，一个对象是否能执行某操作，只取决于它有没有对应的方法，而不在乎它是否是某种类型的对象；比如 JavaScript、Python。

**静态语言：** 相对于动态语言来说，在编译时变量的数据类型就已经确定（使用变量之前必须声明数据类型），在编译时就会进行类型是否匹配；比如 C 语言、Java ；

## 反射的概念

**Java 反射机制：** 在运行过程中，对于任意一个类，都能知道其所有的属性和方法；对于任意一个对象，都能调用其属性和方法；这种动态获取类信息和调用对象方法的功能，就是 Java 反射机制。

既然反射里面有一个“反”字，那么我们先看看何为“正”。

在 Java 中，要使用一个类中的某个方法，“正向”都是这样的：

```Java
ArrayList list = new ArrayList(); //实例化
list.add("reflection");  //执行方法
```

那么反向（反射）要如何实现？

```Java
Class clz = Class.forName("java.util.ArrayList");
Method method_add = clz.getMethod("add",Object.class);
Constructor constructor = clz.getConstructor();
Object object = constructor.newInstance();
method_add.invoke(object, "reflection");
​
Method method_get = clz.getMethod("get",int.class);
System.out.println(method_get.invoke(object, 0));
```

两段代码执行的结果是一样的，但是“正向”代码在编译前，就已经明确了要运行的类是什么（ArrayList），而第二段代码，只有在代码运行时，才知道运行的类是 java.util.ArrayList。

## 反射的作用

讲到这里，有些同学可能会有疑问：“反射有什么用？我明明都已经知道了要使用的类是 ArrayList ，我不能直接 new 一个对象然后执行里面的方法么？”

当然可以！不过很多场景中，在代码运行之前并不知道需要使用哪个类，或者说在运行的时候才决定使用哪个类；

比如有这么一个功能：“调用阿里云的人脸识别 API ”；这还不简单，参考对方的 API 文档，很快就能实现。

```Java
faceRecognition(Object faceImg){
  //调用阿里云的人脸识别 API
}
```

上线一个月后，领导说：“咱公司开始和腾讯云合作了，人脸识别的接口改一下吧”。

```Java
faceRecognition(Object faceImg){
  //调用腾讯云的人脸识别 API
}
```

修改上线运行了两个月，领导说：“换回来吧”......

当然有聪明的程序员会想到设置一个开关配置，让开关决定走哪段代码逻辑，如果领导哪天想变成亚马逊云的服务，继续写 if-else 就好了：

```Java
faceRecognition(Object faceImg){
  if("AL".equals(configStr)){
    //调用阿里云的人脸识别 API
  }else if("TX".equals(configStr)){
    //调用腾讯云的人脸识别 API
  }else if("AM".equals(configStr)){
    //调用亚马逊云的人脸识别 API
  }
}
```

不过还有一种更好的方法：

1. 定义一个接口：

```Java
interface FaceRecognitionInterface(){
  faceRecognition(Object faceImg) ;
}
```

2. 多个实现类：

```Java
class ALFaceRecognition implements FaceRecognitionInterface{
  //调用阿里云的人脸识别 API 的实现
}
​
class TXFaceRecognition implements FaceRecognitionInterface{
  //调用腾讯云的人脸识别 API 的实现
}
```

3. 在调用人脸识别功能的代码中：

```Java
String configStr = "读取配置，走阿里云还是腾讯云";
FaceRecognitionInterface faceRe =  Class.forName(configStr).newInstance();
faceRe.faceRecognition(faceImg);
```

如果上面这个例子，你依然觉得在调用方法中做 if-else 判断，和使用反射实现并没有差太多，但是如果程序员 A 提供接口，程序员 B 提供实现，程序员 C 写客户端呢？

回忆一下 JDBC 的使用，比如创建一个连接：

```Java
public Connection getConnection() throws Exception{
  Connection conn = null;
  //初始化驱动类
  Class.forName("com.mysql.jdbc.Driver");
  conn = DriverManager.getConnection("jdbc:mysql://url","root", "admin");
  return conn;
}
```

其中：
- 程序员 A 提供接口：Oracle 公司提供 JDBC 标准（接口）。
- 程序员 B 提供实现：各个数据库厂商提供针对自家数据库的实现。
- 程序员 C 写客户端：我等码农在 Java 中敲代码访问数据库。

总结一下Java 反射的作用：可以设计出更为通用和灵活的架构，很多框架为了保证其通用性，可以根据配置加载不用的类，这时候要用到反射。除此之外：
- 动态代理：在不改变目标对象方法的情况下对方法进行增强，比如使用 AOP 拦截某些方法打印日志，这就需要通过反射执行方法中的内容。
- 注解：利用反射机制，获取注解并执行对应的行为。

## 反射的用法

上文中我们知道了 Java 运行期的源文件是 class 文件（字节码），所以要使用反射，那么就需要获取到字节码文件对象，在 Java 中，获取字节码文件对象有三种方式：

- 调用某个类的 class 属性：类名.class
- 调用对象的 getClass() 方法：对象.getClass()
- 使用 Class 类中的 forName() 静态方法：Class.forName(类的全路径) ，建议使用这种方法

java.lang.reflect 类库提供了对反射的支持：

- Field ：可以使用 get 和 set 方法读取和修改对象的属性；
- Method ：可以使用 invoke() 方法调用对象中的方法；
- Constructor ：可以用 newInstance() 创建新的对象。

## 反射的优缺点

- 优点：在运行时动态获取类和对象中的内容，极大地提高系统的灵活性和扩展性；夸张一些说，反射是框架设计的灵魂。
- 缺点：会有一定的性能损耗，JVM 无法对这些代码进行优化；破坏类的封装性。

## (待完善)既然Java反射可以访问和修改私有成员变量，那封装成private还有意义么？
1. private 并不是一种安全机制
2. private 可以让一个类从外面看起来不是那么复杂


# 七、注解

## 注解的概念及作用

Annotation（注解）：先看看官方给出的概念，注解是 Java 提供的一种对元程序中元素关联信息和元数据的途径和方法；这种解释让人看的一脸懵，通俗一点说，可以把注解看做是对人对物的标签，比如：

凶猛的狮子，温顺的兔子，木讷的程序员，帅气的大叔；这里的“凶猛”、“温顺”、“木讷”、“帅气”就是对各个事物的标签；

注解也是一种标签，是对 Java 代码的一种标签，可以告诉程序员被注解的代码是用来做什么的；

当然标签的内容可能跟实际情况有所出入，甚至背道而驰，比如【木讷的程序员】，实际上就是一个不准确的标签，所以如果你在 Contrller 层的代码添加了一个 Service 的注解，同样也是一个很准确的标签。


## 元注解

元注解是注解的注解，也就是对标签的描述。比如“木讷”、“帅气”只能用在人或动物身上，那么“只能用在人或动物身上”就是对“木讷”、“帅气”这两个标签的标签；恰好元注解中就有 @Target，表示修饰对象的范围，让我们详细看一下元注解都有哪些。

- @Target：表示修饰对象的范围，注解可以作用于 packages、class、interface、方法、成员变量、枚举、方法入参等等，@Target可以指明该注解可以修饰哪些内容。
- @Retention：时间长短，也就是注解在什么时间范围之内有效，比如在源码中有效，在 class 文件中有效，在 Runtime 运行时有效。
- @Documented：表示可以被文档化，比如可以被 javadoc 或类似的工具生成文档。
- @Inherited：表示这个注解会被继承，比如 @MyAnnotation 被 @Inherited 修饰，那么当 @MyAnnotation 作用于一个 class 时，这个 class 的子类也会被 @MyAnnotation 作用。

## 内置注解

Java 中内置了三种注解：

- @Override：检查该方法是否是重载方法；如果父类或实现的接口中，如果没有该方法，编译会报错。
- @Deprecated：已经过时的方法；如果使用该方法，会有警告提醒。
- @SuppressWarnings：忽略警告；比如使用了一个过时的方法会有警告提醒，可以为调用的方法增加 @SuppressWarnings 注解，这样编译器不在产生警告。

JDK 7之后，又增加了三种：

- @SafeVarargs：是一个参数安全类型注解，作用是告诉开发人员，参数可能会存在不安全的操作，要多注意一些；它会产生一个 unchecked 的警告；@SafeVarargs 注解只能用在可变长度参数的方法上，并且这个方法必须是 static 或 final 的，否则会出现编译错误。
- @FunctionalInterface：表示是只有一个方法的接口，JDK 8 引入。
- @Repeatable：表示注解的值可以有多个，比如我又帅气又幽默，这时候我同时有了“帅气”和“幽默”两个标签。


## 注解的使用场景

注解可以让编译器探测错误和警告；编译阶段可以利用注解生成文档、代码或做其他的处理；在代码运行阶段，一些注解还可以帮助完成代码提取之类的工作。

比如，使用过 Spring 框架的同学应该对 @Autowired 很熟悉了。使用 Spring 开发时，进行配置可以用 xml 配置文件的方式，现在用的更多的就是注解的方式。@Autowired 可以帮助我们注入一个定义好的 Bean。

@Autowired 的核心代码大概是这样的，作用就是 Spring 可以提取到使用 @Autowired 修饰的字段或方法做注入：

```Java
private InjectionMetadata buildAutowiringMetadata(final Class clazz) {
 //省略......
 do {
  //省略......

  //通过反射，获取这个类的所有字段，并遍历所有字段
  ReflectionUtils.doWithLocalFields(targetClass, new ReflectionUtils.FieldCallback() {
   //遍历字段的所有注解
   @Override
   public void doWith(Field field) throws IllegalArgumentException, IllegalAccessException {
    //如果字段使用了 @Autowired
    AnnotationAttributes ann = findAutowiredAnnotation(field);
    if (ann != null) {
     if (Modifier.isStatic(field.getModifiers())) {
      if (logger.isWarnEnabled()) {
       logger.warn("Autowired annotation is not supported on static fields: " + field);
      }
      return;
     }
     boolean required = determineRequiredStatus(ann);
     //先放到 currElements 中，后面一起集中处理
     currElements.add(new AutowiredFieldElement(field, required));
    }
   }
  });

  //省略......
 }
 while (targetClass != null && targetClass != Object.class);

 return new InjectionMetadata(clazz, elements);
}

//统一处理，也就是注入的源码我就不贴了
```

# 八、内部类

Java 的类中，不仅可以定义变量和方法，也能定义类，这种定义在类内的类，就叫做内部类。

## 静态内部类

定义在类内部的静态类，就是静态内部类；

静态内部类可以访问外部类所有的静态变量和方法，其他类访问静态内部类的时候，需要使用[外部类.静态内部类]的方式；

```Java
public class Outer {
	private static int x;
	private int y;

	public static class Inner{
		public void print(){
			System.out.println(x);
			System.out.println(y); //编译报错
		}
	}
}
```

## 成员内部类

定义在类内部的非静态类，就是成员内部类；

成员内部类中不能有静态方法和变量，外部类中的静态和非静态变量方法，内部类都能访问到。

```Java
public class Outer_2 {
	private static int x;
	private int y;

	public class Inner{
		public void print(){
			System.out.println(x);
			System.out.println(y);
		}
	}
}
```

## 匿名内部类

没有名字的内部类，就是匿名内部类...这好像是句废话。

因为没有名字，所以内部类只能使用一次；而且匿名内部类还有一个前提：必须继承一个类或实现一个接口。

```Java
public class Anonymous {
	interface Person {
	    public void say();
	}

	public static void main(String[] args) {
		Person p = new Person() {
            public void say() {
                System.out.println("I say");
            }
        };

        p.say();
	}
}
```

## 局部内部类

定义在方法中的类，就是局部类。

```Java
public class Outer_3 {
	private static int x;
	private int y;

	public void print() {
		class Inner{
			public void print(){
				System.out.println(x);
				System.out.println(y);
			}
		}

		Inner inner = new Inner();
		inner.print();
	}

	public static void main(String[] args) {
		Outer_3 out = new Outer_3();

		out.print();
	}
}
```

## 为什么使用内部类

1. 如果有一个类 A，除了类 B 使用之外不在任何其他地方使用；内部类是一种只在一个地方使用的类进行逻辑分组的方法。
2. 同上，如果一个类只对另外一个类有用，通过内部类嵌套可以更为精简。
3. 增加了封装：如果一个类 A 有特殊的访问权限，其中一些方法不希望被别人看到，这时候内部类可以被 private 修饰。
4. 具有更强的可读性和可维护性。

让我们看几个实际的应用：

1. List 、Set 中的迭代器

```Java
public class ArrayList<E>{

    public Iterator<E> iterator() {
        return new Itr();
    }

   private class Itr implements Iterator<E> {
        //迭代器的实现
   }
}
```

- 不同的容器的结构是不一样的，那么它们的实现细节肯定也是不一样的，比如 ArrayList 和 LinkedList 的遍历方法肯定不一样，所以不同容器的遍历方法不能抽象成公共方法；

- 如果让使用者自己写代码遍历容器，那么使用者就需要了解容器内部结构，那么对于容器使用者来说是一个很大的挑战；但如果为每个 List 、Set 的实现类，都写一个对应的迭代类，又有些多余；

- 所以 List 、Set 的实现类，都会包含一个迭代器内部类，它们只会被外部类使用，而且内部类对象是以外部类对象存在为前提的，没有 ArrayList 对象，就不会需要做遍历，也就不会有 Iterator 对象；

- 内部类被 private 修饰，屏蔽掉了实现细节，对于外部使用者来说，无疑是更为方便的。

2. 上面这个例子中，内部类对象是以外部类对象的存在为前提的（有外有内，没外没内），所以内部类是非静态的成员内部类，如果内部类的对象可以完全独立存在，只是借外部类的壳用一下，这时候可以使用静态内部类；比如 HashMap 内部的数据结构：

```Java
public class HashMap<K,V>{
   static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
   }

   transient Node<K,V>[] table;
}
```

HashMap 内部的数据结构是 Node<K,V> ，这个内部类属于外部类本身，但是不属于外部类的任何对象。

# 九、泛型

## 泛型的概念及好处

## 泛型的使用

### 泛型类

### 泛型方法

## 类型通配符


# 十、序列化

## Java 序列化的概念及作用

Java 在内存中创建可以复用的对象，这些对象的生命周期不会比 JVM 的生命周期更长，如果有一些对象需要在 JVM 停止后保存（硬盘），并在 JVM 启动后继续使用，或者需要在两个 JVM 之间传输这些对象（网络传输），但是只有二级制字节序列才能保存到硬盘或者在网络上传输，所以Java 的序列化和反序列化实际上就是对象和字节序列两种形态之间的转化。

- Java 序列化：把 Java 对象转换成字节序列；
- Java 反序列化：把字节序列转换成 Java 对象。

作用：可以让 Java 对象脱离 JVM 的运行而独立存在，这些字节序列可以保存在硬盘上，或通过网络传输。

注意：上述所有序列化的地方，我都写的是 Java 序列化，实际上现在很多框架已经比较少使用 Java 序列化对象进行网络通信了，大部分都使用某种协议驱动；比如 SOAP 协议就是将 XML 序列化成流。下文中没有特殊提示的地方，序列化都指的是 Java 序列化。

## 序列化实现的方式

一个对象想要被序列化，那么它对应的类要实现 Serializable 接口或它的子接口。

### 1. 实现 Serializable 接口

```Java
public class User implements Serializable {
   private String userName ;
   private String gender ;
   private String age ;

   public User(String userName, String gender, String age) {
      super();
      System.out.println("User Constructor");

      this.userName = userName;
      this.gender = gender;
      this.age = age;
   }

   //省略 toString、set、get 方法
}
```

```Java
public class SerializableTest {
   public static void main(String[] args) {
      try{
       ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("D:\\object.txt"));
       //将对象序列化到文件

       User user = new User("Suku","M","30");
       oos.writeObject(user);

       //从文件反序列化成对象
       ObjectInputStream ois = new ObjectInputStream(new FileInputStream("D:\\object.txt"));
       User user_in = (User) ois.readObject();
       System.out.println(user_in);
    } catch (Exception e) {
      e.printStackTrace();
    }
   }
}
```

运行结果：

```Java
User Constructor
User [userName=Suku, gender=M, age=30]
```

我们可以看到 User 的构造方法只在 new 的时候调用了一次，反序列化的时候并不需要调用构造方法；

如果一个可序列化的类，它的成员变量包含了不可序列化的类型，在序列化的过程中会报错：java.io.NotSerializableException ；

序列化并不能保存静态变量；

如果有不想序列化的字段，可以使用 transient 进行修饰；被 transient 修饰的字段，都会变成默认值（引用类型是 null，基本类型是 0 或 false）；

也可以不使用 transient 修饰符，通过重写 writeObject 和 readObject 方法，选择哪些属性需要序列化，哪些不需要序列化；

```Java
public class UserWriteRead implements Serializable {
   //省略 ...

   private void writeObject(ObjectOutputStream out) throws IOException {
      //将 userName 和 gender 写入二进制流
      out.writeObject(this.userName);
      out.writeObject(this.gender);
   }

   private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException{
      //从二进制流中反写  userName 和 gender
      this.userName = in.readObject().toString();
      this.gender = in.readObject().toString();
   }
}
```

### 2. 实现 Externalizable 接口

Externalizable 是 Serializable 接口的子类，通过实现的 writeExternal 和readExternal 方法，可以自己定义实现序列化和反序列化的方式；这里要注意：必须提供 public 的无参数的构造方法。

```Java
public class UserExter implements Externalizable {
 //省略 ...

 public UserExter() {} //必须有

 @Override
 public void writeExternal(ObjectOutput out) throws IOException {
  //将 userName 和 gender 写入二进制流
  out.writeObject(this.userName);
  out.writeObject(this.gender);
 }

 @Override
 public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
  //从二进制流中反写  userName 和 gender
  this.userName = in.readObject().toString();
  this.gender = in.readObject().toString();
 }
}
```

## 序列化版本

上述例子中，User user = new User("Suku","M","30") 被序列化之后保存到了文件中，如果在反序列化之前，User 中的内容已经发生了变化，那么反序列化还能成功么？或者说你还希望反序列化成功么？

- 如果新版本兼容老版本，比如增加了一个身份证号的属性，其余属性没有变化，那么反序列化是可以成功的，大不了身份证号为 null 嘛，至少不会报错；
- 如果新老版本不兼容，比如 userName 修改成了 name，那么反序列化之后对象就变成了 new User(null,"M","30") ，虽然程序并不会报错，但是却是一个我们不希望看到的结果；

所以随着序列化类的升级，我们可以使用 serialVersionUID 来保证升级前后的兼容性：
- 如果希望升级前后保持兼容，那么保持 serialVersionUID 不改变；
- 如果希望升级前后版本不兼容，那么不同版本中要设置不同的 serialVersionUID 。
- 如果没有显式地指定 serialVersionUID ，系统会自动生成一个，而当类名、类修饰符、属性、构造器等任何一个有改变，serialVersionUID 都会变化；但是不指定 serialVersionUID 的话可能会有一些隐患，因为不同的 JVM 对于 serialVersionUID 的计算规则可能是不一样的。
