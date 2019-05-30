# Spring Data Redis

## 项目依赖

Spring Boot 工程下：

``` xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

非Spring Boot 工程下：

``` xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.0.2.RELEASE</version>
</dependency>
```

## SpringBoot Starter Redis 源码分析

spring-boot-autoconfigure.jar 包里面的 spring.factories 定义了 Spring Boot 默认加载的 AutoConfiguration，因此，打开 spring.factories 文件可以找到 Spring 自动加载了。

### 自动加载 JedisConnectionFactory

``` java
@Bean
@ConditionalOnMissingBean(RedisConnectionFactory.class)
public JedisConnectionFactory redisConnectionFactory()
        throws UnknownHostException {
    return applyProperties(createJedisConnectionFactory());
}
```

JedisConnectionFactory 可以自己配置也可以直接用 Spring Boot 提供的默认配置。

### 查看 createJedisConnectionFactory() 的具体方法

``` java
private JedisConnectionFactory createJedisConnectionFactory() {
    //这里会取我们配置文件里面的配置，如果没有配置，new 一个默认连接池
    JedisPoolConfig poolConfig = this.properties.getPool() != null
            ? jedisPoolConfig() : new JedisPoolConfig();

    //如果配置了Sentinel就取哨兵的配置直接返回
    if (getSentinelConfig() != null) {
        return new JedisConnectionFactory(getSentinelConfig(), poolConfig);
    }
    //如果没有配置中Sentinel，而配置了Cluster切片的配置方法，它就取Cluster的配置方法
    if (getClusterConfiguration() != null) {
        return new JedisConnectionFactory(getClusterConfiguration(), poolConfig);
    }
    //默认取连接pool的配置方法
    return new JedisConnectionFactory(poolConfig);
}
.......
//取配置文件里面的Pool的配置
private JedisPoolConfig jedisPoolConfig() {
    JedisPoolConfig config = new JedisPoolConfig();
    RedisProperties.Pool props = this.properties.getPool();
    config.setMaxTotal(props.getMaxActive());
    config.setMaxIdle(props.getMaxIdle());
    config.setMinIdle(props.getMinIdle());
    config.setMaxWaitMillis(props.getMaxWait());
    return config;
}
.......
//JedisPoolConfig的类默认构造函数
public class JedisPoolConfig extends GenericObjectPoolConfig {
    public JedisPoolConfig() {
        this.setTestWhileIdle(true);
        this.setMinEvictableIdleTimeMillis(60000L);
        this.setTimeBetweenEvictionRunsMillis(30000L);
        this.setNumTestsPerEvictionRun(-1);
    }
}
```

Spring 将 Pool 作为默认的配置方法：

* 在这三种配置中，Pool（连接池）、Sentinel（哨兵、master/slave）和Cluster（切片）只能选择一种来配置。
* 开发者仅需编写配置文件，其余由Spring的JedisConnectionFactory处理。

### 查看 RedisAutoConfiguration 的关键源码

``` java
@Configuration
@ConditionalOnClass({ JedisConnection.class, RedisOperations.class, Jedis.class })
@EnableConfigurationProperties(RedisProperties.class)
public class RedisAutoConfiguration {
......
}
```

``` java
@ConfigurationProperties(prefix = "spring.redis")
public class RedisProperties {
    /**
     * Database index used by the connection factory.
     */
    private int database = 0;
    /**
     * Redis url, which will overrule host, port and password if set.
     */
    private String url;
    /**
     * Redis server host.
     */
    private String host = "localhost";
    /**
     * Login password of the redis server.
     */
    private String password;
    /**
     * Redis server port.
     */
    private int port = 6379;
    /**
     * Enable SSL.
     */
    private boolean ssl;
    /**
     * Connection timeout in milliseconds.
     */
    private int timeout;

    private Pool pool;

    private Sentinel sentinel;

    private Cluster cluster;
    ......
}
```

###  RedisConfiguration 关键源码

``` java
    /**
     * Standard Redis configuration.
     */
    @Configuration
    protected static class RedisConfiguration {

        @Bean
        @ConditionalOnMissingBean(name = "redisTemplate")
        public RedisTemplate<Object, Object> redisTemplate(
                RedisConnectionFactory redisConnectionFactory)
                        throws UnknownHostException {
            RedisTemplate<Object, Object> template = new RedisTemplate<Object, Object>();
            template.setConnectionFactory(redisConnectionFactory);
            return template;
        }

        @Bean
        @ConditionalOnMissingBean(StringRedisTemplate.class)
        public StringRedisTemplate stringRedisTemplate(
                RedisConnectionFactory redisConnectionFactory)
                        throws UnknownHostException {
            StringRedisTemplate template = new StringRedisTemplate();
            template.setConnectionFactory(redisConnectionFactory);
            return template;
        }

    }
```

开发者可以使用类 RedisTemplate 或者 StringRedisTemplate 来直接操作 Redis 的 client 的相关 API。

## 配置方法

### 基本使用配置方法 & 连接池的配置方法

``` prop
spring.redis.host=127.0.0.1 #redis服务器地址
spring.redis.port=6379 #端口
spring.redis.timeout=6000 #连接超时时间 毫秒
spring.redis.pool.max-active=8 # 连接池的配置，最大连接激活数
spring.redis.pool.max-idle=8 # 连接池配置，最大空闲数
spring.redis.pool.max-wait=-1 #  连接池配置，最大等待时间
spring.redis.pool.min-idle=0 # 连接池配置，最小空闲活动连接数
```

实例 DEMO1：

``` java
//例如某个Service里面只需要引用RedisTemplate类即可：
@Autowired
private static RedisTemplate redisTemplate;
//某个service方法中，直接调用redisTemplate操作redis的set集合，储存key和value
public Object cacheAround(String key,String value) throws Throwable {
    ....
    //直接调用redisTemplate操作redis的set集合，储存key和value
    redisTemplate.opsForSet().add(key,value);
    //这里不需要，关心redisTemplate里面配置的是连接池，还是哨兵，还是cluster。
    .....
}
```

### sentinel 哨兵的高可用配置方法

``` prop
spring.redis.sentinel.master= redis_master_name #master 名字
spring.redis.sentinel.nodes= 127.0.0.1:63791,127.0.0.1:63792  #我们配置多个哨兵，用","分割即可
```

### Cluster 分布式切片的配置方法

``` prop
spring.redis.cluster.max-redirects=3 # Maximum number of redirects to follow when executing commands across the cluster.
spring.redis.cluster.nodes= 127.0.0.1:6379,127.0.0.1:6376,127.0.0.1:6378# Comma-separated list of "host:port" pairs to bootstrap from.
```

## 调用实例

DEMO2：通过 RedisTemplate 直接操作 key/value：

``` java
//声明
@Resource(name = "redisTemplate")
private RedisTemplate<String, String> template;

//调用方法
template.opsForValue().set("key","value");
```

DEMO3：通过注入 resdisTemplate 操作 ValueOperations：

``` java
//RedisTemplate还提供了对应的*OperationsEditor，用来通过RedisTemplate直接注入对应的Operation。
//声明
@Resource(name = "redisTemplate")
private ValueOperations<String, Object> vOps;

//调用方法
vOps.set("key","value");
```

DEMO4：通过注解 HashOperations 操作 Hash 数据结构：

``` java
//注入HashOperations对象
@Resource(name = "redisTemplate")
private HashOperations<String,String,Object> hashOps;

//具体调用
Map<String,String> map = new HashMap<String, String>();
map.put("value","code");
map.put("key","keyValue");
hashOps.putAll("hashOps",map);
```

DEMO5：通过 template 操作 Hash 数据结构：

``` java
//注入RedisTemplate对象
@Resource(name = "redisTemplate")
private RedisTemplate<String, String> template;

//具体调用
Map<String,String> map = new HashMap<String, String>();
map.put("value","code");
map.put("key","keyValue");
template.opsForHash().putAll("hashOps",map);
```

## 概况 & API

### 主要的几个类 & 简单用法介绍

![redis class graph](img/04c29d90-e7cb-11e7-8d56-2d95710ec169)

![redis class graph](img/0a289aa0-e7cb-11e7-a510-4914b777019d)

1. JedisConnectionFactory 里面依赖了 JedicConnection 和 JedisPoolConfig、RedisSentineConfiguration、RedisCLusterConfiguration 的三种配置方法。
2. RedisAutoConfiguration 自动加载了 RedisProperties 的配置文件。
3. RedisTemplate 抽象保证了 Redis 相关的 Operations 方法。
4. StringRedisTemplate 继承和扩展了 RedisTemplate，提供了一种扩展思路。

``` java
redisTemplate.opsForValue();//操作字符串
redisTemplate.opsForHash();//操作 hash
redisTemplate.opsForList();//操作 list
redisTemplate.opsForSet();//操作 set
redisTemplate.opsForZSet();//操作有序 set
```

## Spring Cache

Spring 3.1 之后引入了基于注释（annotation）的缓存（cache）技术，它本质上不是一个具体的缓存实现方案（如 EHCache 或者 Redis），而是一个对缓存使用的抽象，通过在既有代码中添加少量它定义的各种 annotation，即能够达到缓存方法的返回对象的效果。

Spring 的缓存技术还具备相当的灵活性，不仅能够使用 SpEL（Spring Expression Language）来定义缓存的 key 和各种 condition，还提供开箱即用的缓存临时存储方案，也支持和主流的专业缓存例如 Redis、EHCache 集成。

Spring 4.1 之后又改进了很多，Spring Cache 属于 Spring framework 的一部分，在如下这个包里面：

![Spring Cache Context Jar](img/bc294c90-e7cb-11e7-8d56-2d95710ec169)

Spring 定义了 org.springframework.cache.CacheManager 和 org.springframework.cache.Cache 接口用来统一不同的缓存的技术。

``` java
public interface CachingConfigurer {
   CacheManager cacheManager();
   CacheResolver cacheResolver();
   KeyGenerator keyGenerator();
   CacheErrorHandler errorHandler();
}
```

Cache 接口包含缓存的各种操作（增加、删除、获得缓存），Cache 的默认的缓存的实现如下：

![spring cache son classes](img/c1d923e0-e7cb-11e7-b355-bb630cc29421)

 **CacheManager** 

``` java
public interface CacheManager {
   Cache getCache(String name);
   Collection<String> getCacheNames();
}
```

CacheManager 是 Spring 提供的各种缓存技术抽象接口，通过它管理 Spring Framework 里面默认的实现的 CacheManager 有如下内容：

![Spring Framework CacheManager](img/c9993d90-e7cb-11e7-87f9-653243442f6c)

CacheResolver 解析器，用于根据实际情况来动态解析使用哪个 Cache，默认实现的有如下：

![CacheResolver](img/cf68d1e0-e7cb-11e7-a510-4914b777019d)

KeyGenerator 当我们使用 Cache 注解的时候，默认 key 的生成规则：

![KeyGenerator](img/d4939ab0-e7cb-11e7-8d56-2d95710ec169)

### 主要的注解

`@Cacheable` 应用到读取数据的方法上，即可缓存的方法，如查找方法：先从缓存中读取，如果没有再调用方法获取数据，然后把数据添加到缓存中。

``` java
public @interface Cacheable {
   @AliasFor("cacheNames")
   String[] value() default {};
//cache的名字。可以根据名字设置不同cache处理类。redis里面可以根据cache名字设置不同的失效时间。
   @AliasFor("value")
   String[] cacheNames() default {};
//缓存的key的名字，支持spel
   String key() default "";
//key的生成策略，不指定可以用全局的默认的。
   String keyGenerator() default "";
   //客户选择不同的CacheManager
   String cacheManager() default "";
   //配置不同的cache resolver
   String cacheResolver() default "";
   //满足什么样的条件才能被缓存，支持SpEL，可以去到方面名，参数
   String condition() default "";
//排除哪些返回结果不加入到缓存里面去，支持SpEL，实际工作中常见的是result ==null等
   String unless() default "";
   //是否 同步读取缓存，更新缓存
   boolean sync() default false;
}
```

DEMO：

``` java
@Cacheable(cacheNames="book", condition="#name.length() < 32", unless="#result.hardback")
   public Book findBook(String name)

@CachePut
```

调用方法时会自动把相应的数据放入缓存，它与 `@Cacheable` 不同的是所有注解的方法每次都会执行，一般配置在 `Update` 和 `Insert` 方法上。看了一下源码里面的字段和用法与 `@Cacheable` 相同，只是使用场景不一样。

`@CacheEvict`：删除缓存，一般配置在删除方法上面。

``` java
public @interface CacheEvict {
//与@Cacheable相同的部分咱们就不重复叙述了。
......
//是否删除所有的实体对象
   boolean allEntries() default false;
   //是否方法执行之前执行。默认在方法调用成功之后删除
   boolean beforeInvocation() default false;
}
```

@Caching：所有 Cache 注解的组合配置方法。

``` java
public @interface Caching {
   Cacheable[] cacheable() default {};
   CachePut[] put() default {};
   CacheEvict[] evict() default {};
}
```

- @CacheConfig：全局 Cache 配置。
- @EnableCaching：开启 SpringCache 的默认配置。

### Spring Boot 快速开始 Demo

#### 添加依赖

``` xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

#### 添加 @EnableCaching 注解开启 Caching

``` java
@SpringBootApplication
@EnableJpaRepositories
@EnableCaching
public class RedisApplication {
   public static void main(String[] args) {
      SpringApplication.run(RedisApplication.class, args);
   }
}
```

#### 使用的地方直接用 @Cacheable、@CachePut 等注解

``` java
@Controller
@RequestMapping("hello")
public class UserInfoController {
    @Autowired
    private UserInfoService userInfoService;
    @RequestMapping("users")
    @ResponseBody
    @Cacheable(value = "user")
    public List<UserInfoEntity> findAll(){
        System.out.println("user........");
        return userInfoService.findAll();
    }
}
```

### 实现过程解析

SpringBoot 会默认加载
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration 配置文件，spring-boot-starter-cache 这个 jar 还会帮我们加载 Spring Cache 所需要的其他 jar 包，如 spring-context 和 spring-context-support，是 Spring Cache 的核心 jar 包。

#### CacheAutoConfiguration 关键源码解读

``` java
@Configuration
@ConditionalOnClass(CacheManager.class)
@ConditionalOnBean(CacheAspectSupport.class)
@ConditionalOnMissingBean(value = CacheManager.class, name = "cacheResolver")
@EnableConfigurationProperties(CacheProperties.class)
@AutoConfigureBefore(HibernateJpaAutoConfiguration.class)
@AutoConfigureAfter({ CouchbaseAutoConfiguration.class, HazelcastAutoConfiguration.class,
      RedisAutoConfiguration.class })
@Import(CacheConfigurationImportSelector.class)
public class CacheAutoConfiguration {
   static final String VALIDATOR_BEAN_NAME = "cacheAutoConfigurationValidator";
   @Bean
   @ConditionalOnMissingBean
   public CacheManagerCustomizers cacheManagerCustomizers(
         ObjectProvider<List<CacheManagerCustomizer<?>>> customizers) {
      return new CacheManagerCustomizers(customizers.getIfAvailable());
   }
....}
```

**CacheConfigurationImportSelector**

``` java
static class CacheConfigurationImportSelector implements ImportSelector {
   @Override
   public String[] selectImports(AnnotationMetadata importingClassMetadata) {
      CacheType[] types = CacheType.values();
      String[] imports = new String[types.length];
      for (int i = 0; i < types.length; i++) {
         imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
      }
      return imports;
   }
}
```

CacheType 的内容如下：

``` java
package org.springframework.boot.autoconfigure.cache;
public enum CacheType {
    GENERIC,
    JCACHE,
    EHCACHE,
    HAZELCAST,
    INFINISPAN,
    COUCHBASE,
    REDIS,
    CAFFEINE,
    /** @deprecated */
    @Deprecated
    GUAVA,
    SIMPLE,
    NONE;

    private CacheType() {
    }
}
```

当我们没有显示手动的指定 CacheManager 或者 CacheResolver 的时候，Spring Boot Cache 会按照以下顺序查找 Cache 的默认实现者：

- Generic
- JCache (JSR-107) (EhCache 3, Hazelcast, Infinispan, etc)
- EhCache 2.x
- Hazelcast
- Infinispan
- Couchbase
- Redis
- Caffeine
- Guava (deprecated)
- Simple

并会自动导入各大提供商的 config。

## Spring Cache 和 Spring Data Redis 实践

### quick start 实现

#### 引入依赖

``` xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>
```

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

#### 配置properties

``` prop
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.timeout=6000
spring.redis.pool.max-active=8
spring.redis.pool.max-idle=8
spring.redis.pool.max-wait=-1
spring.redis.pool.min-idle=0
```

默认使用Redis的Pool配置方式。

#### 添加@EnableCaching 注解，开启 cache

``` java
@SpringBootApplication
@EnableJpaRepositories
@EnableCaching
public class RedisApplication {
    public static void main(String[] args) {
        SpringApplication.run(RedisApplication.class, args);
    }
}
```

#### 直接使用Spring Data Cache 的注解。

``` java
@Controller
@RequstMapping("hello")
public class UserInfoController {
    @AutoWired
    private UserInfoService userInfoService;

    @RequestMapping("users")
    @ResponseBody
    @Cacheable(value = "user")
    public List<UserInfoEntity> findAll() {
        System.out,pringtln("user.....");
        return userInfoService.findAll();
    }
}
```

#### 验证结果

- curl http://127.0.0.1:8080/hello/users  就会发现，第一次会进入到 Controller 的 method 里面，第二次就不进去了。
- 我们打开 redis-client 可以看到，Reids 的 Server 端已经有我们的缓存了。

### Spring Boot 实现

- 当引入 spring-boot-starter-data-redis 的时候，会自动把 Spring Data Redis 的 jar 也加入进来，同时会激活 RedisCacheConfiguration，把 RedisCacheManager 给默认加载进来。
- 当引入 spring-boot-starter-cache 的时候，会自动加载 CacheAutoConfiguration，并且里面 CacheType 有顺序，这时候会把 RedisCacheManager 给激活。

![CacheManager](img/ca5f68b0-f2cc-11e7-aff8-abc1215f4b20)

当打断点在这个类上的时候，会发现 cacheManager 这时候会变成 RedisCacheManager，而不是默认的 Cache Manager。

### 生产使用

#### 实现自定义的配置

1. 新建RedisConfiguration，并配置两个CacheManager。

   ``` java
   /**
    * 自定义 RedisConfiguration
    * 通过@AutoConfigureAfter告诉其在RedisAutoConfiguration之后加载
    * @EnableConfigurationProperties（CacheProperties.class）依赖CacheProperties的配置，无需开发者自己编写配置。
    */
   @Configuration
   @AutoConfigureAfter(RedisAutoConfiguration.class)
   @EnableCOnfigurationProperties(CacheProperties.class)
   public class RedisConfiguration extends CachingConfigurerSupport {
       /*
        * 依赖cache的配置文件
        */
       @Autowired
       private CacheProperties cacheProperties;

       /**
        * 配置一个以RedisTemplate为调用栈的CacheManager
        * 通过@Primary告诉Spring cache哪个是默认的
        *
        * @param redisTemplate
        * @return
        */
       @Bean(name = "redisCacheManager")
       @Primary
       public RedisCacheManager cacheManager(RedisTemplate<Object, Object> redisTemplate) {
           RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
           cacheManager.setUsePrefix(true);
           // 可以设置不同的cache的name不同的过期时间
           Map<String, Long> expries = Maps.newHashMap();
           expries.put("user:", 10 * 60L);
           cacheManager.setExpries(expries);
           List<String> cacheNames = cacheProperties.getCacheNames();
           if (!cacheNames.isEmpty()) {
               cacheManager.setCacheNames(cacheNames);
           }
           return cacheManager;
       }

       @Bean(name = "redisCacheManagerString")
       public RedisCacheManager cacheManagerString(StringRedisTemplate stringRedisTemplate) {
           RedisCacheManager cacheManager = new RedisCacheManager(stringRedisTemplate);
           cacheManager.setUserPrefix(true);
           // 设置默认过期时间
           cacheManager.setDefaultExpiration(10 * 60);
           // cache name
           List<String> cacheNames = cacheProperties.getCacheNames();
           if (!cacheNamse.isEmpty()) {
               cacheManager.setCacheNames(cacheNames);
           }
           return cacheManager;
       }
   }
   ```

   由于RedisTemplate和StringRedisTemplate默认的：

   ``` java
   setKeySerializer(stringSerializer);
   setValueSerializer(stringSerializer);
   setHashKeySerializer(stringSerializer);
   setHashValueSerializer(stringSerializer);
   ```

   都不相同，所以需要分别设置两个CacheManager。通过配置 `@Cacheable(value = "user",cacheManager = "redisCacheManagerString")` 可以选择用哪个 cacheManager。

   通过自定义 CacheManager 实现以下功能：

   * defaultExpiration：默认过期时间是不一样的。
   * 不同的 Cache 的 Name 我们也可以定制化，不一样的过期时间。
   * 不同的 Cache 我们可以选择不同的 RedisTemplate。

2. 自定义 KeyGenerator 覆盖默认的 Cache key 生成规则，只需要在 RedisConfiguration 中增加如下配置即可。

   ``` java
   /**
    * 覆盖默认的key的生成器
    *
    * @return
    */
   @Override
   @Bean
   public KeyGenerator KeyGenerator() {
       return new KeyGenerator() {
           @Override
           public Object generate(Object o, Method method, Object... objects) {
               // This will generate a unique key of the class name, the method name,
               // and all method parameters appended.
               StringBuilder sb = new StringBuilder("Jack-Test:");
               sb.append(o.getClass().getName());
               sb.append(method.getName());
               for (Object obj : objects) {
                   sb.append(obj.toString());
               }
               return sb.toString();
           }
       };
   }
   ```

3. 修改 RedisTemplate 和 StringRedisTemplate 的 KeySerializer 和 ValueSerializer 默认规则。

   ``` java
   /**
    * 修改key serializer 一般建议key的类型使用String，易懂
    * @param redisConnectionFactory
    * @return
    */
   @Bean
   @ConditionalOnMissingBean(name = "redisTemplate")
   public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
       RedisTemplate<Object, Object> template = new RedisTemplate<>();
       template.setConnectionFactory(redisConnectionFactory);
       template.setKeySerializer(new StringRedisSerializer());
       return template;
   }

   /**
    * 一样，修改KeySerializer用String类型
    * ValueSerializer用Json来存储
    * @param redisConnectionFactory
    * @return
    */
   @Bean
   @ConditionalOnMissingBean(StringRedisTemplate.class)
   public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
       StringRedisTemplate template = new StringRedisTemplate();
       template.setConnectionFactory(redisConnectionFactory);
       template.setKeySerializer(new StringRedisSerializer());
       template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
       return template;
   }
   ```

4. 使用的地方 controller 配置如下：

   ``` java
   @RequestMapping("users")
   @ResponseBody
   @Cacheable(value = "user", cacheManager = "redisCacheManagerString")
   public List<UserInfoEntity> findAll() {
       System.out.println("user......");
       return userInfoService.findAll();
   }
   ```

5. 调用 http://127.0.0.1:8080/hello/users 得到的结果如下：

   ![users](img/04538790-f2cd-11e7-9b48-db6544342c73)

* 当使用 `@Cacheable` 的注解的时候一定要加 `condition` 和 `unless`，对结果和查询条件进行过滤。这样可以避免一些用户体验不好的地方。

## Redis Client 的使用及命令

### Redis Desktop Manager 开源工具

Redis Desktop Manager 是一个支持多个操作系统的，开源的，Redis Ui 工具。支持 Windows 10、Mac OS X 10.12、Ubuntu 16、LinuxMint | Fedora | CentOS | OpenSUSE。

1. Git Hub 地址，详见 [这里](https://github.com/uglide/RedisDesktopManager)。

2. 下载 Git hub release 里面的安装文件。

3. 安装完毕，打开如下图，单击 connect to redis server 按钮，创建链接。

   ![connect to redis server](img/7c377400-f37d-11e7-b8fe-b9e896ac18af)

4. 实际使用如下图：

   ![Redis Desktop Manager](img/843dbe70-f37d-11e7-aa2d-b79d04446197)

5. 查看 Redis 系统使用基本信息：

   ![Redis Info](img/8955e9a0-f37d-11e7-8db8-efc5b28eb646)

   ![Redis Info](img/8daff4f0-f37d-11e7-8db8-efc5b28eb646)

6. 通过控制台可以做的事情有：

   - 看清楚通过 Spring Data Redis 创建的所有的 key 和 value 是什么样子的。
   - 可以直接对 key、value 进行界面上的操作（添加、删除、更新、刷新）。
   - 可以直接打开 Redis Client（类似 src/redis-cli）。
   - 非常清楚的看到 Redis Server 上的 redis db0 db1 ....等库的分配和使用情况（会发现其实大部分公司都是使用的 db0，其实不同的业务可以放在不同的 db 里面，这里逻辑比较清晰，也不会有重复覆盖问题）。
   - 可以看到 RedisServer 的监控信息。

7. 工具存在的不足之处：

   - Sentinel 哨兵模式支持的不是特别好。
   - master/savler 模式支持的不是特别好，要自己逐个连接上去才知道。
   - pool 的监控也比较少。

### 常用的命令

命令行可以通过两种方式使用：

- Redis 安装目录下面的 src 目录里面的 redis-cli。
- 上面工具提到的控制台（推荐使用这个，这样可以实时看到自己的操作变化）。

**操作 Redis 的简单的数据结构命令如下：**

1. 切换数据库（默认 Redis 开启16个库）可以同过如下命令切换数据库

   ``` bash
   127.0.0.1:0>select 2
   "OK"
   127.0.0.1:0>select 1
   "OK"
   ```

2. Redis Strings 类型操作

   ``` bash
   127.0.0.1:1>set key1 jack
   "OK"
   127.0.0.1:1>get key1
   "jack"
   ```

3. Redis lists 基本类型操作

   - `LPUSH` 命令可向 list 的左边（头部）添加一个新元素。
   - `RPUSH` 命令可向 list 的右边（尾部）添加一个新元素。
   - `lrange` 取 list 元素。

   ``` bash
   127.0.0.1:1>rpush listdemo a
   "1"
   127.0.0.1:1>rpush listdemo B
   "2"
   127.0.0.1:1>rpush listdemo 3
   "3"
   127.0.0.1:1>lrange listdemo 0 -1
   1) "a"
   2) "B"
   3) "3"
   ```

4. Redis hash 类型操作

   ``` bash
   127.0.0.1:1>hmset user:1 name jack age 20 address shanghai
   "OK"
   127.0.0.1:1>hget user:1 name
   "jack"
   127.0.0.1:1>hgetall user:1
    1)  "name"
    2)  "jack"
    3)  "age"
    4)  "20"
    5)  "address"
    6)  "shanghai"
   ```

5. Redis sorted set 有序集合操作

   ``` bash
   127.0.0.1:1>zadd linkedset 1 redis
   "1"
   127.0.0.1:1>zadd linkedset 2 mongodb
   "1"
   127.0.0.1:1>zadd linkedset 3 mysql
   "1"
   127.0.0.1:1>zrange linkedset 0 -1
    1)  "redis"
    2)  "mongodb"
    3)  "mysql"
   ```

6. 总结

   当调用 RedisTemplate 和 StringRedisTemplate 相关的 API 的时候其实内部就是 socket 直接发送的 Redis Commands 到 Redis Server 上的。

   ![RedisTemplate zadd](img/91f58de0-f37d-11e7-b8fe-b9e896ac18af)

> 更多命令参阅 [Redis 官方命令](https://redis.io/commands)。

### 高危险命令

1. 删除当前选择数据库中的所有 key：`flushdb`
2. 删除所有的数据库中的所有 key：`flushall`

> 一般以上两个命令是禁止在生产直接使用的，高并发情况下，会引起大量缓存失效，如果用 token 的话，可能会导致大量用户被踢出。

### Redis 监控

1. 管理员的监控命令

   ``` bash
   # 查看统计信息
   127.0.0.1:1>info
   # 查看客户端列表
   127.0.0.1:1>client list
   # 获取慢查询
   127.0.0.1:1>showlog get 10
   # 检查连接数
   127.0.0.1:1>info total_connections_received
   ```

> 更多命令参阅 Gitbooks 地址：[管理员命令大全](https://gnuhpc.gitbooks.io/redis-all-about/Problem/latency/total-commands.html)

2. redis-stat 推荐

   redis-stat 是一个 Github 上开源的 redis-server 的监控工具，Github 地址：<https://github.com/junegunn/redis-stat>。

   实际运行的效果如下：

   ![redis-stat](img/9ad0db90-f37d-11e7-90f1-5d6806af6067)

   控制台如下：

   ![redis-stat DashBoard](img/a0ba1260-f37d-11e7-8db8-efc5b28eb646)

## 分布式环境下 Redis 如何正确使用

### Redis 本身的高可用

在分布式环境中，Redis 本身的高可用，都是来突破单个机器的极限的，无非分为两种：

- 备份/Master:Slave 模式
- 切片模式

#### 官方 Sentinel 模式

1. 发展

   期初，Redis 官方只提供了 Master 和 Slave，用于提供主存和备份的模式，可以很好的做读写分离，但是官方没有提供故障转移的功能，所以当我们使用的时候不仅要考虑读写分离，也应该要考虑发生故障的时候如何切换到 Slave，并且把它选成 Master，后来 Redis 官方推出来了 Sentinel 哨兵的模式，帮我们解决了如下的四个问题：

   - 监控（Monitoring）：Sentinel 不断检查你的主从实例是否运转正常。
   - 通知（Notification）：Sentinel 可以通过 API 来通知系统管理员，或者其他计算机程序，被监控的 Redis 实例出了问题。
   - 自动故障转移（Automatic failover）：如果一台主服务器运行不正常，Sentinel 会开始一个故障转移过程，将从服务器提升为主服务器，配置其他的从服务器使用新的主服务器，使用 Redis 服务器的应用程序在连接时会收到新的服务器地址通知。
   - 配置提供者（Configuration provider）：Sentinel 充当客户端服务发现的权威来源，客户端连接到 Sentinel 来询问某个服务的当前 Redis 主服务器的地址。当故障转移发生时，Sentinel 会报告新地址。

2. 配置方法

   - Redis Server 端的 Sentinel 的配置[请参考官方的配置方式](https://redis.io/topics/sentinel)。
   - Spring Data Redis 里面的 Sentinel 配置参考上文。

3. 原理

   - 客户端其实使用了代理的思路，将直接操作 Redis 的命令交给了 Sentinel 给我们提供的连接地址。
   - 而 Sentinel 利用心跳检查，来实时监听原本的 master/slave，实现服务端内部的切换。

   ![img](img/3923b390-fdd2-11e7-a772-21bfb93cfbfb)

#### 官方 cluster 模式

1. Redis 集群解决的问题

   - 自动分割数据到不同的节点上。
   - 整个集群的部分节点失败或者不可达的情况下能够继续处理命令。

2. 原理

   Redis 集群有16384个哈希槽，每个 key 通过$CRC16$ 校验后对 16384 取模来决定放置哪个槽，从而可以放到不同的切片服务器上。

3. 配置使用方法

   - Redis Server 端的 cluster 的配置请参考官方的配置方式： [Redis 官方英文文档](https://redis.io/topics/cluster-tutorial) /  [Redis 官方中文文档](http://www.redis.cn/topics/cluster-tutorial.html)。
   - Spring Data Redis 里面的 cluster 配置参考上文。
   - 还有一种选择：可以按照不同的业务线拆分，分别连不同的 Redis 的 master/slave 组合。

#### 业内集群解决不错的开源产品

- Twitter 的 twemproxy，Github 地址[请见这里](https://github.com/twitter/twemproxy)。
- 豆瓣的 codis，[Github 地址详见这里](https://github.com/CodisLabs/codis)。

#### 成熟云的选择

如果当公司的技术实力跟不上的时候也可以选择一些成熟的云平台的产品。

- 阿里云的云数据库 Redis 版。
- AWS（亚马逊云）的 elasticache 中添加 Redis。

### 缓存雪崩&缓存击穿&缓存穿透

高并发的环境下，我们使用 Redis 的时候最可能碰到的的问题缓存雪崩、缓存击穿、缓存穿透三大问题。

#### 缓存雪崩

缓存雪崩有时又称缓存并发。其实是指在我们设置缓存时，大多数的 Key 采用了相同的过期时间，导致缓存在某一时刻同时失效，请求全部转发到 DB，DB 瞬时压力过重雪崩。

> **解决方法：**不同的 key 设置不同的过期时间，尽量不要通过后台批量创建缓存数据。具体方法参考上文，通过 RedisCacheManager 设置不同的 cacheName 的过期时间即可。

#### 缓存击穿

于一些设置了过期时间的 key，如果这些 key 可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题，这个和缓存雪崩的区别在于这里针对某一key 缓存，前者则是很多 key。

缓存在某个时间点过期的时候，恰好在这个时间点对这个 Key 有大量的并发请求过来，这些请求发现缓存过期一般都会从后端 DB 加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端 DB 压垮。 

> **解决方法：**这个时候用同步就行了，`@Cacheable(key = "test",sync = true)`。

#### 缓存穿透

我们在项目中使用缓存通常都是先检查缓存中是否存在，如果存在直接返回缓存内容，如果不存在就直接查询数据库然后再缓存查询结果返回。这个时候如果我们查询的某一个数据在缓存中一直不存在，就会造成每一次请求都查询 DB，这样缓存就失去了意义，在流量大时，可能 DB 就挂掉了，要是有人利用不存在的 key 频繁攻击我们的应用，这就是漏洞。

> **解决方法：**
>
> - 一般是配置一个 `@Aspect`，针对某些方法，不符合规则的参数，就直接返回找不到结果了。
> - 有一个比较巧妙的作法是，可以将这个不存在的 key 预先设定一个值，如“key”、“&&”。在返回这个 && 值的时候，我们的应用就可以认为这是不存在的 key，那我们的应用就可以决定是否继续等待继续访问，还是放弃掉这次操作。如果继续等待访问，过一个时间轮询点后，再次请求这个 key，如果取到的值不再是 &&，则可以认为这时候 key 有值了，从而避免了传递到数据库，从而把大量的类似请求挡在了缓存之中。

### 在分布式环境中的应用

在分布式的系统中，还可以使用 Redis 解决一些分布式环境下的问题：

1. 分布式锁的问题

   在分布式的环境中，Redis 还可以很好的解决分布式锁的问题，要用到 Redis 里面的 `SETNX` 命令，命令格式 `SETNX key value`。

   - `SETNX` 是 `SET If Not eXists` 的简写。
   - 将 key 的值设为 value，当且仅当 key 不存在。
   - 若给定的 key 已经存在，则 `SETNX` 不做任何动作。

   ``` bash
   redis> SETNX mykey “hello”
   (integer) 1
   redis> SETNX mykey “hello”
   (integer) 0
   ```

   RedisTemplate 的用法如下：

   ``` java
   @Autowired
   private RedisTemplate redisTemplate;
   public void someMethod(Object param){
      ......
      redisTemplate.opsForValue().setIfAbsent(key,lockValue);
   }
   ```

2. 分布式计数器

   在分布式的环境中，如果是分布式数据库，经常会用到自增的 ID，各个库里面不能保持有序自增。Redis 里面帮我们提供了 `INCR` 的命令，命令格式 `INCR key`。

   - 将 key 中储存的数字值增一。
   - 如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 `INCR` 操作。

   ``` bash
   redis> SET page_view 20
   OK
   redis> INCR page_view
   (integer) 21
   redis> GET page_view    # 数字值在 Redis 中以字符串的形式保存
   "21"
   ```

   RedisTemplate 的用法如下：

   ```  java
   @Autowired
   private RedisTemplate redisTemplate;
   public void someMethod(Object param){
      ......
      redisTemplate.opsForValue().increment(key,delta);
   }
   ```

## Redis 原理分析

### Redis 服务器端原理

1. Redis 内部储存的核心对象 RedisObject

   ![redisObject](img/1fb47420-fe78-11e7-834e-1f926ad802e1)

   Redis 内部使用一个 redisObject 对象来表示所有的 key 和 value，redisObject 最主要的信息如图所示： 

   - type 代表一个 value 对象具体是何种数据类型。
   - encoding 是不同数据类型在 Redis 内部的存储方式。
   - vm 字段，只有打开了 Redis 的虚拟内存功能，此字段才会真正的分配内存，该功能默认是关闭状态的。

   下图展示了 redisObject 、Redis 所有数据类型以及 Redis 所有编码方式（底层实现）三者之间的关系：

   ![redisObject 数据结构关系图](img/2dbab1b0-fe78-11e7-834e-1f926ad802e1)

2. 数据类型介绍

   * String（字符串类型的 Value） 

     String，可以 String 字符串，也可是是任意的 byte[] 类型的数组，如图片等。String 在 Redis 内部存储默认就是一个字符串，此时 redisObject 的 type=string，value 存储的是一个普通字符串，那么对应的 encoding 可以是 raw 或者是 int，如果是 int 则代表实际 Redis 内部是按数值型类存储和表示这个字符串的，当然前提是这个字符串本身可以用数值表示，比如："123" "456"这样的字符串。而当遇到 `incr`、`decr` 等操作时会转成数值型进行计算，此时 redisObject 的 encoding 字段为 int。

   * List（List 类型的 Value）

     Redis list 的实现为一个双向链表，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销，Redis 内部的很多实现，包括发送缓冲队列等也都是用的这个数据结构。而此时 redisObject 的 type 属性为`REDIS_LIST`，encoding 属性为`REDIS_ENCODING_LINKEDLIST`，它的值保存在一个双端链表内，而 ptr 指针就指向这个双端链表。

   * Hash（连表结构）

     Redis Hash 对应 Value 内部实际就是一个类似 HashMap 的数据结构，实际上 Hash 的成员比较少时 Redis 为了节省内存会采用类似一维数组的方式来紧凑存储，而不会采用真正的 HashMap 结构，而是使用 ZIPLIST。

     当使用`REDIS_ENCODING_ZIPLIST`编码哈希表时，程序通过将键和值一同推入压缩列表，从而形成保存哈希表所需的键-值对结构：

     ![REDIS_ENCODING_ZIPLIST](img/360c8410-fe78-11e7-9f1b-09e589e39a81)

     新添加的 key-value 对会被添加到压缩列表的表尾。

     此时 redisObject 的 type 属性为 `REDIS_HASH` ， encoding 属性为 `REDIS_ENCODING_ZIPLIST` ，那么这个对象就是一个 Redis 哈希表，它的值保存在一个 zipList 里，而 ptr 指针就指向这个 zipList ；诸如此类。

   * Set/Sorted Set 

     set 的内部实现是一个类 Hash 和跳跃表（SkipList）来保证数据的存储和有序，实际就是通过计算 hash 的方式来快速排重的，这也是 set 能提供判断一个成员是否在集合内的原因。

3. Redis的持久化机制

   Redis 由于支持非常丰富的内存数据结构类型，如何把这些复杂的内存组织方式持久化到磁盘上是一个难题，所以 Redis 的持久化方式与传统数据库的方式有比较多的差别，Redis 一共支持两种持久化方式，分别是：

   - 定时快照方式（snapshot）

     该持久化方式实际是在 Redis 内部一个定时器事件，每隔固定时间去检查当前数据发生的改变次数与时间是否满足配置的持久化触发的条件，如果满足则通过操作系统 fork 调用来创建出一个子进程，这个子进程默认会与父进程共享相同的地址空间，这时就可以通过子进程来遍历整个内存来进行存储操作，而主进程则仍然可以提供服务，当有写入时由操作系统按照内存页（page）为单位来进行 copy-on-write 保证父子进程之间不会互相影响。

     该持久化的主要缺点是定时快照只是代表一段时间内的内存映像，所以系统重启会丢失上次快照与重启之间所有的数据。

   - 基于语句追加文件的方式（aof）

     aof 方式实际类似 MySQL 的基于语句的 binlog 方式，即每条会使 Redis 内存数据发生改变的命令都会追加到一个 log 文件中，也就是说这个 log 文件就是 Redis 的持久化数据。

     aof 的方式的主要缺点是追加 log 文件可能导致体积过大，当系统重启恢复数据时如果是 aof 的方式则加载数据会非常慢，几十G的数据可能需要几小时才能加载完，当然这个耗时并不是因为磁盘文件读取速度慢，而是由于读取的所有命令都要在内存中执行一遍。另外由于每条命令都要写 log，所以使用 aof 的方式，Redis 的读写性能也会有所下降。

4. Redis 主从复制原理

   ![redis主从复制原理图](img/3c6a4ea0-fe78-11e7-9f1b-09e589e39a81)

   * 当启动一个 Slave 进程后，它会向 Master 发送一个 SYNC Command，请求同步连接。无论是第一次连接还是重新连接，Master 都会启动一个后台进程，将数据快照保存到数据文件中，同时 Master 会记录所有修改数据的命令并缓存在数据文件中。
   * 后台进程完成缓存操作后，Master 就发送数据文件（dump.rdb）给 Slave，Slave 端将数据文件保存到硬盘上，然后将其在加载到内存中，接着 Master 就会所有修改数据的操作，将其发送给 Slave 端。 
   * 若 Slave 出现故障导致宕机，恢复正常后会自动重新连接，Master 收到 Slave 的连接后，将其完整的数据文件发送给 Slave，如果 Mater 同时收到多个 Slave 发来的同步请求，Master 只会在后台启动一个进程保存数据文件，然后将其发送给所有的 Slave，确保 Slave 正常。

   **Redis 复制工作原理：**

   * 如果设置了一个 Slave，无论是第一次连接还是重连到 Master，它都会发出一个 SYNC 命令；
   * 当 Master 收到 SYNC 命令之后，会做两件事：
     - Master 执行 BGSAVE，即在后台保存数据到磁盘（rdb 快照文件）；
     - Master 同时将新收到的写入和修改数据集的命令存入缓冲区（非查询类）；
   * 当 Master 在后台把数据保存到快照文件完成之后，Master 会把这个快照文件传送给 Slave，而 Slave 则把内存清空后，加载该文件到内存中；
   * 而 Master 也会把此前收集到缓冲区中的命令，通过 Reids 命令协议形式转发给 Slave，Slave 执行这些命令，实现和 Master 的同步；
   * Master/Slave 此后会不断通过异步方式进行命令的同步，达到最终数据的同步一致。

### RedisTemplate 原理

1. Spring Data Redis 简单源码解读

   - spring-data-redis 提供了 Redis 操作的封装和实现；
   - RedisTemplate 模板类封装了 Redis 连接池管理的逻辑，业务代码无须关心获取，释放连接逻辑；
   - Spring Redis 同时支持了 Jedis、Jredis、rjc 客户端操作。

   Spring Redis 源码设计逻辑可以分为以下几个方面：

   - Redis 连接管理：封装了 Jedis、Jredis、Rjc 等不同 Redis 客户端连接；
   - Redis 操作封装：value、list、set、sortset、hash 划分为不同操作；
   - Redis 序列化：能够以插件的形式配置想要的序列化实现；
   - Redis 操作模板化：Redis 操作过程分为获取连接、业务操作、释放连接，模板方法使得业务代码只需要关心业务操作；
   - Redis 事务模块：在同一个回话中，采用同一个 Redis 连接完成。

   ![spring data redis 源码](img/45aa5190-fe78-11e7-9f1b-09e589e39a81)

2. 实例解读

   （1）redisTemplate 方法：

   ``` java
   redisTemplate.opsForValue().set("key","value");
   ```

   （2）调用到的 ValueOperations 源码实现：

   ``` java
   class DefaultValueOperations<K, V> extends AbstractOperations<K, V> implements ValueOperations<K, V> {
      DefaultValueOperations(RedisTemplate<K, V> template) {
         super(template);
      }
   public void set(K key, V value) {
      final byte[] rawValue = rawValue(value);
      execute(new ValueDeserializingRedisCallback(key) {
         protected byte[] inRedis(byte[] rawKey, RedisConnection connection) {
            connection.set(rawKey, rawValue);
            return null;
         }
      }, true);
   }
   ......}
   ```

   （3）找到 RedisConnection 的实现类，我们查看 RedisAutoConfiguration 可以发现是 JedisConnectionFactory：

   ``` java
   @ConditionalOnClass({JedisConnection.class, RedisOperations.class, Jedis.class})
   @EnableConfigurationProperties({RedisProperties.class})
   public class RedisAutoConfiguration{
   ......
   @ConditionalOnMissingBean({RedisConnectionFactory.class})
   public JedisConnectionFactory redisConnectionFactory() throws UnknownHostException {
       return this.applyProperties(this.createJedisConnectionFactory());
   }
   ......//
   }
   ```

   （4）打开 JedisConnection 分析关键代码如下：

   ``` java
   package org.springframework.data.redis.connection.jedis;
   public class JedisConnection extends AbstractRedisConnection {
   public void set(byte[] key, byte[] value) {
      try {
         if (isPipelined()) {
            pipeline(new JedisStatusResult(pipeline.set(key, value)));
            return;
         }
         if (isQueueing()) {
            transaction(new JedisStatusResult(transaction.set(key, value)));
            return;
         }
         jedis.set(key, value);//后面以这里为例，调用jedis的代码
      } catch (Exception ex) {
         throw convertJedisAccessException(ex);
      }
   }
   ...... //此类里面就会有很多
   }
   ```

   （5）后面的方法的调用就会到 Jedis 包里面的东西了：

   ``` java
   package redis.clients.jedis;
   public class BinaryJedis implements BasicCommands, BinaryJedisCommands, MultiKeyBinaryCommands, AdvancedBinaryJedisCommands, BinaryScriptingCommands, Closeable {
   public String set(byte[] key, byte[] value) {
       this.checkIsInMultiOrPipeline();
       this.client.set(key, value);
       return this.client.getStatusCodeReply();
   }
   ......//this.client.set 就是jedis里面的对socket连接的redis-client的命令的发送。
   }
   ```

   （6）到此，就实现了整个 Spring Data Redis 的完美封装，再去查看 jedis 的源码的时候就会发现，其实后面就是建立 Socket 连接，然后发送一些 Redis-client 里面的命令到 Redis-server 的监听端口。
