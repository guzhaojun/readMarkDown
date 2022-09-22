默认是单例的，   可以加上注解 scope(“prototype”) 或者  @Scope（“request”）


Spring 的 Bean 默认都是单例的，某些情况下，单例是并发不安全的，以 Controller 举例，问题根源在于，我们可能会在 Controller 中定义成员变量，如此一来，多个请求来临，进入的都是同一个单例的 Controller 对象，并对此成员变量的值进行修改操作，因此会互相影响，无法达到并发安全（不同于线程隔离的概念，后面会解释到）的效果。
首先来举个例子，证明单例的并发不安全性：
```Java
@Controller
public class HomeController {
    private int i;
    @GetMapping("testsingleton1")
    @ResponseBody
    public int test1() {
        return ++i;
    }
}
```
多次访问此 url，可以看到每次的结果都是自增的，所以这样的代码显然是并发不安全的。
如何解决呢？
我们为了让无状态的海量 HTTP 请求之间不受影响，我们可以采取以下几种措施：
1、单例变原型
对 web 项目，可以 Controller 类上加注解 @Scope("prototype") 或 @Scope("request")，对非 web 项目，在 Component 类上添加注解 @Scope("prototype") 。
这种方式实现起来非常简单，但是很大程度上增大了 Bean 创建实例化销毁的服务器资源开销。
2、线程隔离类 ThreadLocal
有人想到了线程隔离类 ThreadLocal，我们尝试将成员变量包装为 ThreadLocal，以试图达到并发安全，同时打印出 HTTP 请求的线程名，修改代码如下：
``` Java
@Controller
public class HomeController {
    private ThreadLocal<Integer> i = new ThreadLocal<>();
    @GetMapping("testsingleton1")
    @ResponseBody
    public int test1() {
        if (i.get() == null) {
            i.set(0);
        }
        i.set(i.get().intValue() + 1);
        log.info("{} -> {}", Thread.currentThread().getName(), i.get());
        return i.get().intValue();
    }
}
```
多次访问此 url 测试一把，打印日志如下：
![20220922084217](https://raw.githubusercontent.com/guzhaojun/readMarkDown/main/images/20220922084217.png)
从日志分析出，二十多次的连续请求得到的结果有 1 有 2 有 3 等等，而我们期望不管我并发请求有多少，每次的结果都是 1；同时可以发现 web 服务器默认的请求线程池大小为 10，这 10 个核心线程可以被之后不同的 HTTP 请求复用，所以这也是为什么相同线程名的结果不会重复的原因。
ThreadLocal 的方式可以达到线程隔离，但还是无法达到并发安全。
3、尽量避免使用成员变量
有人说，单例 Bean 的成员变量这么麻烦，能不用成员变量就尽量避免这么用，在业务允许的条件下，将成员变量替换为 RequestMapping 方法中的局部变量，多省事。这种方式自然是最恰当的，本人也是最推荐。代码修改如下：
```  Java
@Controller
public class HomeController {
    @GetMapping("testsingleton1")
    @ResponseBody
    public int test1() {
         int i = 0;
         // TODO biz code
         return ++i;
    }
}
```
但当很少的某种情况下，必须使用成员变量呢，我们该怎么处理？
4、使用并发安全的类
Java 作为功能性超强的编程语言，API 丰富，如果非要在单例 Bean 中使用成员变量，可以考虑使用并发安全的容器，如 ConcurrentHashMap、ConcurrentHashSet 等等，将我们的成员变量（一般可以是当前运行中的任务列表等这类变量）包装到这些并发安全的容器中进行管理即可。
5、分布式或微服务的并发安全
如果还要进一步考虑到微服务或分布式服务的影响，方式 4 便不足以处理了，所以可以借助于可以共享某些信息的分布式缓存中间件如 Redis 等，这样即可保证同一种服务的不同服务实例都拥有同一份共享信息（如当前运行中的任务列表等这类变量）。