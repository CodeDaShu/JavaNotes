分布式架构下的 Session 共享，也称作分布式 Session 一致性；分布式架构下 Session 共享有哪些问题，又有哪些解决方案，让我们一起看一下。

## Session 的作用

如果大家做过 Web 应用开发的话，应该对 Session 比较熟悉；服务器会为每个用户创建一个会话，存储用户的相关信息，以便在后面的请求中，可以够定位到同一个上下文。

✔︎  举个例子

用户在登录之后，再进行页面跳转的时候，存储在 Session 中的信息会被一直保存；

如果用户没有 Session ，那么服务器会创建一个 Session 对象，直到会话过期或主动放弃后（退出），服务器才会把 Session 终止掉。

![Session 的作用](https://github.com/CodeDaShu/JavaNotes/blob/master/img/DistributedSession/Session.jpg)

## 分布式架构中 Session 的问题

✔︎  单体架构

在单体服务器的年代，Session 直接保存在服务器中，是没有问题的，而且实现起来很容易。

✘  分布式架构

但是随着分布式架构的流行，单个服务器已经不能满足系统的需要了，通常都会把系统部署在多台服务器上，通过负载均衡把请求分发到其中的一台服务器上；

那么很有可能第一次请求访问的 A 服务器，创建了 Session ，但是第二次访问到了 B 服务器，这时就会出现取不到 Session 的情况；

于是，分布式架构中，Session 共享就成了一个很大的问题。

![分布式架构下的 Session 问题](https://github.com/CodeDaShu/JavaNotes/blob/master/img/DistributedSession/DistributedSession.jpg)

## 解决方案

**1. 不要有 Session**

大家可能觉得我说了句废话，但是确实在某些场景下，是可以没有 Session 的，其实在很多接口类系统当中，都提倡【 无状态服务 】；也就是每一次的接口访问，都不依赖于 Session ，不依赖于前一次的接口访问；

**2. 存入 Cookie 中**

可以将 Session 存储到 Cookie 中，但是缺点也很明显，例如每次请求都得带着 Session ，数据存储在客户端本地，是有风险的；

**3. Session 同步**

服务器之间进行 Session 同步，这样可以保证每个服务器上都有全部的 Session 信息，不过当服务器数量比较多的时候，同步是会有延迟甚至同步失败；

**4. IP 绑定策略**

使用 Nginx （或其他复杂均衡软硬件）中的 IP 绑定策略，同一个 IP 只能在指定的同一个机器访问，但是这样做失去了负载均衡的意义，当挂掉一台服务器的时候，会影响一批用户的使用，风险很大；

**5. 使用 Redis 存储**

我们现在的系统会把 Session 放到 Redis 中存储，虽然架构上变得复杂，并且需要多访问一次 Redis ，但是这种方案带来的好处也是很大的：

*   实现了 Session 共享；
*   可以水平扩展（增加 Redis 服务器）；
*   服务器重启 Session 不丢失（不过也要注意 Session 在 Redis 中的刷新/失效机制）；
*   不仅可以跨服务器 Session 共享，甚至可以跨平台（例如网页端和 APP 端）。

![Session-Redis](https://github.com/CodeDaShu/JavaNotes/blob/master/img/DistributedSession/Session-Redis.jpg)
