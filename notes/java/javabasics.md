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
  - [元注解](#类型转换)
  - [缓存池机制](#注解处理器)
- [八、内部类](#八内部类)
  - [静态内部类](#静态内部类)
  - [成员内部类](#成员内部类)
  - [匿名内部类](#匿名内部类)
  - [局部内部类](#局部内部类)
- [九、泛型](#九泛型)
  - [泛型的概念及好处](#泛型的概念及好处)
  - [泛型的使用](#泛型的使用)
  - [泛型类](#泛型类)
  - [泛型方法](#泛型方法)
  - [类型通配符](#类型通配符)
- [十、序列化](#十序列化)
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

#### 1. 线程安全
因为 String 的不可变性，所以在多线程的环境下，可以被安全使用。

#### 2. hash 值不需要重复计算
一个 String 被创建后，那么它的 hash 值只需要计算一次；
比如使用 String 做为 HashMap 中的 key ，那么不需要重复计算 hash 值。

#### 3. String Pool
一个 String 被创建后，被可以放入 String Pool 中，就可以被其他地方所引用；因为 String 是不可变的，所以才能实现 String Pool 。

### String, StringBuffer, StringBuilder

#### 1. 不可变性
String 不可变
StringBuffer 和 StringBuilder 可变

#### 2. 线程安全
String 不可变，因此是线程安全的
StringBuilder 不是线程安全的
StringBuffer 是线程安全的，内部使用 synchronized 进行同步


# 三、关键字

## static

## final

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

## 异常的分类

## 异常的处理方式


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

## 元注解

## 注解处理器


# 八、内部类

## 静态内部类

## 成员内部类

## 匿名内部类

## 局部内部类

# 九、泛型

## 泛型的概念及好处

## 泛型的使用

### 泛型类

### 泛型方法

## 类型通配符


# 十、序列化
