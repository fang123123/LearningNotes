SpringBoot 高级

## 一、SpringBoot与缓存

### JSR107

#### 简介

Java Caching定义了5个核心接口，分别是CachingProvider, CacheManager, Cache, Entry和 Expiry

- CachingProvider定义了创建、配置、获取、管理和控制多个CacheManager。一个应用可以在运行期访问多个CachingProvider。
- CacheManager定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。
- Cache是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。
- Entry是一个存储在Cache中的key-value对。
-  Expiry 每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。  

![image-JSR107](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200315152604037.png)

#### 开发

开发中使用JSR107需导入pom坐标，但一般不使用，因为JSR107仅仅定义接口，并未实现

```xml
<!-- https://mvnrepository.com/artifact/javax.cache/cache-api -->
<dependency>
    <groupId>javax.cache</groupId>
    <artifactId>cache-api</artifactId>
    <version>1.1.0</version>
</dependency>
```



### Spring缓存抽象

#### 简介

Spring从3.1开始定义了org.springframework.cache.Cache和org.springframework.cache.CacheManager接口来统一不同的缓存技术；并支持使用JCache（JSR-107）注解简化我们开发；  

CacheManager接口说明

- CacheManager缓存管理器，管理各种缓存(Cache)组件，不同的CacheManager可以管理不同类型的Cache实现，==SpringBoot中SimpleConfiguration配置文件中指定默认缓存管理器是ConcurrentMapCacheManager==；

Cache接口说明

- Cache接口为缓存的组件规范定义，包含缓存的各种操作集合；
- Cache接口下Spring提供了各种xxxCache的实现；如RedisCache， EhCacheCache ,CaffineCache等，==SpringBoot默认使用ConcurrenMapCache，维护一个ConcurrentMap作为数据保存的容器==

Spring缓存策略

- 每次调用需要缓存功能的方法时， Spring会检查检查指定参数的指定的目标方法是否已经被调用过；如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。

- 使用Spring缓存抽象时我们需要关注以下两点：

  1、确定方法需要被缓存以及他们的缓存策略
  2、从缓存中读取之前缓存存储的数据  

![image-SpringBoot缓存](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200315154625352.png)

##### 常用注解说明

| 类型           | 作用                                                         | 调用时机             | 常用范围             |
| -------------- | :----------------------------------------------------------- | -------------------- | -------------------- |
| @Cacheable     | 能够根据方法的请求参数对其结果进行缓存，第二次请求时方法不会被调用；标注方法上 | 方法执行之前与执行后 | 添加数据、查找数据   |
| @CacheEvict    | 清空缓存；标注方法上                                         |                      | 删除数据             |
| @CachePut      | 被注解的方法一定会被调用，即使缓存中存在，同时结果被缓存；标注方法上 | 方法执行之后         | 修改数据库并更新缓存 |
| @Caching       | 用于定义复杂的缓存规则，即使用多个上述注解                   |                      |                      |
| @EnableCaching | 开启基于注解的缓存；标注在主配置类上                         |                      |                      |

##### 注解属性说明

@Cacheable、@CachePut、@CacheEvict注解的属性说明

| 主要参数                        | 说明                                                         | 举例                                                         |
| ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| value\|cacheNames               | 缓存的名称，在 spring 配置文件中定义，必须指定，至少一个     | 例如： @Cacheable(value=”mycache”) 或者 @Cacheable(value={”cache1”,”cache2”} |
| key                             | 缓存的 key，可以为空。如果指定要按照 SpEL 表达式编写；如果不指定，则缺省按照方法的所有参数 进行组合 | 例如： @Cacheable(value=”testcache”,key=”#userName”)         |
| keyGenerator                    | 缓存数据时key生成策略，key\|keyGenerator二选一使用           |                                                              |
| condition                       | 缓存的条件，可以为空，使用 SpEL 编写表达式，返回 true 或者 false。符合条件下缓存，在调用方法之前之后都能判断 | 例如： @Cacheable(value=”testcache”,condition=”#userNam e.length()>2”) |
| unless (@CachePut) (@Cacheable) | 表达式只在方法执行之后判断，此时可以拿到返回值result进行判断。符合条件下不缓存 | 例如： @Cacheable(value=”testcache”,unless=”#result == null”) |
| allEntries (@CacheEvict )       | 是否清空所有缓存内容，缺省为 false，如果指定为 true，则方法调用后将立即清空所有缓存 | 例如： @CachEvict(value=”testcache”,allEntries=true)         |
| beforeInvocation (@CacheEvict)  | 是否在方法执行前就清空，缺省为 false，如果指定 为 true，则在方法还没有执行的时候就清空缓存， 缺省情况下，如果方法执行抛出异常，则不会清空 缓存 | 例如： @CachEvict(value=”testcache”， beforeInvocation=true) |
| cacheManager                    | 缓存管理器，默认org.springframework.cache.CacheManager       |                                                              |
| cacheResolver                   | 缓存解析器，由缓存管理器创建，默认org.springframework.cache.interceptor.CacheResolver；cacheManager\|cacheResolver二选一使用 |                                                              |
| sync                            | 是否使用异步模式                                             |                                                              |
| serialize                       | 缓存数据时value序列化策略                                    |                                                              |

SpEL表达式

| 名字          | 位置               | 描述                                                         | 示例                 |
| ------------- | ------------------ | ------------------------------------------------------------ | -------------------- |
| methodName    | root object        | 当前被调用的方法名                                           | #root.methodName     |
| method        | root object        | 当前被调用的方法                                             | #root.method.name    |
| target        | root object        | 当前被调用的目标对象                                         | #root.target         |
| targetClass   | root object        | 当前被调用的目标对象类                                       | #root.targetClass    |
| args          | root object        | 当前被调用的方法的参数列表                                   | #root.args[0]        |
| caches        | root object        | 当前方法调用使用的缓存列表（如@Cacheable(value={"cache1", "cache2"})）， 则有两个cache | #root.caches[0].name |
| argument name | evaluation context | 方法参数的名字. 可以直接 #参数名 ，也可以使用 #p0或#a0 的 形式， 0代表参数的索引； | #iban 、 #a0 、 #p0  |
| result        | evaluation context | 方法执行后的返回值（仅当方法执行之后的判断有效，如 ‘unless’ ， ’cache put’的表达式 ’cache evict’的表达式 beforeInvocation=false） | #result              |

#### Cache基本使用环境搭建

##### 1、创建SpringBoot应用

使用SpringBoot Initializer创建，选中Cache、Mysql、Mybatis、Web模块

SpringBoot已经默认导入了jdbc，不需要考虑

pom坐标如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.2</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>latest</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

##### 2、配置文件配置数据源信息

mysql6.x以上版本配置

```yaml
spring:
  datasource:
    url: jdbc:mysql://49.233.134.44:3306/cache?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8&useSSL=false
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
    initialization-mode: always
```

mysql5.x版本

```yaml
spring:
  datasource:
    url: jdbc:mysql://49.233.134.44:3306
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
    initialization-mode: always
```

##### 3、创建数据库表

```sql
SET FOREIGN_KEY_CHECKS=0;
-- ----------------------------
-- Table structure for department
-- ----------------------------
DROP TABLE IF EXISTS `department`;
CREATE TABLE `department` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `departmentName` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
-- ----------------------------
-- Table structure for employee
-- ----------------------------
DROP TABLE IF EXISTS `employee`;
CREATE TABLE `employee` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `lastName` varchar(255) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  `gender` int(2) DEFAULT NULL,
  `d_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

##### 4、实现SSM

**dao层：**

​	编写mapper接口和Sql语句，用于SpringBoot通过动态代理创建实现类

mybatis的自动配置

==MybatisAutoConfiguration：自动配置了SqlSessionFactory==
==默认mybatis配置类MybatisProperties，会自动加载以mybatis开头的配置文件==

注解方式（只是举例）

```java
/**
* @Mapper:自动给当前类生成实现类，并添加@Component标签
* 作用相当于之前的Mapper配置文件+@Reposity标签
**/
@Mapper
public interface IDepartmentDao {
    //指明主键递增，并将自增的主键返回回来
    @Options(useGeneratedKeys = true, keyProperty = "id")
    @Insert("insert into department(departmentName) values(#{name})")
    int addDept(Department department);

    @Update("update department set departmentName=#{name} where id=#{id}")
    int updateDept(Department department);
    
    @Delete("delete from department where id=#{id}")
    int deleteDeptById(Integer id);
    
    @Select("select * from department where id=#{id}")
    @Results(
            id="departmentMap",value = {
            @Result(id = true,property = "id",column = "id"),
            @Result(property = "name",column = "departmentName")
    })
    //其他地方使用@ResultMap(value="departmentMap")来引用这个封装
    Department findDeptById(Integer id);

}
```

配置文件方式（个人推荐）

1）应用配置文件application.yml

```yaml
mybatis:
  configuration:
  	#开启驼峰命名法
  	map-underscore-to-camel-case: true
  #指定mybatis全局配置文件的位置
  config-location: classpath:mybatis/mybatis-config.xml 
  #通配符指定多个sql映射文件的位置
  mapper-locations: classpath:mybatis/mapper/*.xml
```

2）mybatis全局配置文件mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 开启别名-->
    <typeAliases>
        <package name="com.atguigu.springboot.domain"/>
    </typeAliases>
</configuration>
```

3）sql映射文件EmployeeMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.springboot.dao.IEmployeeDao">
  <resultMap id="employeeResultMap" type="Employee">
    <id property="id" column="id"/>
    <result property="lastName" column="lastName"/>
    <result property="email" column="email"/>
    <result property="gender" column="gender"/>
    <result property="departmentId" column="d_id"/>
  </resultMap>
  <select id="findEmpById" resultMap="employeeResultMap">
    select * from employee where id = #{id}
  </select>
  <insert id="addEmp">
    insert into employee(lastName,email,gender,d_id) values(#{lastName},#{email},#{gender},#{departmentId})
  </insert>
</mapper>
```

##### 5、在启动类上扫描mapper和开启缓存机制

```java
//扫描mapper
@MapperScan(basePackages = "com.atguigu.cache.dao")
//开启缓存机制
@EnableCaching
@SpringBootApplication
public class SpringBoot01CacheApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBoot01CacheApplication.class, args);
    }
}
```

##### 6、在service层使用cache

```java
@Service
public class EmployeeService {
    @Autowired
    private IEmployeeDao employeeDao;
    
    @Cacheable(cacheNames = "emp")
    public Employee getEmp(Integer id) {
        System.out.println("查询员工id:" + id);
        Employee employee = employeeDao.getEmpById(id);
        return employee;
    }
}
```

此时应用配置配置文件信息

```yaml
spring:
  datasource:
    url: jdbc:mysql://49.233.134.44:3306/cache?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8&useSSL=false
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
    initialization-mode: always
#    schema: classpath:springboot_cache.sql
#打开dao层的日志
logging:
  level:
    com:
      atguigu:
        cache:
          dao: debug
```

#### SpringBoot Cache自动配置

##### 自动配置说明

CacheAutoConfiguration自动配置类

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({CacheManager.class})
@ConditionalOnBean({CacheAspectSupport.class})
@ConditionalOnMissingBean(
    value = {CacheManager.class},
    name = {"cacheResolver"}
)
@EnableConfigurationProperties({CacheProperties.class})
@AutoConfigureAfter({CouchbaseAutoConfiguration.class, HazelcastAutoConfiguration.class, HibernateJpaAutoConfiguration.class, RedisAutoConfiguration.class})
@Import({CacheAutoConfiguration.CacheConfigurationImportSelector.class, CacheAutoConfiguration.CacheManagerEntityManagerFactoryDependsOnPostProcessor.class})
public class CacheAutoConfiguration {
    public CacheAutoConfiguration() {
    }
...
    static class CacheConfigurationImportSelector implements ImportSelector {
        CacheConfigurationImportSelector() {
        }
		//该方法自动导入所有的缓存配置
        public String[] selectImports(AnnotationMetadata importingClassMetadata) {
            CacheType[] types = CacheType.values();
            String[] imports = new String[types.length];
            for(int i = 0; i < types.length; ++i) {
                imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
            }
            return imports;
        }
    }
```

##### SpringBoot支持的缓存配置

这些缓存配置加载顺序如下，优先级与加载顺序一致，即下标越小，优先级越高

![image-Cache自动配置类](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200315170822398.png)

通过debug模式来查看默认加载的缓存配置，可以看出默认加载SimpleCacheConfiguration

![image-自动导入缓存配置-1](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200315171320968.png)

![image-自动导入缓存配置-2](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200315171449625.png)

SimpleCacheConfiguration会给容器中添加ConcurrentMapCacheManager

```java
@Configuration(
    proxyBeanMethods = false
)
//实现优先级的方式在这
//先加载的缓存配置类会创建出自己的CacheManager，后面的缓存配置类发现CacheManager类已经创建，就不会再重复创建
@ConditionalOnMissingBean({CacheManager.class})
@Conditional({CacheCondition.class})
class SimpleCacheConfiguration {
    SimpleCacheConfiguration() {
    }
	//可以看出自动配置了ConcurrentMapCacheManager
    @Bean
    ConcurrentMapCacheManager cacheManager(CacheProperties cacheProperties, CacheManagerCustomizers cacheManagerCustomizers) {
        ConcurrentMapCacheManager cacheManager = new ConcurrentMapCacheManager();
        List<String> cacheNames = cacheProperties.getCacheNames();
        if (!cacheNames.isEmpty()) {
            cacheManager.setCacheNames(cacheNames);
        }
        return (ConcurrentMapCacheManager)cacheManagerCustomizers.customize(cacheManager);
    }
}
```

ConcurrentMapCacheManager：

维护一个ConcurrentMap<String, Cache>

作用：获取或创建一个ConcurrentMap类型缓存组件

```java
private final ConcurrentMap<String, Cache> cacheMap = new ConcurrentHashMap<>(16);
...
@Override
@Nullable
public Cache getCache(String name) {
   //获取ConcurrentMap类型的缓存组件
   Cache cache = this.cacheMap.get(name);
   if (cache == null && this.dynamic) {
       //如果不存在，就上锁然后创建一个缓存组件
      synchronized (this.cacheMap) {
         cache = this.cacheMap.get(name);
         if (cache == null) {
            cache = createConcurrentMapCache(name);
            this.cacheMap.put(name, cache);
         }
      }
   }
   return cache;
}
```

ConcurrentMapCache：

维护一个ConcurrentMap<Object, Object>类型的store，真正保存数据的Map

作用：保存或取出数据

```java
private final ConcurrentMap<Object, Object> store;
...
@Override
@Nullable
//取出数据
protected Object lookup(Object key) {
   return this.store.get(key);
}
@SuppressWarnings("unchecked")
@Override
@Nullable
public <T> T get(Object key, Callable<T> valueLoader) {
    return (T) fromStoreValue(this.store.computeIfAbsent(key, k -> {
        try {
            return toStoreValue(valueLoader.call());
        }
        catch (Throwable ex) {
            throw new ValueRetrievalException(key, valueLoader, ex);
        }
    }));
}
//存入数据
@Override
public void put(Object key, @Nullable Object value) {
    this.store.put(key, toStoreValue(value));
}
```

##### 缓存执行的整个流程

1）收到请求，方法运行之前，先按照CacheNames指定的名称获取Cache组件。如果CacheManager中没有该Cache组件，则自定创建

2）使用key去Cache查找缓存的内容。key默认是方法的参数。key是按照一定策略生成的。默认使用SimpleKeyGenerator生成

![image-KeyGenerator-1](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200315181418722.png)

![image-KeyGenerator-2](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200315181326891.png)

KeyGenerator的种类

![image-KeyGenerator](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200315181501619.png)

```java
//SimpleKeyGenerator.class
public class SimpleKeyGenerator implements KeyGenerator {
	@Override
	public Object generate(Object target, Method method, Object... params) {
		return generateKey(params);
	}
	/**
	 * Generate a key based on the specified parameters.
	 */
	public static Object generateKey(Object... params) {
		if (params.length == 0) {
			return SimpleKey.EMPTY;
		}
        //如果参数长为1，直接使用该参数作为key
		if (params.length == 1) {
			Object param = params[0];
			if (param != null && !param.getClass().isArray()) {
				return param;
			}
		}
		return new SimpleKey(params);
	}
}
//SimpleKey.class
public static final SimpleKey EMPTY = new SimpleKey();
public SimpleKey(Object... elements) {
    Assert.notNull(elements, "Elements must not be null");
    this.params = elements.clone();
    // Pre-calculate hashCode field
    this.hashCode = Arrays.deepHashCode(this.params);
}
```

3）没有查找到，就调用目标方法；如果查找到，直接返回，不调用目标方法

4）将目标方法返回的结果放进缓存

#### 使用注解

##### @Cacheable注解

**指定放入缓存的数据的key**

1）通过属性key指定key

弊端：对于每一个方法都需要指定，容易造成key值重复的情况

```java
@Cacheable(cacheNames = "emp",key = "#root.methodName+'['+#id+']'")
```

注意：

==@Cacheable注解不能使用#result来作为值，因为@Cacheable注解在方法执行前后都调用，但是result只在方法执行之后存在==

2）通过属性keyGenerator指定自定义keyGenerator

```java
//自定义KeyGenerator
@Configuration
public class MyCacheConfig {
    @Bean(name = "myKeyGenerator")
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                return method.getName() + "[" + Arrays.asList(params).toString() + "]";
            }
        };
    }
}
```

```java
@Cacheable(cacheNames = "emp",keyGenerator = "myKeyGenerator")
```

**自定义缓存生成的条件**

通过condition属性和unless属性

注意：

==condition在方法调用前后都判断一次==

==unless只在方法调用之后判断==

**自定义缓存生成的机制**

默认是同步方式生成，即第一次方法调用后直接生成缓存

通过sync=true，设置为异步方式

注意：

==设置为异步方式之后，unless条件判断就不支持了==

##### @CachePut注解

错误写法：

```java
@Cacheable(cacheNames = "emp")
public Employee getEmp(Integer id) {
    System.out.println("查询员工id:" + id);
    Employee employee = employeeDao.getEmpById(id);
    return employee;
}
@CachePut(cacheNames = "emp")
public Employee update(Employee employee) {
    System.out.println("更新员工" + employee);
    employeeDao.updateEmp(employee);
    return employee;
}
```

原因：

==如果按照上面的写法，在查询方法时，SpringBoot使用id作为key，在更新时，SpringBoot使用Employee类作为key，两者会造成缓存对象不一致问题==

正确写法：

```java
@Cacheable(cacheNames = "emp", key = "#id")
public Employee getEmp(Integer id) {
    System.out.println("查询员工id:" + id);
    Employee employee = employeeDao.getEmpById(id);
    return employee;
}
@CachePut(cacheNames = "emp", key = "#result.id")
public Employee update(Employee employee) {
    System.out.println("更新员工" + employee);
    employeeDao.updateEmp(employee);
    return employee;
}
```

注意：

==@CachePut标注的方法一定会被执行，即使缓存中已经存在==

##### @CacheEvict注解

```java
/**
* 属性说明：
* 	key
*		功能：根据key删除缓存数据
*		@CacheEvict(value = "emp",key = "#id")
* 	allEntries
*		功能：清除所有缓存
*		值：
*			true：清除所有缓存
*			false：不清除所有缓存，默认值
*		@CacheEvict(value = "emp",allEntries = true)
* 	beforeInvocation
*		功能：缓存清除时机
*		值：
*			true：在方法执行之前执行
*			false：在方法执行之后执行，默认值
*		举例：
*			场景：如果在执行删除操作失败
*				如果在方法执行之前执行清除缓存操作，则缓存已经清除
*				如果在方法执行之后执行清除缓存操作，则缓存没有被清除
*		@CacheEvict(value = "emp",beforeInvocation = true)
**/
public void deleteEmp(Integer id) {
    System.out.println("删除员工" + id);
    //employeeDao.deleteEmpById(id);
}
```

SpringBoot事务处理应该只包含对于数据库操作的处理，所以缓存部分应该就是在service层与数据库操作一起执行

##### @Caching注解和@CacheConfig注解

```java
@Service
//全局配置,方便将重复配置抽取出来
@CacheConfig(cacheNames = "emp",keyGenerator = "myKeyGenerator")
public class EmployeeService {
    @Autowired
    ...
    private IEmployeeDao employeeDao;
    @Caching(
            cacheable = {
                    @Cacheable(value = "emp", key = "#lastName")
            },
        //由于使用了@CachePut注解，此处查找的数据即使在缓存中，方法也会被调用
            put = {
                    @CachePut(value = "emp", key = "#result.id"),
                    @CachePut(value = "emp", key = "#result.email")
            }
    )
    public Employee getEmpByLastName(String lastName) {
        return employeeDao.getEmpByName(lastName);
    }
}
```

### Redis

#### 简介

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件

Redis常见的五大数据类型：

String、List、Set、Hash、ZSet（有序集合）

#### Redis基本使用环境搭建

1、在服务器中使用docker配置redis

```shell
docker run -d -p 6379:6379 redis(镜像名称)
```

2、IDEA中导入Redis的POM坐标

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

3、添加服务器Redis连接信息

```yaml
spring:
  redis:
    host: 49.233.134.44
    port: 6379
    timeout: 2000
```

#### Redis自动配置

在上面的[SpringBoot支持的缓存配置](#SpringBoot支持的缓存配置)中谈到SpringBoot中所有的缓存配置都存在xxCacheConfiguration文件夹中，并且优先级与加载顺序有关

```java
//注意自动配置类都是在autoconfigure包下
//在包org.springframework.data.redis.cache下也有一个同名类，注意区别
package org.springframework.boot.autoconfigure.cache;

@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisConnectionFactory.class)
@AutoConfigureAfter(RedisAutoConfiguration.class)
@ConditionalOnBean(RedisConnectionFactory.class)
@ConditionalOnMissingBean(CacheManager.class)
@Conditional(CacheCondition.class)
class RedisCacheConfiguration {
	/**
	* cacheProperties：
	* cacheManagerCustomizers
	* redisCacheConfiguration 
	* 
	*/
	@Bean
	RedisCacheManager cacheManager(CacheProperties cacheProperties, 
                                   CacheManagerCustomizers cacheManagerCustomizers,			ObjectProvider<org.springframework.data.redis.cache.RedisCacheConfiguration> redisCacheConfiguration,
ObjectProvider<RedisCacheManagerBuilderCustomizer> redisCacheManagerBuilderCustomizers,
			RedisConnectionFactory redisConnectionFactory, 
                                   ResourceLoader resourceLoader) {
		RedisCacheManagerBuilder builder = RedisCacheManager
            .builder(redisConnectionFactory)
            .cacheDefaults(
				determineConfiguration(
                    cacheProperties, 
                    redisCacheConfiguration, 
                    resourceLoader.getClassLoader()
                )
        );
		List<String> cacheNames = cacheProperties.getCacheNames();
		if (!cacheNames.isEmpty()) {
			builder.initialCacheNames(new LinkedHashSet<>(cacheNames));
		}
		redisCacheManagerBuilderCustomizers.orderedStream().forEach((customizer) -> customizer.customize(builder));
		return cacheManagerCustomizers.customize(builder.build());
	}

	private org.springframework.data.redis.cache.RedisCacheConfiguration determineConfiguration(
			CacheProperties cacheProperties,
			ObjectProvider<org.springframework.data.redis.cache.RedisCacheConfiguration> redisCacheConfiguration,
			ClassLoader classLoader) {
		return redisCacheConfiguration.getIfAvailable(() -> createConfiguration(cacheProperties, classLoader));
	}

	private org.springframework.data.redis.cache.RedisCacheConfiguration createConfiguration(
			CacheProperties cacheProperties, ClassLoader classLoader) {
		Redis redisProperties = cacheProperties.getRedis();
		org.springframework.data.redis.cache.RedisCacheConfiguration config = org.springframework.data.redis.cache.RedisCacheConfiguration
				.defaultCacheConfig();
		config = config.serializeValuesWith(
				SerializationPair.fromSerializer(new JdkSerializationRedisSerializer(classLoader)));
		if (redisProperties.getTimeToLive() != null) {
			config = config.entryTtl(redisProperties.getTimeToLive());
		}
		if (redisProperties.getKeyPrefix() != null) {
			config = config.prefixKeysWith(redisProperties.getKeyPrefix());
		}
		if (!redisProperties.isCacheNullValues()) {
			config = config.disableCachingNullValues();
		}
		if (!redisProperties.isUseKeyPrefix()) {
			config = config.disableKeyPrefix();
		}
		return config;
	}

}

```



#### Redis使用

##### RedisTemplate

SpringBoot已经默认配置好了Redis的环境，提供了两个模板对象RedisTemplate和StringRedisTemplate来操作Redis

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }
	//RedisTemplate<Object, Object>
    @Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    )
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
	//由于字符串操作频繁，所以专门定义模板用于操作字符串
    //StringRedisTemplate 就是 RedisTemplate<String, String>
    @Bean
    @ConditionalOnMissingBean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```

RedisTemplate默认配置说明

```java
//默认RedisTemplate配置
public class RedisTemplate<K, V> extends RedisAccessor implements RedisOperations<K, V>, BeanClassLoaderAware {
	private boolean enableTransactionSupport = false;
	private boolean exposeConnection = false;
	private boolean initialized = false;
	private boolean enableDefaultSerializer = true;
	//默认序列化器
	private @Nullable RedisSerializer<?> defaultSerializer;
	private @Nullable ClassLoader classLoader;
	...
    if (defaultSerializer == null) {
    //默认使用JDK序列化器
    defaultSerializer = new JdkSerializationRedisSerializer(
    classLoader != null ? classLoader : this.getClass().getClassLoader());
    }
```

##### 使用RedisTemplate模板来操作Redis

```java
@Test
void testRedis(){
    //保存字符串数据
    stringRedisTemplate.opsForValue().append("msg","hello");
    //取出数据
    String msg = stringRedisTemplate.opsForValue().get("msg");
    System.out.println(msg);
    //保存列表
    stringRedisTemplate.opsForList().leftPush("mylist","1");
}
@Test
public void testSaveObject(){
    Employee employee = employeeDao.getEmpById(1);
    //以jdk序列化方式保存对象
    redisTemplate.opsForValue().set("emp-01",employee);
}
```

##### 自定义RedisTemplate和RedisCacheManager

```java
Configuration
public class MyRedisConfig {
    @Value("${spring.cache.redis.time-to-live}")
    //缓存生存时间
    private Duration timeToLive = Duration.ofDays(1);

    /**
     * 自定义RedisTemplate
     * 
     * 使用自定义的template:
	 * @Autowired
	 * RedisTemplate<Object, Employee> myRedisTemplate;   
	 * myRedisTemplate.opsForValue().set("emp-01", employee);
     */
    @Bean
    public RedisTemplate<Object, Object> myRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        /*
         * Redis 序列化器.
         *
         * RedisTemplate 默认的系列化类是 JdkSerializationRedisSerializer,用JdkSerializationRedisSerializer序列化的话,
         * 被序列化的对象必须实现Serializable接口。在存储内容时，除了属性的内容外还存了其它内容在里面，总长度长，且不容易阅读。
         *
         * Jackson2JsonRedisSerializer 和 GenericJackson2JsonRedisSerializer，两者都能系列化成 json，
         * 但是后者会在 json 中加入 @class 属性，类的全路径包名，方便反系列化。前者如果存放了 List 则在反系列化的时候如果没指定
         * TypeReference 则会报错 java.util.LinkedHashMap cannot be cast to
         */
        RedisSerializer genericJackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        RedisSerializer stringRedisSerializer = new StringRedisSerializer();

        RedisTemplate<Object, Object> template = new RedisTemplate();

        //自定义key的序列化器
        template.setKeySerializer(stringRedisSerializer);
        template.setHashKeySerializer(stringRedisSerializer);
        //自定义Value的序列化器
        template.setValueSerializer(genericJackson2JsonRedisSerializer);
        template.setHashValueSerializer(genericJackson2JsonRedisSerializer);
        //设置连接工厂
        template.setConnectionFactory(redisConnectionFactory);
        template.afterPropertiesSet();
        return template;
    }

    /**
     * 自定义CacheManager
     * 
     * 配置之后，使用SpringBoot提供的缓存注解时，都使用自定义配置完成
     */
    @Bean
    public CacheManager myRedieCacheManager(RedisConnectionFactory factory) {
        RedisSerializer stringRedisSerializer = new StringRedisSerializer();
        RedisSerializer genericJackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();

        //自定义配置RedisCacheManager
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(timeToLive)
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(stringRedisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(genericJackson2JsonRedisSerializer))
                .disableCachingNullValues();

        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
                .cacheDefaults(config)                .build();
        return cacheManager;
    }
}
```

## 二、SpringBoot与消息

### 消息服务简介

#### 1、消息代理和目的地

​	当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目的。

==消息代理是一种架构模式==，用于消息验证、变换、路由。虽然不同的消息中间件架构和实现各不相同，但是大部分都实现了Broker：其实就是消息中间件服务器，它是中间件的核心。

​	注意:RabbitMQ、Kafka、RocketMQ等都有消息代理，但是注意，不是所有中间件都这么选，例如ZeroMQ，它用了套接字风格的API。

​	==在一些地方其实说消息代理就是指消息中间件==，如Python语言知名的分布式任务队列框架Celery中就这么称呼的(所谓的「任务」其实就是一个包含了任务全部数据的消息)。

目的地的形式主要有两种：

1. ==队列==（queue） ：点对点消息通信（point-to-point）
2. ==主题==（topic） ：发布（publish） /订阅（subscribe）消息通信 

#### 2、点对点式队列

– 消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，
消息读取后被移出队列
– ==消息只有唯一的发送者和接受者==：可以存在多个接受者，但是一条消息只能被一个接受者获取

#### 3、发布订阅式

– 发送者（发布者）发送消息到主题，==多个接收者（订阅者）监听（订阅）这个主题==，那么就会在消息到达时同时收到消息

#### 4、JMS（Java Message Service） JAVA消息服务

– 基于JVM消息代理的规范。 ActiveMQ、 HornetMQ是JMS实现

#### 5、AMQP（Advanced Message Queuing Protocol）

– 高级消息队列协议，也是一个消息代理的规范，兼容JMS
– RabbitMQ是AMQP的实现  

JMS与AMQP

|              | JMS                                                          | AMQP                                                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 定义         | Java api                                                     | 网络线级协议                                                 |
| 跨语言       | 否                                                           | 是                                                           |
| 跨平台       | 否                                                           | 是                                                           |
| Model        | 提供两种消息模型： <br />（1）、 Peer-2-Peer <br />（2）、 Pub/sub | 提供了五种消息模型： <br />（1）、 direct exchange (P2P)<br />（2）、 fanout exchange <br />（3）、 topic change <br />（4）、 headers exchange <br />（5）、 system exchange <br />本质来讲，后四种和JMS的pub/sub模型没有太大差别，仅是在 路由机制上做了更详细的划分； |
| 支持消息类型 | 多种消息类型：<br /> TextMessage<br /> MapMessage<br /> BytesMessage<br /> StreamMessage <br />ObjectMessage Message （只有消息头和属性） | byte[] 当实际应用时，有复杂的消息，可以将消息序列化后发送。  |
| 综合评价     | JMS 定义了JAVA API层面的标准；在java体系中，多个client 均可以通过JMS进行交互，不需要应用修改代码，但是其对跨 平台的支持较差； | AMQP定义了wire-level层的协议标准；天然具有跨平台、跨语 言特性。 |

#### 6、Spring支持

– spring-jms提供了对JMS的支持
– spring-rabbit提供了对AMQP的支持
– 需要ConnectionFactory的实现来连接消息代理
– 提供JmsTemplate、 RabbitTemplate来发送消息
– @JmsListener（JMS）、 @RabbitListener（AMQP）注解在方法上监听消息代理发
布的消息
– @EnableJms、 @EnableRabbit开启支持

#### 7、 Spring Boot自动配置

– JmsAutoConfiguration
– RabbitAutoConfiguration  

#### 8、消息中间件的优点

**异步处理**

<img src="D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200326172252457.png" alt="image-消息服务与异步处理-1"  />

<img src="D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200326172319096.png" alt="image-消息服务与异步处理-2"  />

![image-消息服务与异步处理-3](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200326172348533.png)

**应用解耦**

![image-消息服务与应用解耦-1](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200326172422626.png)

![image-消息服务与应用解耦-2](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200326172444263.png)

**流量削峰**

![image-消息服务与流量削峰](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200326172728108.png)

#### 9、消息中间件的缺点

场景：

A系统的哥们要把系统关键数据发送给BC系统的

未引入消息中间件之前方式：

A系统直接调用BC系统接口，传入数据

引入消息中间件之前方式：

A系统将数据放入MQ，BC系统从MQ中取数据

**系统可用性降低**

MQ可能会挂掉。只要MQ挂了，数据没了，系统运行就不对了。

**系统复杂度提高**

原先系统只需要通过接口调用一下，但是加入一个MQ之后，需要考虑消息重复消费、消息丢失、甚至消息顺序性的问题。

为了解决这些问题，又需要引入很多复杂的机制，这样一来是不是系统的复杂度提高了。

**数据一致性问题**

本来好好的，A系统调用BC系统接口，如果BC系统出错了，会抛出异常，返回给A系统让A系统知道，这样的话就可以做回滚操作了

但是使用了MQ之后，A系统发送完消息就完事了，认为成功了。而刚好C系统写数据库的时候失败了，但是A认为C已经成功了?这样一来数据就不一致了。

**MQ消息丢失问题**

1. 生产者弄丢数据：系统A将数据发送到MQ的时候,可能数据在网络传输中搞丢了
2. 消费者弄丢数据：在消费消息的时候，刚拿到消息，结果进程挂了，MQ就会认为你已经消费成功了，这条数据就丢了

使用了MQ之后，还要关心消息丢失的问题。这里我们挑RabbitMQ来说明一下吧。

**生产者弄丢了数据**

RabbitMQ生产者将数据发送到rabbitmq的时候,可能数据在网络传输中搞丢了，这个时候RabbitMQ收不到消息，消息就丢了。

### RabbitMQ简介

RabbitMQ是一个由erlang开发的AMQP(Advanved Message Queue Protocol)的开源实现。

#### RabbitMQ核心概念

**Message**
消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键，指定接收者）、 priority（相对于其他消息的优先权）、 delivery-mode（指出该消息可能需要持久性存储）等。
**Publisher**
消息的生产者，也是一个向交换器发布消息的客户端应用程序。
**Exchange**
交换器，用来接收生产者发送的消息并根据路由键与路由规则将这些消息路由给服务器中的队列。
Exchange有4种类型： direct(默认)， fanout, topic, 和headers，不同类型的Exchange转发消息的策略有所区别
**Queue**
消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。
**Binding**
绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。
Exchange 和Queue的绑定可以是多对多的关系。
**Connection**
网络连接，比如一个TCP连接。
**Channel**
信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内的虚拟连接， AMQP 命令都是通过信道Queue
消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。
**Consumer**
消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。
**Virtual Host**
虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。 vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。
**Broker**
消息代理，表示消息队列服务器实体  

![image-消息代理架构图](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327000743158.png)

#### RabbitMQ运行机制 

##### AMQP 中的消息路由  

AMQP 中消息的路由过程和 Java 开发者熟悉的 JMS 存在一些差别， AMQP 中增加了Exchange 和 Binding 的角色。生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到那个队列。

![image-AMQP消息路由机制](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327002149200.png)

##### Exchange 类型

Exchange分发消息时根据类型的不同分发策略有区别，目前共四种类型：direct、 fanout、 topic、 headers 。 
headers 匹配 AMQP 消息的 header而不是路由键， headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种类型：

**direct类型(单播)**

消息中的路由键（routing key）如果和 Binding 中的 bindingkey 一致， 交换器就将消息发到对应的队列中。路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为“dog”，则只转发 routing key 标记为“dog”的消息，不会转发“dog.puppy”，也不会转发“dog.guard”等等。它是完全匹配、单播的模式。  

![image-Exchange-direct](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327002947975.png)

**fanout(广播)**

每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。 fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。 fanout 类型转发消息是最快的。  

![image-Exchange-fanout](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327003207854.png)

**topic(多播)**

topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符：符号“#”和符号“*” 。 #匹配0个或多个单词， *匹配一个单词。  

![image-Exchange-topic](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327003728885.png)

### 搭建RabbitMQ环境

#### 创建RabbitMQ容器

```shell
# 在docker中安装rabbit镜像
docker pull rabbit
# 运行镜像，启动容器
docker run -d -p 5672:5672 -p 15672:15672 --name myrabbitmq xx(镜像名称|id)
# 这里先访问下这个端口，如果访问不了，进行下面操作
# 进入容器，运行插件
docker exec -it 
rabbitmq-plugins enable rabbitmq_management
# 退出容器
Ctrl+P+Q
```

访问RabbitMQ客户端的端口15672

连接使用RabiitMQ的端口是5672

账户密码：guest

![image-RabbitMQ管理界面](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327140852020.png)

#### 客户端创建RabbitMQ

自定义RabbitMQ架构图

![image-自定义RabbitMQ架构图](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327141659632.png)

添加交换机

<img src="D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327142131123.png" alt="image-添加交换机" style="zoom:67%;" />

添加消息队列

<img src="D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327142920064.png" alt="image-添加消息队列" style="zoom: 67%;" />

将交换机与消息队列绑定

direct交换机必须指明具体路由键

fanout交换机可以不需要指定路由键

topic交换机使用带通配符的路由键

<img src="D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327144336275.png" alt="image-绑定交换机与消息队列" style="zoom:67%;" />

#### 客户端测试RabbitMQ

交换机发送消息

<img src="D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327145626067.png" alt="image-交换机发送消息" style="zoom:67%;" />

消息队列接受具体消息

<img src="D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327151041111.png" alt="image-消息队列获取消息" style="zoom:67%;" />%

### SpringBoot引入RabbitMQ

#### 搭建环境

引入POM坐标

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit-test</artifactId>
    <scope>test</scope>
</dependency>
```

配置RabbitMQ参数

```yaml
spring:
  rabbitmq:
    host: 49.233.134.44
    username: guest
    password: guest
    port: 5672
    virtual-host: /
```

#### RabbitTemplate发送接收消息

```java
    //单播
    @Test
    void contextLoads() {
        /**
         * 发消息方式一：
         * rabbitTemplate.send(...)
         * exchange：交换机
         * routeKey：路由规则
         * Message：要发送的消息
         * Message需要自己构造：定义消息体内容和消息头
         */
//        rabbitTemplate.send(exchange,routeKey,message);
        Map<String, Object> map = new HashMap<>();
        map.put("msg1", "helloworld");
        map.put("data", Arrays.asList("apple", "orange", "pear"));
        /**
         * 发消息方式二：
         * exchange：交换机
         * routeKey：路由规则
         * object：要发送的消息，该方法会自动以java序列化方式序列化为字节数组作为消息体
         * 较第一种方式简单
         */
        rabbitTemplate.convertAndSend("exchange.direct", "atguigu.news", map);
    }
    @Test
    void test1(){
        //接受数据，数据接受之后，消息队列的数据会自动删除
        Object object = rabbitTemplate.receiveAndConvert("atguigu.news");
        System.out.println(object);
        System.out.println(object.getClass());
    }
```

可以看到接受的数据格式：x-java-serialized-object

![接受的数据](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327195653831.png)

### SprigBoot中RabbitMQ自动配置

#### SpringBoot提供的RabbitMQ组件

自动添加CachingConnectionFactory、RabbitTemplate、AmqpAdmin

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {
	
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnMissingBean(ConnectionFactory.class)
   protected static class RabbitConnectionFactoryCreator {
       //根据RabbitProperties来生成连接工厂
      @Bean
      public CachingConnectionFactory rabbitConnectionFactory(RabbitProperties properties,
            ObjectProvider<ConnectionNameStrategy> connectionNameStrategy) throws Exception {
   ...
   @Configuration(proxyBeanMethods = false)
	@Import(RabbitConnectionFactoryCreator.class)
	protected static class RabbitTemplateConfiguration {
		//提供一个RabbitTemplate，用于给RabbitMQ发送和接受消息
		@Bean
		@ConditionalOnSingleCandidate(ConnectionFactory.class)
		@ConditionalOnMissingBean(RabbitOperations.class)
		public RabbitTemplate rabbitTemplate(RabbitProperties properties,
				ObjectProvider<MessageConverter> messageConverter,
				ObjectProvider<RabbitRetryTemplateCustomizer> retryTemplateCustomizers,
				ConnectionFactory connectionFactory) {
            ...
            //当ioc容器中不存在MessageConverter，使用默认的MessageConverter
            messageConverter.ifUnique(template::setMessageConverter);
        ...
        //AmqpAdmin：RabbitMQ系统管理功能组件，用于创建exchange、队列
        @Bean
		@ConditionalOnSingleCandidate(ConnectionFactory.class)
		@ConditionalOnProperty(prefix = "spring.rabbitmq", name = "dynamic", matchIfMissing = true)
		@ConditionalOnMissingBean
		public AmqpAdmin amqpAdmin(ConnectionFactory connectionFactory)
```

#### RabbitTemplate自动配置

**MessageConverter消息转换器**

SpringBoot中默认使用SimpleMessageConverter类型的MessageConverter

```java
SimpleMessageConverter.class
//反序列化
public Object fromMessage(Message message) throws MessageConversionException {
    Object content = null;
    MessageProperties properties = message.getMessageProperties();
    if (properties != null) {
        String contentType = properties.getContentType();
        if (contentType != null && contentType.startsWith("text")) {
            String encoding = properties.getContentEncoding();
            if (encoding == null) {
                encoding = this.defaultCharset;
            }

            try {
                content = new String(message.getBody(), encoding);
            } catch (UnsupportedEncodingException var8) {
                throw new MessageConversionException("failed to convert text-based Message content", var8);
            }
        } else if (contentType != null && contentType.equals("application/x-java-serialized-object")) {
            try {
            //将message转换成Object类型的content
                content = SerializationUtils.deserialize(this.createObjectInputStream(new ByteArrayInputStream(message.getBody()), this.codebaseUrl));
            } catch (IllegalArgumentException | IllegalStateException | IOException var7) {
                throw new MessageConversionException("failed to convert serialized Message content", var7);
            }
        }
    }

    if (content == null) {
        content = message.getBody();
    }

    return content;
}

protected Message createMessage(Object object, MessageProperties messageProperties) throws MessageConversionException {
    byte[] bytes = null;
    if (object instanceof byte[]) {
        bytes = (byte[])((byte[])object);
        messageProperties.setContentType("application/octet-stream");
    } else if (object instanceof String) {
        try {
            bytes = ((String)object).getBytes(this.defaultCharset);
        } catch (UnsupportedEncodingException var6) {
            throw new MessageConversionException("failed to convert to Message content", var6);
        }

        messageProperties.setContentType("text/plain");
        messageProperties.setContentEncoding(this.defaultCharset);
    } else if (object instanceof Serializable) {
        try {
        //将object类型的message转换成字节数组
            bytes = SerializationUtils.serialize(object);
        } catch (IllegalArgumentException var5) {
            throw new MessageConversionException("failed to convert to serialized Message content", var5);
        }

        messageProperties.setContentType("application/x-java-serialized-object");
    }

    if (bytes != null) {
        messageProperties.setContentLength((long)bytes.length);
        //将字节数组和消息头封装成消息
        return new Message(bytes, messageProperties);
    } else {
        throw new IllegalArgumentException(this.getClass().getSimpleName() + " only supports String, byte[] and Serializable payloads, received: " + object.getClass().getName());
    }
```

```java
//SimpleMessageConverter实际用到的序列化与反序列化工具类
public static byte[] serialize(Object object) {
    if (object == null) {
        return null;
    } else {
        ByteArrayOutputStream stream = new ByteArrayOutputStream();

        try {
            (new ObjectOutputStream(stream)).writeObject(object);
        } catch (IOException var3) {
            throw new IllegalArgumentException("Could not serialize object of type: " + object.getClass(), var3);
        }

        return stream.toByteArray();
    }
}

public static Object deserialize(byte[] bytes) {
    if (bytes == null) {
        return null;
    } else {
        try {
            return deserialize(new ObjectInputStream(new ByteArrayInputStream(bytes)));
        } catch (IOException var2) {
            throw new IllegalArgumentException("Could not deserialize object", var2);
        }
    }
}
```

### SpringBoot中RabbitMQ自定义配置

#### 自定义MessageConverter

将Object对象保存为json格式

```java
@Configuration
public class MyAMQPConfig {
    @Bean
    public MessageConverter myMessageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```

自定义消息转换器后，队列的消息格式为json

![image-自定义MessageConverter](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200327204854446.png)

#### 自定义Object对象

需要注意：

1、RabbitMQ默认序列化方式和之前Redis默认序列化方式不一样

RabbitMQ默认序列化是将对象转为字节流

Redis默认使用的是JDK序列化方式，需要实现Serilizable接口

2、不论哪种序列化方式，如果对象定义了有参构造器，则需要再单独声明一个无参构造器

原因：反序列化时，使用java反射机制需要使用无参构造器

未定义有参构造器时，可以不声明无参构造器的原因：java会给没有构造器的类自动声明一个无参构造器



### SpringBoot使用RabbitMQ

#### 消息监听

##### @EnableRabbit注解

开启RabbitMQ

```java
@EnableRabbit
@SpringBootApplication
public class SpringBoot02AmqpApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBoot02AmqpApplication.class, args);
    }
}
```

##### @RabbitListener

监听队列消息

```java
@Service
public class BookService {
    @RabbitListener(queues = "atguigu.news")
    public void receive(Book book){
        System.out.println("收到消息:\n"+book);
    }
    @RabbitListener(queues = "atguigu")
    public void receive2(Message message){
        System.out.println(message.getMessageProperties());
        System.out.println(message.getBody());
    }
}
```

#### AmqpAdmin创建和删除组件

```java
@Autowired
private AmqpAdmin amqpAdmin;

    @Test
    void createExchange() {
        //创建交换机
        amqpAdmin.declareExchange(new DirectExchange("amqpExchange"));
        //创建队列
        amqpAdmin.declareQueue(new Queue("amqpQueue"));
        /**
         * 绑定参数：
         *      String destination
         *          目的地的名称
         *      Binding.DestinationType destinationType
         *          目的地类型
         *          可选参数：
         *              Binding.DestinationType.QUEUE
         *              Binding.DestinationType.EXCHANGE
         *      Exchange exchange
         *          当前交换机的名称
         *      Map<String, Object> arguments
         *          其他参数，没有就填null
         */
        //将交换机和队列绑定
        amqpAdmin.declareBinding(new Binding("amqpQueue", Binding.DestinationType.QUEUE, "amqpExchange", "amqp.news", null));
        amqpAdmin.deleteExchange("amqpExchange");
        amqpAdmin.deleteQueue("amqpQueue");
    }
```

## 三、SpringBoot与检索

### Elasticsearch简介

​	我们的应用经常需要添加检索功能，开源的 ElasticSearch 是目前全文搜索引擎的首选。 他可以快速的存储、搜索和分析海量数据。 Spring Boot通过整合Spring Data ElasticSearch为我们提供了非常便捷的检索功能支持；
​	Elasticsearch是一个分布式搜索服务，提供Restful API，底层基于Lucene，采用多shard（分片）的方式保证数据安全，并且提供自动resharding的功能， github、wiki等大型的站点也是采用了ElasticSearch作为其搜索服务。

### 配置Elasticsearch

```shell
docker pull elasticsearch:7.6.1
docker network create somenetwork
docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "ES_JAVA_PORTS=-Xms256m -Xmx256m" elasticsearch:7.6.1
```

当控制台显示这样的json数据，说明配置成功

![Elasticsearch控制台](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200330172750621.png)

### 对Elasticsearch进行CRUD操作

Elasticsearch是支持Restful风格的操作

```java
//添加,修改数据
PUT http://xx.xx.xx.xx:9200/customer/_doc/1
//删除数据,如果数据不存在，返回404状态
DELETE http://xx.xx.xx.xx:9200/customer/_doc/1
//查询数据
GET http://xx.xx.xx.xx:9200/customer/_doc/1
```

url说明

每个索引下有多个类型的对象，每个对象有多条id不同的数据

```
http://xx.xx.xx.xx:9200/customer/_doc/1
主机地址:端口号/索引名称/索引类型/索引id
```

![image-elasticsearch架构图](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200330202133311.png)



#### Elasticsearch条件查询

```json
GET /_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }},
        { "match": { "content": "Elasticsearch" }}
      ],
      "filter": [ 
        { "term":  { "status": "published" }},
        { "range": { "publish_date": { "gte": "2015-01-01" }}}
      ]
    }
  }
}
```

### SpringBoot引入Elasticsearch

对于SpringBoot来说，操作Elasticsearch与操作mysql都一样

参考网址：[SpringBoot整合ES的三种方式（API、REST Client、Data-ES）](https://blog.csdn.net/jacksonary/article/details/82729556?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)

#### SpringBoot默认提供配置

1）Spring Data：Client、模板ElasticsearchTemplate

2）ElasticsearchRepository接口

3）Jest(过时了)

4）RestClient（推荐使用）

```java
@EnableConfigurationProperties({ElasticsearchProperties.class})
public class ElasticsearchAutoConfiguration {
    private final ElasticsearchProperties properties;

    public ElasticsearchAutoConfiguration(ElasticsearchProperties properties) {
        this.properties = properties;
    }

	//SpringBoot提供一个客户端Client
    @Bean
    @ConditionalOnMissingBean
    public TransportClient elasticsearchClient() throws Exception {
        TransportClientFactoryBean factory = new TransportClientFactoryBean();
        factory.setClusterNodes(this.properties.getClusterNodes());
        factory.setProperties(this.createProperties());
        factory.afterPropertiesSet();
        return factory.getObject();
    }
	//这个客户端需要资源
    private Properties createProperties() {
        Properties properties = new Properties();
        properties.put("cluster.name", this.properties.getClusterName());
        properties.putAll(this.properties.getProperties());
        return properties;
    }
}

/**
* 通过配置文件设置ElasticsearchProperties
* 配置文件中添加资源clusterName、clusterNodes
*/
@ConfigurationProperties(
    prefix = "spring.data.elasticsearch"
)
public class ElasticsearchProperties {
    private String clusterName = "elasticsearch";
    private String clusterNodes;
    private Map<String, String> properties = new HashMap();
```

#### 使用RestClient操作Elasticsearch

```java
@ConfigurationProperties(
    prefix = "spring.elasticsearch.rest"
)
public class RestClientProperties {
    private List<String> uris = new ArrayList(Collections.singletonList("http://localhost:9200"));
    private String username;
    private String password;
    private Duration connectionTimeout = Duration.ofSeconds(1L);
    private Duration readTimeout = Duration.ofSeconds(30L);

    public RestClientProperties() {
    }
```

#### 自定义配置RestClient

```java
@Configuration
public class MyRestClientConfig {
    @Bean
    public RestClient getClient(){
        RestClientBuilder builder = RestClient.builder(new HttpHost("49.233.134.44",9200,"http"));
        return builder.build();
    }
}
```

更多配置

##### **请求头**

```java
// 设置请求头，每个请求都会带上这个请求头
Header[] defaultHeaders = {new BasicHeader("charset", "utf-8"),
                new BasicHeader("content-type", "application/json")};
clientBuilder.setDefaultHeaders(defaultHeaders);
```

##### **超时时间**

```java
// 设置超时时间，多次尝试同一请求时应该遵守的超时。默认值为30秒，与默认套接字超时相同。若自定义套接字超时，则应相应地调整最大重试超时
clientBuilder.setMaxRetryTimeoutMillis(60000);
```

##### **节点失败监听器**

```java
// 设置监听器，每次节点失败都可以监听到，可以作额外处理
clientBuilder.setFailureListener(new RestClient.FailureListener() {
    @Override
    public void onFailure(Node node) {
        super.onFailure(node);
        System.out.println(node.getName() + "==节点失败了");
    }
});
```

##### **节点选择器**

```java
/* 配置节点选择器，客户端以循环方式将每个请求发送到每一个配置的节点上，
发送请求的节点，用于过滤客户端，将请求发送到这些客户端节点，默认向每个配置节点发送，
这个配置通常是用户在启用嗅探时向专用主节点发送请求（即只有专用的主节点应该被HTTP请求命中）
*/
clientBuilder.setNodeSelector(NodeSelector.SKIP_DEDICATED_MASTERS);

// 进行详细的配置
clientBuilder.setNodeSelector(new NodeSelector() {
    // 设置分配感知节点选择器，允许选择本地机架中的节点（如果有），否则转到任何机架中的任何其他节点。
    @Override
    public void select(Iterable<Node> nodes) {
        boolean foundOne = false;
        for (Node node: nodes) {
            String rackId = node.getAttributes().get("rack_id").get(0);
            if ("rack_one".equals(rackId)) {
                foundOne = true;
                break;
            }
        }
        if (foundOne) {
            Iterator<Node> nodesIt = nodes.iterator();
            while (nodesIt.hasNext()) {
                Node node = nodesIt.next();
                String rackId = node.getAttributes().get("rack_id").get(0);
                if ("rack_one".equals(rackId) == false) {
                    nodesIt.remove();
                }
            }
        }
    }
});
```

##### **HTTP异步请求ES的线程数**

```java
/* 配置异步请求的线程数量，Apache Http Async Client默认启动一个调度程序线程，以及由连接管理器使用的许多工作线程
（与本地检测到的处理器数量一样多，取决于Runtime.getRuntime().availableProcessors()返回的数量）。线程数可以修改如下,
这里是修改为1个线程，即默认情况
*/
clientBuilder.setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
    @Override
    public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpAsyncClientBuilder) {
        return httpAsyncClientBuilder.setDefaultIOReactorConfig(
                IOReactorConfig.custom().setIoThreadCount(1).build()
        );
    }
});
```

##### **连接超时和套接字超时**

```java
/*
    配置请求超时，将连接超时（默认为1秒）和套接字超时（默认为30秒）增加，
    这里配置完应该相应地调整最大重试超时（默认为30秒），即上面的setMaxRetryTimeoutMillis，一般于最大的那个值一致即60000
    */
clientBuilder.setRequestConfigCallback(new RestClientBuilder.RequestConfigCallback() {
    @Override
    public RequestConfig.Builder customizeRequestConfig(RequestConfig.Builder requestConfigBuilder) {
        // 连接5秒超时，套接字连接60s超时
        return requestConfigBuilder.setConnectTimeout(5000).setSocketTimeout(60000);
    }
});
```

##### **ES安全认证**

```
/*
如果ES设置了密码，那这里也提供了一个基本的认证机制，下面设置了ES需要基本身份验证的默认凭据提供程序
    */
final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
credentialsProvider.setCredentials(AuthScope.ANY,
        new UsernamePasswordCredentials("user", "password"));
clientBuilder.setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
    @Override
    public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
        return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
    }
});

/*
上面采用异步机制实现抢先认证，这个功能也可以禁用，这意味着每个请求都将在没有授权标头的情况下发送，然后查看它是否被接受，
并且在收到HTTP 401响应后，它再使用基本认证头重新发送完全相同的请求，这个可能是基于安全、性能的考虑
    */
clientBuilder.setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
    @Override
    public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
        // 禁用抢先认证的方式
        httpClientBuilder.disableAuthCaching();
        return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
    }
});
```

##### **通信加密**

```java
/*
配置通信加密，有多种方式：setSSLContext、setSSLSessionStrategy和setConnectionManager(它们的重要性逐渐递增)
    */
KeyStore truststore = KeyStore.getInstance("jks");
try (InputStream is = Files.newInputStream(keyStorePath)) {
    truststore.load(is, keyStorePass.toCharArray());
}
SSLContextBuilder sslBuilder = SSLContexts.custom().loadTrustMaterial(truststore, null);
final SSLContext sslContext = sslBuilder.build();
clientBuilder.setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
    @Override
    public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
        return httpClientBuilder.setSSLContext(sslContext);
    }
});
```

#### 使用ResetClient操作

传输的Object

```java
@Document(indexName = "entertainment")
public class Article {
//主键
    @Id
    private Integer id;
    private String author;
    private String title;
    private String content;
```

保存|更新、查询操作

```java
@SpringBootTest
class SpringBoot03ElasticsearchApplicationTests {
    @Autowired
    RestClient client;
	//查询
    @Test
    void find() {
        try {
            Response response = client.performRequest(new Request("GET", "/customer/_doc/1"));
            System.out.println(response);
            System.out.println(EntityUtils.toString(response.getEntity()));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
	//保存|更新
    @Test
    void save() {
        Article article = new Article();
        article.setId(1);
        article.setTitle("震惊");
        article.setAuthor("fj");
        article.setContent("xx竟然...");
        try {
            Request request = new Request("POST", new StringBuilder("/article/_doc/").append(article.getId()).toString());
            request.addParameter("pretty", "true");
            Gson gson = new Gson();
            String content = gson.toJson(article);
            request.setEntity(new NStringEntity(content));
            Response response = client.performRequest(request);
            System.out.println(EntityUtils.toString(response.getEntity()));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 四、SpringBoot与任务

### 异步任务

在Java应用中，绝大多数情况下都是通过同步的方式来实现交互处理的；但是在处理与第三方系统交互的时候，容易造成响应迟缓的情况，之前大部分都是使用多线程来完成此类任务，其实，在Spring 3.x之后，就已经内置了@Async来完美解决这个问题。  

**@EnableAsync、@Async注解完成异步任务**

```java
//开启异步任务
@EnableAsync
@SpringBootApplication
public class SpringBoot04TaskApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBoot04TaskApplication.class, args);
    }
}
...
    /**
    * 在方法上使用@Async注解表明该方法执行的是异步任务
    * SpringBoot会自动将该方法加入线程池中
    **/
    @Async
    public void hello() {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("处理数据中");
    }
```

### 定时任务

项目开发中经常需要执行一些定时任务，比如需要在每天凌晨时候，分析一次前一天的日志信息。 Spring为我们提供了异步执行任务调度的方式，提供TaskExecutor 、 TaskScheduler 接口  

**@EnableScheduling、@Scheduled注解完成定时任务**

```java
@EnableAsync
//开启定时任务
@EnableScheduling
@SpringBootApplication
public class SpringBoot04TaskApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBoot04TaskApplication.class, args);
    }
}
...
    /**
     * cron表达式：
     *      second, minute, hour, day of month, month, and day of week
     *      0 * * * * MON-FRI
     *      周一到周五每一分钟（整点）启动一次
     * eg:
     * 		【0 0/5 14,18 * * ?】每天14整和18点整，每隔5分钟执行一次
     *    	【0 15 10 ? * 1-6】每个月周一到周六10:15分执行一次
     *    	【0 0 2 ？ * 6L】每个月最后一个周六凌晨2:00执行一次
     *    	【0 0 2 LW * ?】每个月最后一个工作日凌晨2:00执行一次
     *    	【0 0 2-4 ? * 1#1】每天第一个星期一凌晨2点到4点期间，每个整点执行一次
     */
    @Scheduled(cron = "0 * * * * MON-FRI")
    public void hello(){
        System.out.println("hello....");
    }
```

#### Cron表达式

| 字段 | 允许值                | 允许的特殊字符  |
| ---- | --------------------- | --------------- |
| 秒   | 0-59                  | , - * /         |
| 分   | 0-59                  | , - * /         |
| 小时 | 0-23                  | , - * /         |
| 日期 | 1-31                  | , - * ? / L W C |
| 月份 | 1-12                  | , - * /         |
| 星期 | 0-6或SUN-SAT 0,7是SUN | , - * ? / L C # |

| 特殊字符 | 代表含义                   | 举例                                                         |
| -------- | -------------------------- | ------------------------------------------------------------ |
| ,        | 枚举                       | 1,2,3,4<br />表示1,2,3,4都行                                 |
| -        | 区间                       | 1-4<br />表示1,2,3,4都行                                     |
| *        | 任意                       |                                                              |
| /        | 步长                       | 1/4<br />表示从0开始，每间隔4触发一次                        |
| ?        | 日/星期冲突匹配            | 每个星期一触发<br />日不能写*，得写？，不然每天和每个星期一冲突<br />每月1号触发<br />星期不能写*，得写？，不然每月1号和所有星期冲突 |
| L        | 最后                       |                                                              |
| W        | 工作日                     |                                                              |
| C        | 和calendar联系后计算过的值 |                                                              |
| #        | 星期， 4#2，第2个星期四    |                                                              |

### 邮件任务

参考网址：[SpringBoot配置Email发送功能](https://www.cnblogs.com/James-1024/p/12203770.html)

**邮件发送需要引入spring-boot-starter-mail**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

**Spring Boot 自动配置MailSenderAutoConfiguration**

SpringBoot提供JavaMailSenderImpl来进行邮件操作

```java
class MailSenderPropertiesConfiguration {
    MailSenderPropertiesConfiguration() {
    }
	//SpringBoot自动添加了JavaMailSenderImpl
    @Bean
    @ConditionalOnMissingBean({JavaMailSender.class})
    JavaMailSenderImpl mailSender(MailProperties properties) {
        JavaMailSenderImpl sender = new JavaMailSenderImpl();
        this.applyProperties(properties, sender);
        return sender;
    }
```

MailProperties内容

```java
//可以定义的参数
public class MailProperties {
    private static final Charset DEFAULT_CHARSET;
    private String host;
    
    private Integer port;
    private String username;
    private String password;
    private String protocol = "smtp";
    private Charset defaultEncoding;
    private Map<String, String> properties;
    private String jndiName;
```

配置文件

```properties
# smtp的主机地址
spring.mail.host=smtp.qq.com
spring.mail.port=465
spring.mail.username=邮箱地址
spring.mail.password=邮箱授权码
spring.mail.properties.mail.smtp.ssl.enable=true
```

测试邮件发送**

```java
    @Value("${spring.mail.username}")
    String from;
    @Autowired
    JavaMailSenderImpl mailSender;

    @Test
    void contextLoads() {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setSubject("通知开会");
        message.setText("今晚7：30开会");
        message.setFrom(from);
        message.setTo("fangjie1229@163.com");
        mailSender.send(message);
    }

    @Test
    void test01() {
        //复杂邮件
        MimeMessage message = mailSender.createMimeMessage();
        try {
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            //邮件信息
            helper.setSubject("通知开会");
            //发送html内容
            helper.setText("<h1>今天7:30开会</h1>",true);
            helper.setTo(to);
            helper.setFrom(from);
            //附件
            helper.addAttachment("1.jpg", new File("D:\\Saved Pictures\\1.jpg"));
            helper.addAttachment("1.jpg", new File("D:\\Saved Pictures\\1.jpg"));
            //发送邮件
            mailSender.send(message);
        } catch (MessagingException e) {
            e.printStackTrace();
        }
    }
```



## 五、SpringBoot与安全

### 简介

常用场景：用户身份认证、权限控制、漏洞攻击

软件：apach shiro、Spring Security

SpringBoot底层使用Spring Security作为安全工具

Spring Security是针对Spring项目的安全框架，也是Spring Boot底层安全模块默认的技术选型。他可以实现强大的web安全控制。对于安全控制，我们仅需引入spring-boot-starter-security模块，进行少量的配置，即可实现强大的安全管理。  

### “认证”和“授权”

应用程序的两个主要区域是“认证”和“授权”（或者访问控制）。
这两个主要区域是Spring Security 的两个目标。
• “认证”（Authentication）， 是建立一个他声明的主体的过程（一个“主体”一般是指用户，设备或一些可以在你的应用程序中执行动作的其他系统） 。
• “授权”（Authorization）指确定一个主体是否允许在你的应用程序执行一个动作的过程。为了抵达需要授权的店，主体的身份已经有认证过程建立。

| **认证**                                                     | **授权**                                                   |
| ------------------------------------------------------------ | ---------------------------------------------------------- |
| 身份验证确认您的身份以授予对系统的访问权限。                 | 授权确定您是否有权访问资源。                               |
| 这是验证用户凭据以获得用户访问权限的过程。                   | 这是验证是否允许访问的过程。                               |
| 它决定用户是否是他声称的用户。                               | 它确定用户可以访问和不访问的内容。                         |
| 身份验证通常需要用户名和密码。                               | 授权所需的身份验证因素可能有所不同，具体取决于安全级别。   |
| 身份验证是授权的第一步，因此始终是第一步。                   | 授权在成功验证后完成。                                     |
| 例如，特定大学的学生在访问大学官方网站的学生链接之前需要进行身份验证。这称为身份验证。 | 例如，授权确定成功验证后学生有权在大学网站上访问哪些信息。 |

### SpringBoot引入Spring Security

参考[SpringBoot集成Spring Security](https://www.cnblogs.com/zhengqing/p/11612654.html)

#### 步骤

1. 登陆/注销
   – HttpSecurity配置登陆、注销功能
2. Thymeleaf提供的SpringSecurity标签支持
   – 需要引入thymeleaf-extras-springsecurity4
   – sec:authentication=“name” 获得当前用户的用户名
   – sec:authorize=“hasRole(‘ADMIN’)” 当前用户必须拥有ADMIN权限时才会显示标签内容
3. remember me
   – 表单添加remember-me的checkbox
   – 配置启用remember-me功能
4. CSRF（Cross-site request forgery）跨站请求伪造
   – HttpSecurity启用csrf功能，会为表单添加_csrf的值，提交携带来预防CSRF；  

#### 搭建环境

**导入POM坐标**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<!-- thymeleaf中引入security-->
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**配置文件**

```properties
spring.thymeleaf.cache=false
```

#### 测试

**自定义配置类**

```java
@EnableWebSecurity
public class MySecurityConfig extends WebSecurityConfigurerAdapter {

    /**
     * 开启授权功能
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //定义认证规则
        http.authorizeRequests()
                //资源路径和默认页面允许所有人访问
                .antMatchers("/css/**", "/index").permitAll()
                .antMatchers("/level1/**").hasRole("VIP1")
                .antMatchers("/level2/**").hasRole("VIP2")
                .antMatchers("/level3/**").hasRole("VIP3")
                .and()
                //开启自动配置的登录功能
                //访问/login,表示用户登录,SpringBoot自动配置了这个页面
                .formLogin()
                //自定义登录页面，如果自定义登录页面，则默认的/login页面不存在
                /**
                 * SpringBoot默认处理方式
                 *      /login GET 进入认证页面
                 *      /login POST 提交认证请求
                 * 如果使用自定义页面
                 *      /userlogin GET 进入认证页面
                 *      /userlogin POST 提交认证请求
                 */
                .loginPage("/userlogin")
                .loginProcessingUrl("/userlogin")
                //自定义用户名和密码的name
//                .usernameParameter("user")
//                .passwordParameter("pass")
                .and()
                //访问/logoug,表示用户注销,SpringBoot自动配置了这个页面
                .logout()
                //注销成功后返回主页
                .logoutSuccessUrl("/")
                .and()
                //开启保存账户密码功能，保存在cookie中
                .rememberMe()
                //默认checkbox的name为"remember-me"
                .rememberMeParameter("remember");
    }

    /**
     * 从数据库中获取用户信息放入内存
     * @return
     */
//    @Bean
//    public UserDetailsService userDetailsService() {
//        UserDetails userDetails = User.withDefaultPasswordEncoder()
//                .username("user")
//                .password("password")
//                .roles("USER")
//                .build();
//        return new InMemoryUserDetailsManager(userDetails);
//    }

    /**
     * 开启认证功能
     * 从数据库中获取用户的信息，放入内存
     * @param auth
     * @throws Exception
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
//        super.configure(auth);
        //设置密码加密器
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        // 在内存中配置用户，配置多个用户调用`and()`方法
        auth.inMemoryAuthentication()
                .passwordEncoder(encoder)
                .withUser("张三").password(encoder.encode("123456")).roles("VIP1","VIP2")
                .and()
                .withUser("李四").password(encoder.encode("123456")).roles("VIP2","VIP3");
    }

}
```

**测试页面**

使用thymeleaf与security-thymeleaf来编写页面

welcome.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" xmlns:sec="https://www.thymeleaf.org/spring-security">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Insert title here</title>
    </head>
    <body>
        <h1 align="center">欢迎光临武林秘籍管理系统</h1>
        <h2 sec:authorize="!isAuthenticated()" align="center">游客您好，如果想查看武林秘籍 <a th:href="@{/userlogin}">请登录</a></h2>
        <!-- 方式为post-->
        <div sec:authorize="isAuthenticated()">
            Logged in user: <span sec:authentication="name"></span>
            Roles: <span sec:authentication="principal.authorities"></span><br/>
            <form th:action="@{/logout}" method="post">
                <input type="submit" value="注销">
            </form>
        </div>
        <hr>

        <div sec:authorize="hasRole('VIP1')">
            <h3>普通武功秘籍</h3>
            <ul>
                <li><a th:href="@{/level1/1}">罗汉拳</a></li>
                <li><a th:href="@{/level1/2}">武当长拳</a></li>
                <li><a th:href="@{/level1/3}">全真剑法</a></li>
            </ul>
        </div>
        <div sec:authorize="hasRole('VIP2')">
            <h3>高级武功秘籍</h3>
            <ul>
                <li><a th:href="@{/level2/1}">太极拳</a></li>
                <li><a th:href="@{/level2/2}">七伤拳</a></li>
                <li><a th:href="@{/level2/3}">梯云纵</a></li>
            </ul>
        </div>
        <div sec:authorize="hasRole('VIP3')">
            <h3>绝世武功秘籍</h3>
            <ul>
                <li><a th:href="@{/level3/1}">葵花宝典</a></li>
                <li><a th:href="@{/level3/2}">龟派气功</a></li>
                <li><a th:href="@{/level3/3}">独孤九剑</a></li>
            </ul>
        </div>

    </body>
</html>
```

login.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Insert title here</title>
    </head>
    <body>
        <h1 align="center">欢迎登陆武林秘籍管理系统</h1>
        <hr>
        <div align="center">
            <form th:action="@{/userlogin}" method="post">
                用户名：<input type="text" name="username"/><br>
                密码：<input type="password" name="password"/><br/>
                记住我：<input type="checkbox" name="remember"/>
                <input type="submit" value="登陆">
            </form>
        </div>
    </body>
</html>
```

## 六、SpringBoot与分布式

### 分布式架构简介

在分布式系统中，国内常用zookeeper+dubbo组合，而Spring Boot推荐使用全栈的Spring， Spring Boot+Spring Cloud  

![image-分布式系统](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200402172212245.png)

- **单一应用架构**
  当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。
- **垂直应用架构**
  当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的Web框架(MVC)是关键。
- **分布式服务架构**
  当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的分布式服务框架(RPC)是关键。
- **流动计算架构**
  当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键。  

#### ORM框架

​	对象关系映射ORM（Object Relational Mapping）是随着面向对象的软件开发方法发展而产生的。面向对象的开发方法是当今企业级应用开发环境中的主流开发方法，关系数据库是企业级应用环境中永久存放数据的主流数据存储系统。对象和关系数据是业务实体的两种表现形式，业务实体在内存中表现为对象，在数据库中表现为关系数据。内存中的对象之间存在关联和继承关系，而在数据库中，关系数据无法直接表达多对多关联和继承关系。因此，对象-关系映射(ORM)系统一般以中间件的形式存在，主要实现程序对象到关系数据库数据的映射。

#### MVC框架

​	MVC（Model View Controller）是一种架构设计模式，该模式主要应用于图形化用户界面(GUI)应用程序。MVC由Model（模型）、View（视图）及Controller（控制器）三部分组成

#### RPC框架

​	RPC（Remote Procedure Call Protocol）是远程过程调用协议。简单来说就是有两台服务器A，B，一个应用部署在A上想要调用B服务器应用提供的函数方法，由于不在一个内存空间，不能直接进行调用，需要通过网络来表达调用的语义和传达调用的数据。

RPC框架需要解决的几点问题：

- 通讯问题

  ​	主要是在客户端和服务器之间建立Tcp连接，远程过程调用的所有交换的数据都在这个连接里传输。连接可以是按需连接，调用结束后就断掉，也可以是长连接，多个远程过程调用共享同一个连接。

- 寻址的问题

  ​	A服务器上的应用告诉底层的RPC框架，如何连接到B服务器（如主机或IP地址）以及特定的端口，方法的名称是什么，这样才能完成调用。比如基于Web服务协议的RPC，就要提供一个endpoint URI,或者是从UDDI服务上查找（web service 中具体用用了rpc框架）。如果是RMI调用的话，还需要一个RMI Registry来注册服务的地址。

- 序列化与反序列化

  ​	当 A服务器上的应用远程过程调用时，方法的参数需要通过底层的网络协议如TCP传递到B服务器 ，由于网络协议是基于二进制的，内存中的参数的值要序列化成二进制的形式，通过寻址和传输将序列化的二进制发送给B服务器。

  ​	当B服务器收到请求后，需要对参数进行反序列化，恢复成内存中的传递方式，然后找到对应的方法进行本地回调，然后得到返回值。

  ​	返回值还要发送回服务器A上的应用，也要经过序列化的方式发送，服务器A接到后，再反序列化，恢复为内存中的表达式

#### SOA框架

​	SOA（Service Oriented Architecture）是一个组件模型，它将应用程序的不同功能单元（称为服务）进行拆分，并为这些服务之间定义良好的接口和契约从而联系起来。接口是采用中立的方式进行定义的，它应该独立于实现服务的硬件平台、操作系统和编程语言。这使得构建在各种各样的系统中的服务可以以一种统一和通用的方式进行交互。
​	当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。 此时，用于提高机器利用率的==资源调度和治理中心(SOA)== 是关键。

​	主要功能：

- 透明化的远程方法调用
  像调用本地方法一样调用远程方法；只需简单配置，没有任何API侵入。

- 软负载均衡及容错机制
  可在内网替代nginx lvs等硬件负载均衡器。

- 服务注册中心自动注册 & 配置管理
  -不需要写死服务提供者地址，注册中心基于接口名自动查询提供者ip。
  使用类似zookeeper等分布式协调服务作为服务注册中心，可以将绝大部分项目配置移入zookeeper集群。

- 服务接口监控与治理（服务被谁调用、谁调用了哪些服务）
  -Dubbo-admin与Dubbo-monitor提供了完善的服务接口管理与监控功能，
  针对不同应用的不同接口，可以进行多版本，多协议，多注册中心管理。

#### ZooKeeper和Dubbo

**ZooKeeper**
ZooKeeper 是一个分布式的，开放源码的分布式应用程序协调服务。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、
域名服务、分布式同步、组服务等。
**Dubbo**
Dubbo是Alibaba开源的分布式服务框架，它最大的特点是按照分层的方式来架构，使用这种方式可以使各个层之间解耦合（或者最大限度地松耦
合）。从服务模型的角度来看， Dubbo采用的是一种非常简单的模型，要么是提供方提供服务，要么是消费方消费服务，所以基于这一点可以抽象
出服务提供方（Provider）和服务消费方（Consumer）两个角色。  

Dubbo架构具有连通性、健壮性、伸缩性、升级性的特点

```
连通性
注册中心负责服务地址的注册于查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小。
监控中心负责统计个服务调用次数，调用时间等，统计在内存汇总后每分钟一次发送到监控中心服务器，以报表展示。
服务提供者向注册中心注册其提供的服务，并汇报调用时间到监控中心，此时间不包含网络开销。
服务消费者向注册中心获取服务提供者地址列表，并根据负载均衡算法直接调用提供者，同时汇报调用时间到监控中心，此时间包含网络开销。
注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外。
注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者。
注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表。
注册中心和监控中心都是可选的，服务消费者可以直连服务提供者。

健壮性
监控中心宕掉不影响使用，只是丢失部分采样数据。
注册中心对等集群，任意一台宕掉后，将自动切换到另一台。
注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯。
服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重新等待服务提供者恢复。

伸缩性
注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心。
服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者
```

![Dubbo架构](D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\14.jpg)

<img src="D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200402211041006.png" alt="image-Dubbo架构-2" style="zoom:67%;" />

Provider：暴露服务的服务提供方
Consumer：调用远程服务的服务于消费方
Register：用于服务注册和发现服务的注册中心
Monitor：统计服务调用次数和调用时间的监控中心
Container：服务运行的容器

**调用关系如下**
0：服务容器负责启动、加载、运行服务提供者
1：服务提供者启动时向注册中心注册自己提供的服务
2：服务消费者启动时向注册中心订阅自己需要的服务
3：注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送更新后的数据给消费者
4：服务消费者从地址列表中，基于软负载均衡算法，选一个服务提供者调用，如果调用失败，再选另一个调用
5：服务提供者和服务消费者在内存中累计次数和调用时间，定时发送一次统计数据到监控中心

### SpringBoot整合ZooKeeper和Dubbo

#### 安装ZooKeeper作为注册中心

```shell
docker pull zookeeper
docker run --name some-zookeeper -p 2181:2181 --restart always -d zookeeper
2181: the zookeeper client port
2888: follower port
3888: election port
```

#### 整合ZooKeeper和Dubbo

##### 配置服务提供者

**1、导入POM坐标**

```xml
<properties>
   <java.version>1.8</java.version>
   <dubbo.version>2.7.5</dubbo.version>
   <zkclient.version>0.11</zkclient.version>
</properties>
<dependencies>
	<!-- SpringBoot web服务-->
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <exclusions>
         <exclusion>
            <artifactId>log4j-api</artifactId>
            <groupId>org.apache.logging.log4j</groupId>
         </exclusion>
      </exclusions>
   </dependency>
   <!-- dubb相关依赖-->
   <dependency>
      <groupId>org.apache.dubbo</groupId>
      <artifactId>dubbo-spring-boot-starter</artifactId>
      <version>${dubbo.version}</version>
      <exclusions>
         <exclusion>
            <groupId>org.springframework</groupId>
            <artifactId>spring</artifactId>
         </exclusion>
         <exclusion>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
         </exclusion>
         <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
         </exclusion>
      </exclusions>
   </dependency>
   <dependency>
      <groupId>org.apache.dubbo</groupId>
      <artifactId>dubbo</artifactId>
      <version>${dubbo.version}</version>
   </dependency>
   <dependency>
      <groupId>org.apache.curator</groupId>
      <artifactId>curator-framework</artifactId>
      <version>4.0.1</version>
   </dependency>
   <dependency>
      <groupId>org.apache.curator</groupId>
      <artifactId>curator-recipes</artifactId>
      <version>2.8.0</version>
      <exclusions>
         <exclusion>
            <artifactId>log4j</artifactId>
            <groupId>log4j</groupId>
         </exclusion>
      </exclusions>
   </dependency>
   <!-- zookeeper的客户端依赖-->
   <dependency>
      <groupId>com.101tec</groupId>
      <artifactId>zkclient</artifactId>
      <version>${zkclient.version}</version>
   </dependency>
	<!-- SpringBoot单元测试-->
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
      <exclusions>
         <exclusion>
            <groupId>org.junit.vintage</groupId>
            <artifactId>junit-vintage-engine</artifactId>
         </exclusion>
      </exclusions>
   </dependency>
</dependencies>
```

**2、配置dubbo环境**

```properties
# dubbo应用的名称
dubbo.application.name=provider-ticket
# ZooKeeper客户端地址
dubbo.registry.address=zookeeper://49.233.134.44:2181
# 扫描的包
dubbo.scan.base-packages=com.atguigu.ticket.service
```

**3、使用@Service发布服务**

服务接口

```java
/**
* 接口路径
* com.atguigu.ticket.service.ITicketService
**/
public interface ITicketService {
    String getTicket();
}
```

具体服务

```java
import com.atguigu.ticket.service.ITicketService;
import org.apache.dubbo.config.annotation.Service;

@org.springframework.stereotype.Service
/**
* 将服务注册到ZooKeeper注册中心
* 注意：
* 	@Service是dubbo包下的
**/
@Service
public class TicketServiceImpl implements ITicketService {
    @Override
    public String getTicket() {
        return "《厉害了，我的国》";
    }
}
```

##### 配置服务消费者

**1、导入POM坐标**

和服务提供者POM坐标一致

**2、配置dubbo环境**

```properties
# dubbo应用的名称
dubbo.application.name=consumer-ticket
# ZooKeeper客户端地址
dubbo.registry.address=zookeeper://49.233.134.44:2181
```

**3、使用@Reference调用远程服务**

首先需要在本地新建一个与远程服务提供者中路径一样的服务接口

```java
/**
* 路径
* com.atguigu.ticket.service.ITicketService
**/
public interface ITicketService {
    String getTicket();
}
```

<figure class="half">
    <img src="D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200403133309295.png" alt="RPC-服务提供者" style="width: 30%;display: inline;" />
	<img src="D:\Dailylearning\J2EE\springboot\Springboot整合\笔记\SpringBoot 高级.assets\image-20200403131342392.png" alt="RPC-服务消费者" style="width: 30%;display: inline;" />
</figure>


调用服务

```java
import com.atguigu.ticket.service.ITicketService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    /**
    * 从ZooKeepr注册中心获取服务地址列表，并使用服务
    * 注意：
    * 	@Reference是dubbo包下的
    * 	服务提供者正在运行
    */
    @Reference
    ITicketService ticketService;

    public void hello(){
        String ticket = ticketService.getTicket();
        System.out.println("买到票"+ticket);
    }
}
```

```java
//测试RPC
@SpringBootTest
class ConsumerUserApplicationTests {
    @Autowired
    UserService service;
    
    @Test
    void contextLoads() {
        service.hello();
    }
}
```

### Spring Cloud

​	Spring Cloud是一个分布式的整体解决方案。 Spring Cloud 为开发者提供了在分布式系统（配置管理，服务发现，熔断，路由，微代理，控制总线，一次性token，全局琐， leader选举，分布式session，集群状态）中快速构建的工具，使用Spring Cloud的开发者可以快速的启动服务或构建应用、同时能够快速和云平台资源进行对接  
**SpringCloud分布式开发五大常用组件**
​	服务发现（注册中心）——Netflix Eureka
​	客服端负载均衡——Netflix Ribbon
​	断路器——Netflix Hystrix
​	服务网关——Netflix Zuul
​	分布式配置——Spring Cloud Config  

### SpringBoot整合Spring Cloud

#### 配置Eureka注册中心

**1、使用SpringBoot Initializer，选用Spring Cloud Discovery模块中Eureka Server**

```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

**2、应用配置**

```yaml
server:
  port: 8761 #服务端口号
eureka:
  instance:
    hostname: eureka-server # eureka实例名
  client:
    register-with-eureka: false # 不把自己注册到eureka上，高可用的时候注册
    fetch-registry: false # 不从euraka上来获取服务器的注册信息，因为本身就是注册中心
    service-url: # 注册中心服务注册地址，Map类型
      defaultZone: http://localhost:8761/eureka/
```

**3、使用@EnableEurekaServer启动Eureka Server**

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekeServerApplication {

   public static void main(String[] args) {
      SpringApplication.run(EurekeServerApplication.class, args);
   }

}
```

**4、访问Eureka控制台** 

http://localhost:8761/

#### 编写多个服务提供者

**1、使用SpringBoot Initializer，选用web和Eureka Server模块**

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**2、应用配置**

```yaml
server:
  port: 8001
spring:
  application:
    name: provider-ticket
eureka:
  instance:
    prefer-ip-address: true # 注册服务的时候使用服务的ip地址
  client:
    service-url: # 注册中心服务注册地址，Map类型
      defaultZone: http://localhost:8761/eureka/
```

**3、配置服务**

```java
@Service
public class TicketService {
    public String getTicket(){
        System.out.println("port：8001");
        return  "《厉害了，我的国》";
    }
}
```

**4、以http方式暴露服务**

```java
@RestController
public class TicketController {
    @Autowired
    TicketService service;

    @RequestMapping(method = RequestMethod.GET, path = "/ticket")
    public String getTicket() {
        return service.getTicket();
    }
}
```

**5、将项目打成jar包**

**6、将上面项目中端口号改为8002，重复上述步骤**

**7、以java -jar命令运行两个jar包**

<img src="D:%5CDailylearning%5CJ2EE%5Cspringboot%5CSpringboot%E6%95%B4%E5%90%88%5C%E8%AF%BE%E4%BB%B6%5C%E6%96%87%E6%A1%A3%5CSpringBoot%20%E9%AB%98%E7%BA%A7.assets%5Cimage-20200403161406087.png" alt="image-运行多个服务提供者" style="zoom:80%;" />

**8、查看Eureka注册中心情况**

<img src="D:%5CDailylearning%5CJ2EE%5Cspringboot%5CSpringboot%E6%95%B4%E5%90%88%5C%E8%AF%BE%E4%BB%B6%5C%E6%96%87%E6%A1%A3%5CSpringBoot%20%E9%AB%98%E7%BA%A7.assets%5Cimage-20200403161643183.png" alt="image-20200403161643183" style="zoom:80%;" />



#### 配置服务消费者

**1、使用SpringBoot Initializer，选用web和Eureka Server模块**

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**2、应用配置**

```yaml
spring:
  application:
    name: consumer-user # 注册中心中显示的名字
server:
  port: 8200
eureka:
  instance:
    prefer-ip-address: true # 注册服务的时候使用服务的ip地址
  client:
    service-url: # 注册中心服务注册地址，Map类型
      defaultZone: http://localhost:8761/eureka/
```

**3、使用@EnableDiscoveryClient开启服务发现功能**

添加RestTemplate，用于发送http请求

同时开启==负载均衡机制==，当存在多个相同服务时，会轮训调用服务

```java
//开启服务发现功能
@EnableDiscoveryClient
@SpringBootApplication
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

    //使用负载均衡机制：轮训机制，一个服务器访问一次
    @LoadBalanced
    @Bean
    /**
     * 用于发送http请求模板
     */
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

**4、使用RestTemplate远程调用服务**

```java
@RestController
public class UserController {

    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/buy")
    public String buyTicket(String name) {
        /**
         *  url格式：
         *      http://应用名/服务名
         */
        String object = restTemplate.getForObject("http://PROVIDER-TICKET/ticket", String.class);
        return name + "购买了" + object;
    }
}
```



## 七、SpringBoot与开发热部署

​	在开发中我们修改一个Java文件后想看到效果不得不重启应用，这导致大量时间花费，我们希望不重启应用的情况下，程序可以自动部署（热部署）。有以下四种情况，如何能实现热部署  

1、模板引擎
– 在Spring Boot中开发情况下禁用模板引擎的cache
– 页面模板改变ctrl+F9可以重新编译当前页面并生效  

2、 Spring Loaded
Spring官方提供的热部署程序，实现修改类文件的热部署
– 下载Spring Loaded（项目地址https://github.com/springprojects/spring-loaded）
– 添加运行时参数；
-javaagent:C:/springloaded-1.2.5.RELEASE.jar –noverify  

3、 JRebel
– 收费的一个热部署软件
– 安装插件使用即可  

4、 Spring Boot Devtools（推荐）
– 引入依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
</dependency>  
```

– ==IDEA使用ctrl+F9==
– Eclipse直接使用ctrl+S保存就行。Eclipse设置了自动编译之后，修改类它会自动编译
Intellij IEDA和Eclipse不同， Eclipse设置了自动编译之后，修改类它会自动编译，而IDEA在非RUN或DEBUG情况下
才会自动编译（前提是你已经设置了Auto-Compile）。
• 设置自动编译（settings-compiler-make project automatically）
• ctrl+shift+alt+/（maintenance）
• 勾选compiler.automake.allow.when.app.running  

## 八、SpringBoot与监控管理

参考网址：[SpringBoot Actuator应用监控](https://www.cnblogs.com/zwqh/p/11851300.html)

### Actuator简介

​	Actuator 是 Spring Boot 提供的对应用系统的自省和监控功能。通过 Actuator，可以使用数据化的指标去度量应用的运行情况，比如查看服务器的磁盘、内存、CPU等信息，系统的线程、gc、运行状态等。我们可以通过HTTP， JMX， SSH协议来进行操作，自动得到审计、健康及指标信息等  

#### Actuator端口说明

| 端点             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| auditevents      | 获取当前应用暴露的审计事件信息                               |
| beans            | 获取应用中所有的 Spring Beans 的完整关系列表（bean 的别名、类型、是否单例、类的地址、依赖等信息） |
| caches           | 获取公开可以用的缓存                                         |
| conditions       | 获取自动配置条件信息，记录哪些自动配置条件通过和没通过的条件 |
| configprops      | 获取所有配置属性，包括默认配置，显示一个所有 @ConfigurationProperties 的整理列版本 |
| env              | 获取所有关于当前 Spring Boot 应用程序的运行环境信息，如：操作系统信息（systemProperties）、环境变量信息、JDK 版本及 ClassPath 信息、当前启用的配置文件（activeProfiles）、propertySources、应用程序配置信息（applicationConfig）等 |
| flyway           | 获取已应用的所有Flyway数据库迁移信息，需要一个或多个 Flyway Bean |
| liquibase        | 获取已应用的所有Liquibase数据库迁移。需要一个或多个 Liquibase Bean |
| health（常用）   | 获取应用程序健康指标（运行状况信息）,监控软件通常使用该接口实时监测应用运行状况，在系统出现故障时把报警信息推送给相关人员，如磁盘空间使用情况、数据库和缓存等的一些健康指标。 |
| heapdump         | 返回hprof堆转储文件                                          |
| httptrace        | 获取HTTP跟踪信息（默认情况下，最近100个HTTP请求-响应交换）。需要 HttpTraceRepository Bean |
| info             | 获取应用程序信息（application.properties 中配置）            |
| integrationgraph | 显示 Spring Integration 图。需要依赖 spring-integration-core |
| jolokia          | 通过HTTP公开JMX bean（当Jolokia在类路径上时，不适用于WebFlux）。需要依赖 jolokia-core |
| loggers          | 显示和修改应用程序中日志的配置                               |
| logfile          | 返回日志文件的内容（如果已设置logging.file.name或logging.file.path属性） |
| metrics          | 获取系统度量指标信息（jvm参数、cpu线程使用情况等）           |
| mappings         | 显示所有@RequestMapping路径的整理列表                        |
| prometheus       | 以Prometheus服务器可以抓取的格式公开指标。需要依赖 micrometer-registry-prometheus |
| scheduledtasks   | 显示应用程序中的计划任务                                     |
| sessions         | 允许从Spring Session支持的会话存储中检索和删除用户会话。需要使用Spring Session的基于Servlet的Web应用程序 |
| shutdown         | 关闭应用。要求management.endpoint.shutdown.enabled设置为true，默认为 false。以post方式请求http://127.0.0.1:8080/actuator/shutdown关闭应用 |
| threaddump       | 获取系统线程转储信息（线程名、线程ID、线程的状态、是否等待锁资源等信息） |

### SpringBoot整合Actuator

#### 测试各个监控端点

**1、导入POM坐标**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- 应用热部署-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

**2、应用配置**

```yaml
management:
  endpoints:
#    设置为false，表示关闭所有端点访问
#    设置为true，表示开启默认端口，actuator默认只开放health和info两个端点
    enabled-by-default: true
    web:
      exposure:
#        启动所有端点
        include: "*"
#        禁用部分端点
        exclude:
          - "env"
          - "beans"
#       自定义管理端点路径
#      base-path: /manage
  endpoint:
    health:
#      显示具体健康指标
      show-details: always
info:
  app:
    name: "Spring Boot Actuator Demo"
    version: "v1.0.0"
    description: "Spring Boot Actuator Demo"
```

**3、访问所有Actuator端点列表**

http://localhost:8080/actuator

#### 自定义HealthIndicator

由于默认的health端点就显示了diskSpace和ping信息，如果有需要可以添加自定义信息。

SpringBoot默认为许多组件自动配置了许多HealthIndicator，如RedisHealthIndicator，MongoHealthIndicator等，一旦使用这些组件，就会将组件信息添加到health端点

我们也可以自定义信息添加到health端点中

```java
@Component
public class MyAppHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        /**
         * 自定义检查方法
         *     健康：
         *          返回Health.up().build()
         *     不健康：
         *          返回Health.down().withDetail("msg","服务异常").build()
         */
        return Health.down().withDetail("msg","服务异常").build();
    }
}
```

![image-自定义Health检查方法](D:%5CDailylearning%5CJ2EE%5Cspringboot%5CSpringboot%E6%95%B4%E5%90%88%5C%E8%AF%BE%E4%BB%B6%5C%E6%96%87%E6%A1%A3%5CSpringBoot%20%E9%AB%98%E7%BA%A7.assets%5Cimage-20200403235743880.png)

