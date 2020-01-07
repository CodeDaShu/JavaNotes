先举个例子，说明为什么要做“限流”。

旅游景点通常都会有最大的接待量，不可能无限制的放游客进入，比如故宫每天只卖八万张票，超过八万的游客，无法买票进入，因为如果超过八万人，景点的工作人员可能就忙不过来，过于拥挤的景点也会影响游客的体验和心情，并且还会有安全隐患；

只卖 N 张票，这就是一种限流的手段。

### 软件架构中的限流
软件架构中的限流也是类似，也是当系统资源不够的时候，已经不足以应对大量的请求，为了保证服务还能够正常运行，那么按照规则，系统会把多余的请求直接拒绝掉，以达到限流的效果；

**对外限流：** 用户过多，或因为某个活动或热点问题引发的访问量的增加；恶意攻击，或被爬虫抓取数据等等。不知道大家注意过没有，比如双11，刚过12点有些顾客的网页或APP会显示下单失败的提示，有些就是被限流掉了。

**对内限流：** 大部分公司，系统和系统之间都会相互调用，假如 A 系统 被 X、Y、Z 三个系统调用，当 X 系统的访问量徒增，导致 A 系统被调的挂掉了，那么会导致 Y、Z 系统也无法正常使用（依赖于 A 系统）。

### 计数器法
计数器法的原理就是限制单位时间内的处理请求数不超过阈值。

比如一个接口一分钟可以处理 1000 次请求，那么可以设置一个计数器，当有一次请求过来，计数器就加 1，如果一分钟以内计数器超过了 1000，那么后面再过来的请求就不再处理；

但是这个方法的缺点也很明显，因为请求的访问不一定是很平稳的，如果 0:59 过来了 1000 个请求，1:01 已经是下一个窗口，又过来了 1000 个请求，但实际上三秒内来了 2000 个请求，已经超过我们的限流上限了；

![计数器法](https://github.com/CodeDaShu/JavaNotes/blob/master/img/algorithm/count.jpg)

```
public class CounterTest {
   public static void main(String[] args) {
       long timeStamp = System.currentTimeMillis();
       int timeCount = 1;

       int reqCount = 0; //单位时间内的请求数量
       int reqCountTotal = 0; //请求总数

       final int limit = 10; // 时间窗口内最大请求数
       final long interval = 1000; // 时间窗口ms

       for(;;){
        // 当前时间
        long now = System.currentTimeMillis();

        if (now < timeStamp + interval) {
         //在之间窗口内
         reqCount ++ ;
         //没有超过单位时间的请求数量
         if(reqCount < limit){
          reqCountTotal ++ ;
          System.out.println(timeCount + " : " + reqCountTotal);
         }
        }else {
         //下一个时间窗口
         timeStamp = now;
         reqCount = 0 ;
         timeCount ++ ;
        }
     }
   }
}
```
### 滑动窗口算法
还拿上面的例子，一分钟分 6 份，每份 10 秒；每过 10 秒钟，我们的时间窗口就会往右滑动一格，每个格子都有独立的计数器，我们每次都计算时间窗口内的数量，可以解决计数器法中的问题，而且**当滑动窗口的格子越多，那么限流的统计就会越精确。**

具体可以参考下图，看图比较清晰：

![滑动窗口法](https://github.com/CodeDaShu/JavaNotes/blob/master/img/algorithm/slidingWindow.jpg)

### 漏桶算法
这个算法也很简单，就是我们有一个固定容量的桶，有水流进来，也有水流出去，我们不需要控制流进来的速度，只需要控制流出去的速度，如果水流进来的太快，桶满了，多余的水会溢出去，并不会影响水流出去的速度。
![漏桶算法](https://github.com/CodeDaShu/JavaNotes/blob/master/img/algorithm/LeakyBucket.jpg)

### 令牌桶算法
还是有一个桶，桶里面有 N 个令牌，所有的请求在处理之前都需要拿到一个可用的令牌才会被处理，如果桶里面没有令牌的话，则拒绝服务；令牌桶算法的原理是系统会以一个恒定的速度往桶里放入令牌。

有人会把漏桶算法和令牌桶算法混淆，而从某种意义上讲，令牌桶算法确实是对漏桶算法的一种改进，**两者的不同之处在于：漏桶算法能够强行限制数据的传输速率，而令牌桶算法能够在限制数据的平均传输速率的同时还允许某种程度的突发传输；**

也就是说，如果令牌桶中存在令牌，则允许发送流量；而如果令牌桶中不存在令牌，则不允许发送流量；放入令牌的动作是持续执行的，比如桶的大小为 100 ，当一直没有请求的时候，桶内的令牌数量始终保持在 100 ，当下一秒有大量请求过来的时候，**可以瞬间将“攒下”的 100 个令牌处理掉**，然后再按照恒定速度进行处理。

![令牌桶算法](https://github.com/CodeDaShu/JavaNotes/blob/master/img/algorithm/TokenBucket.jpg)


再安利一下 Google 开源的 guava 包，使用 **RateLimiter** 我们可以很轻松的创建一个令牌桶算法的限流器。
```
import java.util.concurrent.Executors;
import com.google.common.util.concurrent.ListeningExecutorService;
import com.google.common.util.concurrent.MoreExecutors;
import com.google.common.util.concurrent.RateLimiter;

public class RateLimiterTest {
    public static void main(String[] args) {
        testRateLimiter();
    }

    public static void testRateLimiter() {   
        ListeningExecutorService executorService = MoreExecutors
                .listeningDecorator(Executors.newFixedThreadPool(5));
        RateLimiter limiter = RateLimiter.create(4); // 每秒不超过4个任务被提交

        int num = 0 ;
        for (;;) {
            limiter.acquire(); // 请求RateLimiter, 超过permits会被阻塞
            num ++ ;
            executorService.submit(new Task("is "+ num));
        }
    }
}

class Task implements Runnable{
    String str;
    public Task(String str){
        this.str = str;
    }
    @Override
    public void run() {
        System.out.println("Task call execute.." + str);
    }

}
```
