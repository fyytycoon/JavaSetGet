Dubbo

分布式 

架构演变

微服务

nacos gateway feign

微服务概念

# Spring Cloud 框架

组件：

- 服务注册与发现：**服务提供方**将己方调用地址注册到**服务注册中心**，让服务调用方能够方便地找到自己；**服务调用方**从**服务注册中心**找到自己需要调用的服务的地址。Eureka，Zookeeper，Consul,Nacos等。

- 服务调用组件：

  - 负载均衡：**服务提供方**一般以多实例的形式提供服务，负载均衡功能能够让**服务调用方**连接到合适的服务节点。并且，服务节点选择的过程对服务调用方来说是透明的。**Ribbon**（客户端负载均衡，用于提供客户端的软件负载均衡算法，提供了一系列完善的配置项：连接超时、重试等），**OpenFeign**（优雅的封装Ribbon，是一个声明式RESTful网络请求客户端，它使编写Web服务客户端变得更加方便和快捷）。

  - 服务限流、降级、熔断:Hystrix(熔断降级，在出现依赖服务失效的情况下，通过隔离系统依赖服务的方式，防止服务级联失败，同时提供失败回滚机制，使系统能够更快地从异常中恢复)，

- 服务网关：服务网关是服务调用的唯一入口，可以在这个组件中实现用户鉴权、动态路由、灰度发布、A/B测试、负载限流等功能。Zuul，Gateway。
- 配置中心：将本地化的配置信息(Properties、XML、YAML等形式)注册到配置中心，实现程序包在开发、测试、生产环境中的无差别性，方便程序包的迁移，也是无状态特性。提供了配置集中管理，动态刷新配置的功能；配置通过Git或者其他方式来存储。
- 调用链监控：记录完成一次请求的先后衔接和调用关系，并将这种串行或并行的调用关系展示出来。在系统出错时，可以方便地找到出错点。Spring Cloud Sleuth（收集调用链路上的数据），Zipkin（对Sleuth收集的信息，进行存储，统计，展示）。
- 消息组件：Spring Cloud Stream（对分布式消息进行抽象，包括发布订阅、分组消费等功能，实现了微服务之间的异步通信）和Spring Cloud Bus（主要提供服务间的事件通信，如刷新配置）
- 安全控制组件：Spring Cloud Security 基于OAuth2.0开放网络的安全标准，提供了单点登录、资源授权和令牌管理等功能。







## 服务注册与发现

CAP 一致性（Consistency）、可用性（Availability）、分区容错性（Partition tolerance），一般情况下，保证AP，会舍弃C强一致性，但要保证最终一致性。





注册中心 Eureka、Zookeeper、Consul、nacos  


Eureka(AP) 为了保证了可用性，`Eureka` 不会等待集群所有节点都已同步信息完成，它会无时无刻提供服务。

Zookeeper(CP) 为了保证一致性，在所有节点同步完成之前是阻塞状态的。

### Eureka

P:三级缓存，本地缓存 Guava，缓存注册表

#### 自我保护

默认情况下，Eureka Server在一定时间内，没有接收到某个微服务心跳，会将某个微服务注销（90S）。但是当网络故障时，微服务与Server之间无法正常通信，上述行为就非常危险，因为微服务正常，不应该注销。

自我保护机制的触发条件：
（当每分钟心跳次数( renewsLastMin ) 小于 numberOfRenewsPerMinThreshold 时，并且开启自动保护模式开关( eureka.server.enable-self-preservation = true ) 时，触发自我保护机制，不再自动过期租约。）
numberOfRenewsPerMinThreshold = expectedNumberOfRenewsPerMin * 续租百分比( eureka.server.renewalPercentThreshold, 默认0.85 )
expectedNumberOfRenewsPerMin = 当前注册的应用实例数 x 2
为什么乘以 2：
默认情况下，注册的应用实例每半分钟续租一次，那么一分钟心跳两次，因此 x 2 。

服务实例数：10个，期望每分钟续约数：10 * 2=20，期望阈值：20*0.85=17，自我保护少于17时 触发。

自我保护关闭时，导致`isLeaseExpirationEnabled`为true，会剔除服务。
开启自我保护，结果为false，在 1 分钟后，Renews (last min) < Renews threshold * 0.85，就会触发自我保护机制，不剔除；反之剔除。

```
Renews (last min)：表示Eureka在最后一分钟，接收到的心跳数量

Renews threshold：表示Eureka认为在最后一分钟，应该接收到的心跳数量
```



注册 Register 。当 `Eureka` 客户端向 `Eureka Server` 注册时，它提供自身的**元数据**，比如IP地址、端口，运行状况指示符URL，主页等。

续约 Renew。**`Eureka` 客户会每隔30秒(默认情况下)发送一次心跳来续约**。 通过续约来告知 `Eureka Server` 该 `Eureka` 客户仍然存在，没有出现问题。 正常情况下，如果 `Eureka Server` 在90秒没有收到 `Eureka` 客户的续约，它会将实例从其注册表中删除。

**获取注册列表信息 Fetch Registries**： 

官方解释：`Eureka` 客户端从服务器获取注册表信息，并将其缓存在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。每次返回注册列表信息可能与 `Eureka` 客户端的缓存信息不同, `Eureka` 客户端自动处理。如果由于某种原因导致注册列表信息不能及时匹配，`Eureka` 客户端则会重新获取整个注册表信息。 `Eureka` 服务器缓存注册列表信息，整个注册表以及每个应用程序的信息进行了压缩，压缩内容和没有压缩的内容完全相同。`Eureka` 客户端和 `Eureka` 服务器可以使用JSON / XML格式进行通讯。在默认的情况下 `Eureka` 客户端使用压缩 `JSON` 格式来获取注册列表的信息。

下线 Cancel。Eureka客户端在程序关闭时向Eureka服务器发送取消请求。 发送请求后，该客户端实例信息将从服务器的实例注册表中删除。该下线请求不会自动完成，它需要调用以下内容：`DiscoveryManager.getInstance().shutdownComponent();`

剔除。连续90秒(3个续约周期)没有心跳，eureka server将它从注册表剔除。

#### Eureka的弱一致性

Eureka Server 启动后，才会通过 Eureka Client 请求其他 Eureka Server 节点中的一个节点，获取注册的服务信息，然后复制到其他 peer 节点。会存在数据同步不及时情况，而ZK在所有节点数据同步完成之前是阻塞状态。

Eureka只能保证最终一致性，不能保证强一致性。由于异步性，Eureka Client从Eureka Server获取的微服务节点会有失效的。这样，当访问失效的节点就会产生错误。所以开发的时候最好采用重试机制减少出错的可能。使最终一致性更加趋向于强一致性。

Ribbon是一个客户端负载均衡组件，它也属于spring cloud生态环境。它被用于微服务客户端，在请求微服务的时候实现负载均衡。其中一些配置就实现了重试机制，这种机制由两部分组成，一部分定义重试同一节点的次数，一部分定义重试多少个节点。通过配置这种机制就可以改善访问失效问题。

#### 客户端启动时如何注册到服务端

Eureka客户端在启动时，读取EurekaServer配置信息，从Eureka服务端拉去注册信息，缓存到本地，创建**三个心跳的定时任务**，一个定时向服务端**发送心跳**信息，服务端会对客户端心跳做出响应；另一个定时器**拉去注册表信息**，更新本地缓存；第三个**按需注册**，发生变化时发去注册信息。

#### 为什么有时候3个实例，后来都变成2个实例了？

```
多节点注意事项
问题：eureka server间 设置peer。A->B,B->C,C->A，结果注册信息并不同步。
看例子:
依次启动7901,7902,7903。
启动成功，注册api-driver ->7901
发现只有7901和7902有 api-driver而 7903没有。

简单说：api-driver向 7901注册，7902将api-driver同步到7902，但是不会同步到7903。后面源码会讲到。
多节点建议：设置A->B,A->C其他类似。尽量不要跨 eureka节点。一对多，面面对到。
```



#### Eurake优化

server优化

```
##    关闭自我保护机制，保证不可用服务被及时踢除
#    enable-self-preservation: false
#    eviction-interval-timer-in-ms: 2000
```



client优化

server最多3个 ，client`service-url`端配置随机打乱 。原因是，client启动时默认拉去配置的service-url第一个server注册表，失败才会拉去下一个server的注册表。





## 服务调用组件

服务端负载均衡：在客户端和服务端中间使用代理，nginx。

客户端负载均衡：根据自己的情况做负载。Ribbon就是。

### Ribbon 负载均衡

[Ribbon负载均衡](./document/springcloud组件/05-Ribbon负载均衡.md)

1. 几种负载均衡。（硬，软（服务端，客户端（Ribbon）））

   负载均衡，不管 `Nginx` 还是 `Ribbon` 都需要其算法的支持，如果我没记错的话 `Nginx` 使用的是 轮询和加权轮询算法。而在 `Ribbon` 中有更多的负载均衡调度算法，其默认是使用的 `RoundRobinRule` 轮询策略。

   * **`RoundRobinRule`**：轮询策略。`Ribbon` 默认采用的策略。若经过一轮轮询没有找到可用的 `provider`，其最多轮询 10 轮。若最终还没有找到，则返回 `null`。
   * **`RandomRule`**: 随机策略，从所有可用的 `provider` 中随机选择一个。
   * **`RetryRule`**: 重试策略。先按照 `RoundRobinRule` 策略获取 `provider`，若获取失败，则在指定的时限内重试。默认的时限为 500 毫秒。

2. Ribbon可以单独使用。需要提供服务地址列表。

3. 原理。拦截请求，然后替换地址（servicename到ip+port）。

```
1. 拦截请求。
2. 获取url。
3. 通过url中 serviceName 获取 List<ServiceInstance>。
4. 通过负载均衡算法选取一个ServiceInstance。
5. 替换请求，将原来url中的 serviceName换成ip+port。
```



1. 源码。ILoadBalancer，Map<服务名，ILoadBalancer>

```
由于加了@LoadBalanced注解，使用RestTemplateCustomizer对所有标注了@LoadBalanced的RestTemplate Bean添加了一个LoadBalancerInterceptor拦截器。利用RestTempllate的拦截器，spring可以对restTemplate bean进行定制，加入loadbalance拦截器进行ip:port的替换，也就是将请求的地址中的服务逻辑名转为具体的服务地址。
LoadBalancer 承接 eureka 和 ribbon。获取服务地址列表，选择一个。

每个服务都有ILoadBalancer。

选择服务用 IRule（负载均衡策略）。
```



1. @LoadBalanced，拦截器。（LoadBalancerInterceptor中intercept）



饥饿加载

```sh
ribbon:
  eager-load:
    enabled: true
    clients:
    - SERVICE-SMS

```

Spring Cloud默认是懒加载，指定名称的Ribbon Client第一次请求时，对应的上下文才会被加载，所以第一次访问慢。



改成以上饥饿加载后，将在启动时加载对应的程序上下文，从而提高首次请求的访问速度。

测试：

1. 上面配置为false启动，控制台没打印服务列表。
2. 为true：打印服务列表如下。



#### ribbon自定义负载均衡策略(Ribbon灰度)

发生在 服务调服务 情况下。

1. 自定义Rule。继承自com.netflix.loadbalancer.AbstractLoadBalancerRule
2. 实现 choose，灰度规则。从eureka注册列表中拿服务列表，获取*metadata*属性。用户信息，需要 AOP切面 拦截请求，拿到header，用户信息，放到ThreadLocal中。用户信息与服务信息匹配时，进行请求。



[Ribbon灰度方案](https://blog.csdn.net/sdmanooo/article/details/115480381)

[Spring Cloud灰度发布方案----自定义路由规则](https://blog.csdn.net/han949417140/article/details/121420529)

### Feign 服务调用

feign主要是构建微服务消费端。只要使用OpenFeign提供的注解修饰定义网络请求的接口类，就可以使用该接口的实例发送RESTful的网络请求。还可以集成Ribbon和Hystrix，提供负载均衡和断路器。

 GET多参数请求

1. 接口方法种使用  方法（@RequestParam("id") long id）。
2. 用map，方法（@RequestParam Map<String , Object> map）。

POST多参数请求

1. 用bean。方法（@RequestBody User bean）

原理

```
1. 主程序入口添加@EnableFeignClients注解开启对Feign Client扫描加载处理。根据Feign Client的开发规范，定义接口并加@FeignClient注解。
2. 当程序启动时，会进行包扫描，扫描所有@FeignClient注解的类，并将这些信息注入Spring IoC容器中。当定义的Feign接口中的方法被调用时，通过JDK的代理方式，来生成具体的RequestTemplate。当生成代理时，Feign会为每个接口方法创建一个RequestTemplate对象，该对象封装了HTTP请求需要的全部信息，如请求参数名、请求方法等信息都在这个过程中确定。
3. 然后由RequestTemplate生成Request，然后把这个Request交给client处理，这里指的Client可以是JDK原生的URLConnection、Apache的Http Client，也可以是Okhttp。最后Client被封装到LoadBalanceClient类，这个类结合Ribbon负载均衡发起服务之间的调用。
```

RestTemplate，自由，更贴近httpclient，方便调用别的第三方的http服务。

feign，更面向对象一些，更优雅一些。



#### [FeignClient如何共享Header以及踩坑过程](https://www.cnblogs.com/logan-w/p/12498448.html)



### Hystrix 服务限流、降级、熔断

概念：

- 舱壁模式

舱壁模式（Bulkhead）隔离了每个工作负载或服务的关键资源，如连接池、内存和CPU，硬盘。每个工作单元都有独立的 连接池，内存，CPU。

使用舱壁避免了单个服务消耗掉所有资源，从而导致其他服务出现故障的场景。
这种模式主要是通过防止由一个服务引起的级联故障来增加系统的弹性。



给我们的思路：可以对每个请求设置，单独的连接池，配置连接数，不要影响 别的请求。就像一个一个的防水舱。



- 雪崩效应

​		每个服务 发出一个HTTP请求都会 在 服务中 开启一个新线程。而下游服务挂了或者网络不可达，通常线程会阻塞住，直到Timeout。如果并发量多一点，这些阻塞的线程就会占用大量的资源，很有可能把自己本身这个微服务所在的机器资源耗尽，导致自己也挂掉。

​		如果服务提供者响应非常缓慢，那么服务消费者调用此提供者就会一直等待，直到提供者响应或超时。在高并发场景下，此种情况，如果不做任何处理，就会导致服务消费者的资源耗竭甚至整个系统的崩溃。一层一层的崩溃，导致所有的系统崩溃。

​		雪崩：由基础服务故障导致级联故障的现象。描述的是：提供者不可用 导致消费者不可用，并将不可用逐渐放大的过程。像滚雪球一样，不可用的服务越来越多。影响越来越恶劣。



雪崩三个流程：

服务提供者不可用

重试会导致网络流量加大，更影响服务提供者。

导致服务调用者不可用，由于服务调用者 一直等待返回，一直占用系统资源。

（不可用的范围 被逐步放大）



服务不可用原因：

服务器宕机

网络故障

宕机

程序异常

负载过大，导致服务提供者响应慢

缓存击穿导致服务超负荷运行



总之 ： 基础服务故障  导致 级联故障   就是  雪崩。



- 容错机制

1. 为网络请求设置超时。

   必须为网络请求设置超时。一般的调用一般在几十毫秒内响应。如果服务不可用，或者网络有问题，那么响应时间会变很长。长到几十秒。

   每一次调用，对应一个线程或进程，如果响应时间长，那么线程就长时间得不到释放，而线程对应着系统资源，包括CPU,内存，得不到释放的线程越多，资源被消耗的越多，最终导致系统崩溃。

   因此必须设置超时时间，让资源尽快释放。

2. 使用断路器模式。

   想一下家里的保险丝，跳闸。如果家里有短路或者大功率电器使用，超过电路负载时，就会跳闸，如果不跳闸，电路烧毁，波及到其他家庭，导致其他家庭也不可用。通过跳闸保护电路安全，当短路问题，或者大功率问题被解决，在合闸。

   自己家里电路，不影响整个小区每家每户的电路。

- 断路器

 		如果对某个微服务请求有大量超时（说明该服务不可用），再让新的请求访问该服务就没有意义，只会无谓的消耗资源。例如设置了超时时间1s，如果短时间内有大量的请求无法在1s内响应，就没有必要去请求依赖的服务了。

1. 断路器是对容易导致错误的操作的代理。这种代理能统计一段时间内的失败次数，并依据次数决定是正常请求依赖的服务还是直接返回。

2. 断路器可以实现快速失败，如果它在一段时间内检测到许多类似的错误（超时），就会在之后的一段时间，强迫对该服务的调用快速失败，即不再请求所调用的服务。这样对于消费者就无须再浪费CPU去等待长时间的超时。

3. 断路器也可自动诊断依赖的服务是否恢复正常。如果发现依赖的服务已经恢复正常，那么就会恢复请求该服务。通过重置时间来决定断路器的重新闭合。

   这样就实现了微服务的“自我修复”：当依赖的服务不可用时，打开断路器，让服务快速失败，从而防止雪崩。当依赖的服务恢复正常时，又恢复请求。

> 断路器开关时序图



```sh
第一次正常

第二次提供者异常

提供者多次异常后，断路器打开

后续请求，则直接降级，走备用逻辑。
```



​	断路器状态转换的逻辑：

```
关闭状态：正常情况下，断路器关闭，可以正常请求依赖的服务。

打开状态：当一段时间内，请求失败率达到一定阈值，断路器就会打开。服务请求不会去请求依赖的服务。调用方直接返回。不发生真正的调用。重置时间过后，进入半开模式。

半开状态：断路器打开一段时间后，会自动进入“半开模式”，此时，断路器允许一个服务请求访问依赖的服务。如果此请求成功(或者成功达到一定比例)，则关闭断路器，恢复正常访问。否则，则继续保持打开状态。

断路器的打开，能保证服务调用者在调用异常服务时，快速返回结果，避免大量的同步等待，减少服务调用者的资源消耗。并且断路器能在打开一段时间后继续侦测请求执行结果，判断断路器是否能关闭，恢复服务的正常调用。
```

> 《熔断.doc》《断路器开关时序图》《状态转换》



- 降级

为了在整体资源不够的时候，适当放弃部分服务，将主要的资源投放到核心服务中，待渡过难关之后，再重启已关闭的服务，保证了系统核心服务的稳定。当服务停掉后，自动进入fallback替换主方法。

用fallback方法代替主方法执行并返回结果，对失败的服务进行降级。当调用服务失败次数在一段时间内超过了断路器的阈值时，断路器将打开，不再进行真正的调用，而是快速失败，直接执行fallback逻辑。服务降级保护了服务调用者的逻辑。

```sh
熔断和降级：
共同点：
	1、为了防止系统崩溃，保证主要功能的可用性和可靠性。
	2、用户体验到某些功能不能用。
不同点：
	1、熔断由下级故障触发，主动惹祸。
	2、降级由调用方从负荷角度触发，无辜被抛弃。

```



## 服务网关

网关是介于客户端（外部调用方比如app，h5）和微服务的中间层。

### Zuul

zuul默认集成了：ribbon和hystrix。

[Zuul网关](./document/springcloud组件/08-网关.md)

Zuul的大部分功能都是有过滤器实现的。

#### 4种过滤器

```sh
PRE: 在请求被路由之前调用，可利用这种过滤器实现身份验证。选择微服务，记录日志，限流。
ROUTE:在将请求路由到微服务调用，用于构建发送给微服务的请求，并用http clinet（或者ribbon）请求微服务。
POST:在调用微服务执行后。可用于添加header，记录日志，将响应发给客户端。
ERROR:在其他阶段发生错误是，走此过滤器。
```

4种过滤器执行顺序：

pre->route->post 中间任何环节报错，走error和post，post环节报错只走error。



自定义filter步骤：

1. 继承 zuulfilter

2.  shouldFilter 执行条件设置为 true

3. run方法，过滤器的业务逻辑

4. filterType：pre,route,post,error

5. filterOrder：执行顺序，在谁前，在谁后，可以+1，-1，数字越小越容易执行

   

#### zuul filter 转发路由

背景：

老项目改造，合作方不愿意换老接口url，生产线上url需要按照需要通过网关转发给不同的service，

方案：

1、使用[静态路由](https://so.csdn.net/so/search?q=静态路由&spm=1001.2101.3001.7020)配置的方式，在yml配置文件中将原请求路径前缀和service-id进行对应
2、实现zuulfilter，将[url](https://so.csdn.net/so/search?q=url&spm=1001.2101.3001.7020)进行转发

对比：

之前用过自定义路由，在yml文件配置route的方式去做转发，遇到一个问题那就是

```
 zuul:
 	route:
 		serviceid-zuul-name: #此处名字随便取
 			path = /account/** 

```

但是不能保证请求的url,在/account/后面的url路径跟 account服务里面的路径一致，所以这样会有问题。`不优雅，不易拓展`这样的话，只能用另外一种方式了，那就是通过filter转发

```java
@Component
public class CommonServicePathFilter extends ZuulFilter {

    private final static String GETWAY_FOWARD_PREFIX="getway_forward_";

    private final static String GETWAY_COMPAY_CONFIG_KEY = "getway_company";

    @Autowired
    private RedisTemplate redisTemplate;

    //过滤器的类型
    @Override
    public String filterType() {
       //这里很重要，必须是route
        return "route";
    }

     //过滤器执行的顺序 一个请求在同一个阶段存在多个过滤器时候，多个过滤器执行顺序问题 
    @Override
    public int filterOrder() {
        return 1;
    }


    @Override
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        String url = ctx.getRequest().getRequestURI();
        Map<String,String> forwardMap =  getForwardMap(url);
        if(forwardMap != null){
            String fowardUrl = forwardMap.get(url);
            String serviceId = getServiceId(fowardUrl);
            String requestUrl = getRequestUrl(fowardUrl,serviceId);
              //1.设置目标service的Controller的路径
            ctx.put(FilterConstants.REQUEST_URI_KEY,requestUrl);
             //2.设置目标service的serviceId
            ctx.put(FilterConstants.SERVICE_ID_KEY,serviceId);
            /**
            *ctx.put(FilterConstants.REQUEST_URI_KEY,requestUrl);
但是我使用的时候，一直报404，后来跟踪服务之后，发现其实ctx里面还有一个serviceId的属性，它是跟当前请求的这个原始url映射的serviceId保持一致的，但是如果你要转发的serviceId并不是这个的话，那就会报404，所以必须要在这里重新定义serviceId,
            *
            */
        }
        return null;
    }
    private String getServiceId(String url){
        if(url.startsWith("/")){
            String temp = url.substring(1);
            return temp.split("/")[0];
        }else{
            return null;
        }
    }

    private String getRequestUrl(String url,String serviceId){
        return url.substring(serviceId.length() +1);
    }

    //判断过滤器是否生效
    @Override
    public boolean shouldFilter() {
       return true;
    }



    private Map<String,String> getForwardMap(String originalUrl){
           //todo:这里是返回一个map，传入一个originUrl,返回一个要转发的url
    }

}
```



#### 网关灰度

要完成灰度发布，要做的就是修改ribbon的负载均衡策略，通过一些特定的标识，比如我们针对某个接口路径/gray/publish/test。将10%的请求转发到新的服务上，将90%的请求转发到旧的服务上，诸如此类，我们可以制定各种规则进行灰度测试。

1. 通过eurake自定义元数据metadata，来区分新旧服务。
2. 自定义zuulfilter，按照自己的策略，修改ribbon的负载均衡。

```java
//核心代码就这么一行，实现了灰度，这里的version与要访问的服务的metadata-map中的key和value进行对应
RibbonFilterContextHolder.getCurrentContext().add("version","v1");

```

3. 可以通过数据库配置进行灰度灵活发布。



[Netflix-Ribbon灰度方案之Zuul网关灰度](https://blog.csdn.net/sdmanooo/article/details/115479360)



[微服务Zuul网关进行灰度发布](https://www.cnblogs.com/java-spring/p/13397270.html)



#### zuul限流

令牌桶限流：以一定 **固定的速率** 会往里面放令牌，一个请求过来首先要从桶中获取令牌，如果没有获取到，那么这个请求就拒绝，如果获取到那么就放行。

1. 限流发生在pre，每秒产生1000个令牌

```java
//每秒产生1000个令牌
private static final RateLimiter RATE_LIMITER = RateLimiter.create(1000);
```

2. 获取到就放行，获取不到就停止。

```java
// 如果获取不到就直接停止
requestContext.setSendZuulResponse(false);
//setSendZuulResponse(false) 是不往ROUTE filter 后面执行，因此应该在后面所有的过滤器加条件。set标志，以此做判断依据。
requestContext.set("limit",false);
```




[zuul开发实战（限流，超时解决）](https://www.cnblogs.com/wlwl/p/10413151.html)

#### zuul使用中遇到的问题

1. token 不会传播的其它的微服务中去。

```sh
zuul:
  #一下配置，表示忽略下面的值向微服务传播，以下配置为空表示：所有请求头都透传到后面微服务。
  sensitive-headers: token
  
```





Gateway



Resilience4j 服务容错

## 配置中心

Config 分布式配置中心

[网关](./document/springcloud组件/08-网关.md)

[链路追踪和健康检查](./document/springcloud组件/10-链路追踪和健康检查.md)

SpringCloud Bus 消息总线 

[Spring Cloud Bus](http://www.ityouknow.com/springcloud/2017/05/26/springcloud-config-eureka-bus.html)

- 1、提交代码触发post给客户端A发送bus/refresh
- 2、客户端A接收到请求从Server端更新配置并且发送给Spring Cloud Bus
- 3、Spring Cloud bus接到消息并通知给其它客户端
- 4、其它客户端接收到通知，请求Server端获取最新配置
- 5、全部客户端均获取到最新的配置

## 调用链监控

## 常见问题

1. 解决服务注册慢，被其他服务发现慢的问题。

   ```sh
   eureka.instance.lease-renewal-interval-in-seconds: 10,续约的时间间隔，默认是30秒，建议用默认值。
   因为服务最少续约3次心跳才能被其他服务发现，所以我们缩短心跳时间。
   ```

2. 已停止的微服务节点，注销慢或不注销。建议默认。

```sh
eureka server：

eureka:
  server: 
    #关闭自我保护
    enable-self-preservation: false
    #缩短清理间隔时间
    eviction-interval-timer-in-ms: 5000
    
eureka client:
eureka: 
  instance: 
    lease-renewal-interval-in-seconds: 10 //缩短心跳间隔。默认30秒
    lease-expiration-duration-in-seconds: 90 //缩短续约到期时间，默认90秒。

```

3. instanceId的设置，要一目了然。
4. 整合hystrix后，首次请求失败。

原因：hystrix默认超时时间是1秒，如果1秒内无响应，就会走fallback逻辑。由于spring的懒加载机制，首次请求要去获取注册表信息等。所以首次请求一般会超过1秒。

解决方法1：配置饥饿加载

```sh
ribbon:
  eager-load:
    enabled: true
    clients:
    - SERVICE-SMS
    
如果是网关
zuul: 
  ribbon:
    eager-load:
      enabled: true
      
```

解决方法2：设长hystrix超时时间，在command命令中设置

```sh
execution.isolation.thread.timeoutInMilliseconds
```



# Spring Cloud Alibaba

Nacos 注册、配置中心

OpenFeign 服务调用

Sentinel 流控

Seata 分布式事务