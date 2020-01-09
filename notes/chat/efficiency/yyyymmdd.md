大叔我北漂十多年，一直没有摇到北京的车牌，每周都需要通过一个 APP 办理“进京证”，当我办理 19 年最后一次进京证的时候，APP 给出了这样的提示：

![错误的日期](https://github.com/CodeDaShu/JavaNotes/blob/master/img/chat/jinjingzheng.jpg)

日期显示：“2020-12-31”！

车友群里面立马有人不淡定了，虽然大家都猜出来，这应该是 APP 的 Bug，但还是难免要吐槽一下，讨论到最后，就快要“杀个程序员祭天了”！

那么产生这个 Bug 的原因是什么呢？其实很简单，就是 把 yyyy-MM-dd 写成了 YYYY-MM-dd 。

如果对时间处理不那么熟悉的程序员看到这里，会认为 yyyy 和 YYYY 有什么区别么？在代码里面敲一下，他们的结果也都是相同的啊！

```
public class DateTest {
  public static void main(String[] args) {
    Calendar calendar = Calendar.getInstance();

    calendar.set(2019, Calendar.AUGUST, 31);

    Date strDate = calendar.getTime();

    DateFormat formatUpperCase = new SimpleDateFormat("yyyy-MM-dd");
    System.out.println("2019-08-31 to yyyy-MM-dd: " + formatUpperCase.format(strDate));

    formatUpperCase = new SimpleDateFormat("YYYY-MM-dd");
    System.out.println("2019-08-31 to YYYY/MM/dd: " + formatUpperCase.format(strDate));
  }
}
```
运行结果为：

```
2019-08-31 to yyyy-MM-dd: 2019-08-31
2019-08-31 to YYYY/MM/dd: 2019-08-31
```
但是如果我们把日期改成 2019-12-31 再试试呢？结果产生了差异：

```
2019-12-31 to yyyy-MM-dd: 2019-12-31
2019-12-31 to YYYY-MM-dd: 2020-12-31
```

那么产生这个问题的原因是什么呢？其实很简单：**Y 和 y 实际上代表了不同的含义。**

![Java的时间](https://github.com/CodeDaShu/JavaNotes/blob/master/img/chat/time.jpg)

- y：year-of-era；正正经经的年；
- Y：week-based-year；只要本周跨年，那么这周就算入下一年；也就是 12 月

**这是开发过程中的一个小细节，一不小心就掉到坑里了。**
