1.spring boot starter 自加载是怎么实现的？

2.在生命周期哪个阶段？ 

3.自定starter





## 4.总结

由上所述， `starter`的大体的工作流程：

- **`SpringBoot`启动时会自动搜索包含`spring.factories`文件的JAR包；**
- **根据`spring.factories`文件加载自动配置类`AutoConfiguration`；**
- **通过`AutoConfiguration`类，加载满足条件`(@ConditionalOnXxx)`的`bean`到`Spring IOC`容器中；**
- **使用者可以直接使用自动加载到`IOC`的`bean`**



SpringBoot的自动化配置让我们的开发彻底远离了Spring繁琐的各种配置，让我们专注于开发，但是SpringBoot的自动化配置是怎么实现的呢？

SpringBoot最为重要的一个注解是@SpringBootApplication，它其实是一个组合元注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

从这个注解可以看出来，它包含了@EnableAutoConfiguration 这 个注解，这个注解就是自动化配置的原理核心所在：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    /**
     * Exclude specific auto-configuration classes such that they will never be applied.
     * @return the classes to exclude
     */
    Class<?>[] exclude() default {};

    /**
     * Exclude specific auto-configuration class names such that they will never be
     * applied.
     * @return the class names to exclude
     * @since 1.3.0
     */
    String[] excludeName() default {};

}
```

很明显，@EnableAutoConfiguration  这个注解使用的是第二种情况，导入@Import(AutoConfigurationImportSelector.class) 类，借助于AutoConfigurationImportSelector， @EnableAutoConfiguration 可以帮助SpringBoot 应用将所有符合条件的@Configuration 配置都加载到当前SpringBoot 创建并使用IoC容器。



借助于Spring框架原有的一个工具类，SpringFactoriesLoader 的支持，@EnableAutoConfiguration 可以智能的自动配置功效才得以大功告成！



https://segmentfault.com/a/1190000020550458



## 1. SpringBoot Starter源码分析

 **Q：`@SpringBootApplication` 注解中核心注解`@EnableAutoConfiguration`注解在starter起什么作用呢?**

**`@EnableAutoConfiguration`源码分析：**

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
    Class<?>[] exclude() default {};
    String[] excludeName() default {};
}
```

  可以从源码看出关键功能是`@import`注解导入自动配置功能类`AutoConfigurationImportSelector`类，主要方法`getCandidateConfigurations()`使用了`SpringFactoriesLoader.loadFactoryNames()`方法加载META-INF/spring.factories的文件（spring.factories声明具体自动配置）。

```
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
            AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
                getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
        Assert.notEmpty(configurations,
                "No auto configuration classes found in META-INF/spring.factories. If you "
                        + "are using a custom packaging, make sure that file is correct.");
        return configurations;
}
```

**Q：通常情况下，`starter`会根据条件进行操作处理，比如根据不同条件创建不同`Bean`。在`SpringBoot`有哪些注解可用呢?**  
可使用`org.springframwork.boot.autoconfigure.condition`的条件注解，具体如下所示：

| 注解                            | 解析                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| @ConditionalOnBean              | 当容器里有指定的Bean的条件下。                               |
| @ConditionalOnClass             | 当类路径下有指定的类的条件下。                               |
| @ConditionalOnExpression        | 基于SpEL表达式作为判断条件。                                 |
| @ConditionalOnJava              | 基于JVM版本作为判断条件。                                    |
| @ConditionalOnJndi              | 在JNDI存在的条件下查找指定的位置。                           |
| @ConditionalOnMissingBean       | 当容器里没有指定Bean的情况下。                               |
| @ConditionalOnMissingClass      | 当类路径下没有指定的类的条件下。                             |
| @ConditionalOnNotWebApplication | 当前项目不是Web项目的条件下。                                |
| @ConditionalOnProperty          | 指定的属性是否有指定的值。                                   |
| @ConditionalOnResource          | 类路径是否有指定的值。                                       |
| @ConditionalOnSingleCandidate   | 当指定Bean在容器中只有一个， 或者虽然有多个但是指定首选的Bean。 |
| @ConditionalOnWebApplicatio     | 当前项目是Web项目的条件下。                                  |

#### SpringFactoriesLoader

SpringFactoriesLoader属于Spring框架私有的一种扩展方案(类似于Java的SPI方案java.util.ServiceLoader)，其主要功能就是从指定的配置文件META-INF/spring-factories加载配置，spring-factories是一个典型的java properties文件，只不过Key和Value都是Java类型的完整类名，比如：



```xml
#-------starter自动装配---------
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.demo.starter.config.MsgAutoConfiguration
```

https://www.jianshu.com/p/2a446ce48803

## 2. 自定starter

在此将模拟公司获取生产密码模块进行自定义`starter demo`

### 2.1 核心依赖

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.1.5.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
    </dependency>
</dependencies>
```

### 2.2 服务类service以及属性配置注入

`PasswordService`服务类：

```
public class PasswordService {
    //第三方系统获取密码所需的key
    private String objectKey;
    @Autowired
    //模拟的第三方系统service
    private ThirdPartySystemService thirdPartySystemService;

    public String getSystemPassword(String objectKey,String originalPassord){

            if(StringUtils.isEmpty(objectKey)){
                return  originalPassord;
            }
            //从第三方系统获取密码
            String password= thirdPartySystemService.getPassword(objectKey);
            //返回密码
            return password!=null?password:originalPassord;

    }
}

//模拟第三方系统service
public class ThirdPartySystemService {
    public String getPassword(String objectKey){
        //返回一个32位随机数
        return UUID.randomUUID().toString();
    }
}
```

**属性配置类：**

```
//通过@ConfigurationProperties注解获取属性值
@ConfigurationProperties(prefix = "project.starter")
public class BaseServiceProperties {
    private String serviceName;
    private String serviceVersion;

    public String getServiceName() {
        return serviceName;
    }

    public void setServiceName(String serviceName) {
        this.serviceName = serviceName;
    }

    public String getServiceVersion() {
        return serviceVersion;
    }

    public void setServiceVersion(String serviceVersion) {
        this.serviceVersion = serviceVersion;
    }
}
```

**配置属性使用类：**

```
public class BaseStarterService {

    public void addServiceName(BaseServiceProperties baseServiceProperties){
        System.out.println("serviceName:"+baseServiceProperties.getServiceName()+"----"+"serviceVersion"+baseServiceProperties.getServiceVersion());
    }
}
```

**其他类：**

```
//判断是否windows系统
public class WindowsCondition implements Condition {

    private final static String WINDOWS="Windows";

    /**
     * ConditionContext：判断条件能使用的上下文（环境）
     * AnnotatedTypeMetadata：注释信息
     */
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        //获取当前环境变量
        Environment environment=conditionContext.getEnvironment();
        //获取bean注册器
        BeanDefinitionRegistry registry = conditionContext.getRegistry();
        //能获取到ioc使用的beanfactory
        ConfigurableListableBeanFactory beanFactory = conditionContext.getBeanFactory();
        //获取环境变量中操作系统
        String property = environment.getProperty("os.name");
        //判断操作系统是否为windows
        if(property.contains(WINDOWS)){
            //判断是否存在baseWindowsSevice类，不存在则进行bean注册
            boolean isWindowsSevice = registry.containsBeanDefinition("baseStarterService");
            if(!isWindowsSevice){
                //指定Bean定义信息；（Bean的类型，Bean的一系列信息）
                RootBeanDefinition beanDefinition = new RootBeanDefinition(BaseStarterService.class);
                //注册一个Bean，指定bean名
                registry.registerBeanDefinition("baseStarterService", beanDefinition);
                BaseStarterService windowsSevice = (BaseStarterService)beanFactory.getBean("baseStarterService");
            }
            return true;
        }
        return false;
    }
}
```

### 2.3自动配置类

代码解读：

- `@EnableConfigurationProperties`：读取配置文件的属性
- `@Import`：导入其他配置类或者自定义类
- `@Conditional`：判断当前环境是否为windows，是则注册该类
- `@ConditionalOnProperty`：判断属性`spring.project.ThirdPartySystemService.isPassword`是否等于`true`，不为`true`则不注册该类
- `@ConditionalOnClass`：判断IOC容器中是否存在`ThirdPartySystemService`类，存在则创建`PasswordService bean`



```
@Configuration
//自动加载配置文件属性值
@EnableConfigurationProperties(BaseServiceProperties.class)
@Import(BeanConfiguration.class)
//判断当前环境是否为windows
@Conditional(WindowsCondition.class)：
//判断属性spring.project.ThirdPartySystemService.isPassword是否等于true
@ConditionalOnProperty(prefix = "spring.project.ThirdPartySystemService",value = "enablePassword", havingValue = "true",matchIfMissing = true)
public class AutoConfigurationPassoword {
    @Autowired
    private BaseServiceProperties baseServiceProperties;
    @Autowired
    private BaseStarterService baseWindowsService;

    //加载第三方系统service
    @Bean("thirdPartySystemService")
    public ThirdPartySystemService thirdPartySystemService(){
        baseWindowsService.addServiceName(baseServiceProperties);
        return new ThirdPartySystemService();
    }
    @Bean
    //判断IOC容器中是否存在ThirdPartySystemService类，存在则创建PasswordService bean
    @ConditionalOnClass(ThirdPartySystemService.class)
    public PasswordService passwordService(){
        baseWindowsService.addServiceName(baseServiceProperties);
        return new PasswordService();
    }
}
```

### 2.4 注册配置

  **想自动配置生效， 需要注册自动配置类，即在`src/main/resources`下新建`METAINF/spring.factories`。在`spring.factorie`配置如下：**

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.cn.ccww.configuration.AutoConfigurationPassoword
```

**若有多个自动配置， 则用“**，**”隔开， 此处“”是为了换行后还能够读取到属性。**

## 3. 测试自定义starter

### 3.1 import 依赖

```
 <dependencies>
        <dependency>
            <artifactId>spring-boot-starter-base-service</artifactId>
            <groupId>com.cn.ccww</groupId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
</dependencies>
```

### 3.2 application.properties属性

`application.properties`文件有对应的字段是否启动自定义starter，还可以设置starter所需的属性。如下所示：

​        

```
//自定义Starter配置
//当该属性的值不为true时，才不会启动自定义starter
spring.project.ThirdPartySystemService.enablePassword=true
project.starter.serviceName=ccww
project.starter.serviceVersion=1.0
```









#### 总结

创建 自定义 starter 的步骤：

1. 确保在Pom.xml文件中声明了使用该组件所需要的全部dependency
2. 利用@ConfigurationProperties注解对外暴露恰当的properties
3. 利用条件注解@ConditionalXXX编写XXXAutoConfigration类
4. 把写好的的XXXAutoConfigration类加到META-INF/spring.factories文件的EnableAutoConfiguration配置中，这样在应用启动的时候就会自动加载XXXAutoConfiguration。



2.在生命周期哪个阶段？

ConfigurationClassPostProcessor类的职责是处理配置类；
ConfigurationClassPostProcessor是BeanDefinitionRegistryPostProcessor接口的实现类，它的postProcessBeanDefinitionRegistry方法在容器初始化阶段会被调用（BeanDefinitionRegistryPostProcessor接口的更多细节请参考《spring4.1.8扩展实战之六：注册bean到spring容器(BeanDefinitionRegistryPostProcessor接口)》）；
postProcessBeanDefinitionRegistry方法又调用processConfigBeanDefinitions方法处理具体业务；
processConfigBeanDefinitions方法中通过ConfigurationClassParser类来处理Configuration注解，如下图：

