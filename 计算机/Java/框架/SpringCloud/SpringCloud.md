# SpringCloud

[TOC]

## 一、SpringCloud概述

### 1.1 微服务概述

#### 微服务定义

微服务的思想将单一应用程序划分成一组小的服务，每个服务运行独立的自己的进程中，服务之间互相协调、互相配合



#### 微服务带来的优势

| 优势       | 说明                                                 |
| ---------- | ---------------------------------------------------- |
| 独立开发   | 所有微服务都可以根据各自的功能轻松开发               |
| 独立部署   | 根据他们所提供的服务，可以在任何应用中单独部署       |
| 故障隔离   | 即使应用中的一个服务不起作用，系统仍然继续运行       |
| 混合技术栈 | 可以用不同的语言和技术来构建同一应用程序的不同服务   |
| 粒度缩放   | 各个组件可根据需要进行扩展，无需将所有组件融合到一起 |



#### 微服务架构的优缺点

| 微服务架构的优点           | 微服务架构的缺点             |
| -------------------------- | ---------------------------- |
| 自由使用不同的技术         | 增加故障排除挑战             |
| 每个微服务都侧重于单一功能 | 由于远程呼叫而增加延迟       |
| 支持单个可部署单元         | 增加了配置和其他操作的工作量 |
| 允许经常发布软件           | 难以保持交易安全             |
| 确保每项服务的安全性       | 艰难地跨越各种便捷跟踪数据   |
| 多个服务是并行开发和部署的 | 难以在服务之间进行编码       |

### 1.2 SpringCloud组件

+ 注册中心
  + Eureka（停止维护了）
  + Zookeeper
  + Consul
  + Nacos
+ 服务调用
  + Ribbon
  + LoadBalancer
  + Feign（停止维护）
  + OpenFeign
+ 服务降级
  + Hystrix
  + resilience4j
  + Sentinel
+ 服务网关
  + Zuul（停止维护）
  + gateway
+ 服务配置
  + Config（停止维护）
  + Nacos
+ 服务总线
  + Bus（停止维护）
  + Nacos



### 1.3 HTTP远程调用类 RestTemplate

通过RestTemplate可以远程调用另外一个模块或者程序的controller，例如有两个模块，分别是订单Order模块和支付Payment模块，其中订单Order模块是通过视图层将相关接口暴露给用户，然后调用支付Payment模块的方法。支付Payment模块视图层代码如下：

```java
@Slf4j
@RestController
@RequestMapping("/payment")
public class PaymentController {

    @Resource
    private PaymentService paymentService;

    @PostMapping("/create")
    public CommonResult create(@RequestBody Payment payment) {
        int result = paymentService.create(payment);
        log.info("插入结果:{}", result);

        if (result > 0) {
            return new CommonResult(200, "插入数据库成功", result);
        }

        return new CommonResult(444, "插入数据库失败", null);
    }

    @GetMapping("/get/{id}")
    public CommonResult getPaymentById(@PathVariable("id") Long id) {
        Payment payment = paymentService.getPaymentById(id);
        log.info("哈哈哈哈呵呵呵");
        if (payment != null) {
            return new CommonResult(200, "查询成功", payment);
        }

        return new CommonResult(444, "没有对应记录, 查询ID:" + id, null);
    }
}
```



那么如果在订单Order模块需要调用支付Payment模块需要通过通过RestTemplate来调用，首先需要通过配置类来获取RestTemplate对象：

```java
@Configuration
public class ApplicationContextConfig {

    // @Bean将该方法返回值注入到Spring容器中
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

之后在视图层中通过RestTemplate调用支付Payment模块的接口

```java
@Slf4j
@RestController
@RequestMapping("/consumer")
public class OrderController {

    public static final String PAYMENT_URL = "http://localhost:8001";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/payment/create")
    public CommonResult<Payment> create(Payment payment) {
        return restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
    }

    @GetMapping("/payment/get/{id}")
    public CommonResult<Payment> getPayment(@PathVariable("id") Long id) {
        return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
    }
}
```



## 二、注册中心

### 2.1 概述

在服务注册发现中，服务器启动，会将当前服务器信息比如服务通讯地址以别名的形式注册到注册中心，消费者或者服务提供者通过别名的方式去注册中心获取实际的服务通讯地址，然后实现本地RPC。



### 2.2 Eureka（已停止维护）

#### 2.2.1 配置

配置文件application.yaml

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost  #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己（想注册也可以，不过没必要）
    register-with-eureka: false
    #false表示自己端就是注册中心，职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与eurekaServer交互的地址查询服务和注册服务都需要依赖这个地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```



启动类，需要加上@EnableEurekaServer注解，表示当前程序为服务注册中心程序

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaMain7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaMain7001.class, args);
    }
}
```

启动服务后，通过http://localhost:7001 访问Eureka服务注册中心



#### 2.2.2 Eureka集群

##### 集群原理示意图

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/SpringCloud/Eureka%E9%9B%86%E7%BE%A4%E7%A4%BA%E6%84%8F%E5%9B%BE.png" align="left" alt="Eureka集群示意图">



##### Eureka集群搭建

创建两个Eureka注册中心模块，分别为cloud-eureka-server7001和cloud-eureka-server7002



搭建集群需要将注册中心互相注册，也就是每一个注册中心里面都必须有其余所有注册中心注册信息

在cloud-eureka-server7001中将自己注册进cloud-eureka-server7002中，配置文件如下

```yaml
server:
  port: 7001

eureka:
  instance:
    #eureka服务端的实例名称
    hostname: eureka7001.com
  client:
    #false表示不向注册中心注册自己（想注册也可以，不过没必要）
    register-with-eureka: false
    #false表示自己端就是注册中心，职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与eurekaServer交互的地址查询服务和注册服务都需要依赖这个地址
      defaultZone: http://eureka7002.com:7002/eureka/
```

同时在cloud-eureka-server7002中将自己注册进cloud-eureka-server7001中，配置文件如下

```yaml
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com  #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己（想注册也可以，不过没必要）
    register-with-eureka: false
    #false表示自己端就是注册中心，职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与eurekaServer交互的地址查询服务和注册服务都需要依赖这个地址
      defaultZone: http://eureka7001.com:7001/eureka/
```



修改windows系统hosts文件，添加两行数据

```
127.0.0.1       eureka7001.com
127.0.0.1       eureka7002.com
```



##### 支付Payment服务集群配置

将原来的cloud-provider-payment8001中文件进行拷贝入cloud-provider-payment8002，修改两个的配置文件，将这两个服务注册进入Eureka集群中

```yaml
eureka:
  client:
    #true表示向注册中心注册自己，默认为true
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```



##### 订单Order服务的修改

将对应的订单Order服务和支付Payment服务注册进注册中心中，具体操作是修改这两个服务配置文件中defaultZone字段的值

```yaml
defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```



修改订单Order服务中OrderController中对访问的支付Payment服务url，将

```java
public static final String PAYMENT_URL = "http://localhost:8001";
```

改成

```java
public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";
```

> 这里的`CLOUD-PAYMENT-SERVICE`是Eureka注册中心里面支付Payment服务的名称，通过 http://localhost:7001 或者 http://localhost:7002 查看



同时需要修改Config文件，加上@LoadBalanced注解已达到轮询负载的访问8001和8002端口下的服务

```java
@Configuration
public class ApplicationContextConfig {

    // @Bean将该方法返回值注入到Spring容器中
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```





## 三、负载均衡

### 3.1 客户端负载均衡和服务端负载均衡

服务端负载均衡的通常是使用nginx实现的，客户端发送的请求都会打到nginx上，nginx根据对应的负载均衡算法将请求转发给对应的服务器

而客户端的负载均衡是当前服务向注册中心获取对应服务的列表（例如有多个支付服务），根据负载均衡算法，向列表中对应的算法发出请求



常见负载均衡算法：

+ 轮询
+ 随机
+ 权重，相应速度越快，权重越大，越容易被选择
+ 过滤掉多次访问故障和处于跳闸状态服务后，选择一个并发量小的服务



### 3.2 Ribbon

#### Ribbon配置

最新的Eureka中已经引入了Ribbon的依赖，故不需要再次引入



对RestTemplate配置类加上@LoadBalanced注解开启负载均衡

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```



在Controller中通过RestTemplate进行远程调用

```java
@Slf4j
@RestController
@RequestMapping("/consumer")
public class OrderController {

    public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/payment/create")
    public CommonResult<Payment> create(Payment payment) {
        return restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
    }

    @GetMapping("/payment/get/{id}")
    public CommonResult<Payment> getPayment(@PathVariable("id") Long id) {
        return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
    }

    @GetMapping("/payment/getForEntity/{id}")
    public CommonResult<Payment> getPaymentEntity(@PathVariable("id") Long id) {
        ResponseEntity<CommonResult> entities = restTemplate.getForEntity(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);

        if (entities.getStatusCode().is2xxSuccessful()) {
            return entities.getBody();
        }

        return new CommonResult<>(444, "操作失败");
    }
}
```





##### 自定义负载均衡规则

注意：不能放在@ComponentScan所扫描的当前包下及其子包下，否则配置会被所有Ribbon客户端共享，无法达到特制化目的。例如原来相关文件都在`com.ecifics.springcloud`包下，那么自定义规则可以放在`com.ecifics.myrule`，创建自定义类myRule：

```java
@Configuration
public class MySelfRule {

    @Bean
    public IRule myRule() {
        return new RandomRule();
    }
}
```



在启动类上加上`@RibbonClient(value = "CLOUD-PAYMENT-SERVICE", configuration = MySelfRule.class)`并执行对应规则类

```java
@SpringBootApplication
@EnableEurekaClient
@RibbonClient(value = "CLOUD-PAYMENT-SERVICE", configuration = MySelfRule.class)
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class, args);
    }
}
```



### 3.3 OpenFeign

#### 基本配置

OpenFeign不同于Ribbon，Ribbon需要和RestTemplate配合使用，而OpenFeign只需要在对应的服务层接口类上通过注解的形式绑定对应的服务来进行负载均衡



主启动类上需要加上注解`@EnableFeignClients`

```java
@EnableFeignClients
@SpringBootApplication
public class OrderFeignMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderFeignMain80.class, args);
    }
}
```



在对应的服务类接口上加上注解@FeignClient，并制定服务名称

```java
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
@RequestMapping("/payment")
public interface PaymentFeignService {

    @GetMapping("/get/{id}")
    CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);

    @GetMapping("/feign/timeout")
    String paymentFeignTimeout();
}
```



在Controller中直接调用对应服务接口提供的方法

```java
@RequestMapping("/consumer")
public class OrderFeignController {

    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping("/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id) {
        return paymentFeignService.getPaymentById(id);
    }


    @GetMapping("/payment/feign/timeout")
    public String paymentFeignTimeout() {
        return paymentFeignService.paymentFeignTimeout();
    }
}
```



#### 超时控制

为了防止服务提供方没有响应以及响应后处理过长导致超时报错，可以根据业务场景设置超时时间，主要通过下面两个参数

- `connectTimeout` :prevents blocking the caller due to the long server processing time.
- `readTimeout` :is applied from the time of connection establishment and is triggered when returning the response takes too long.



对应配置文件中

```yaml
ribbon:
  ReadTimeout: 5000
  ConnectTimeout: 5000
```



### 日志功能

OpenFeign提供的日志级别有：

- `NONE`, No logging (**DEFAULT**).
- `BASIC`, Log only the request method and URL and the response status code and execution time.
- `HEADERS`, Log the basic information along with request and response headers.
- `FULL`, Log the headers, body, and metadata for both requests and responses.



修改配置文件

```yaml
logging:
  level:
    # feign日志以什么级别监控那个类下的接口
    com.ecifics.springcloud.service.PaymentFeignService: debug
```



通过相关配置类进行配置，例如

```java
@Configuration
public class FooConfiguration {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

