<!-- TOC -->
- [一、数据类型](#一数据类型)
- [二、String](#String)
  - [String 的不可变性](#String的不可变性)
- [三、关键字](#三关键字)
- [四、继承](#四继承)
- [五、异常](#异常)
- [六、反射](#六反射)
- [七、注解](#七注解)
- [八、内部类](#八内部类)
- [九、泛型](#九泛型)
- [十、序列化](#十序列化)
<!-- TOC -->


# 一、数据类型


# 二、String

## String 的不可变性
String 被声明为 final，是不可变的，它也不可被继承。

### 通过源码了解 String 的不可变性

在 Java 8 中，String 是使用 char 数组实现的。
```
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
   /** The value is used for character storage. */
   private final char value[];

   /** Cache the hash code for the string */
   private int hash; // Default to 0
}
```
到了 Java 9 ，String 类的实现改用 byte 数组实现，同时使用 coder 来标识使用了哪种编码。
```
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


# 四、继承


# 五、异常


# 六、反射


# 七、注解


# 八、内部类


# 九、泛型


# 十、序列化
