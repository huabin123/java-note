# 概述

Spring Boot 提供了大量的自动配置，极大地简化了spring 应用的开发过程，当用户创建了一个 Spring Boot 项目后，即使不进行任何配置，该项目也能顺利的运行起来。当然，用户也可以根据自身的需要使用配置文件修改 Spring Boot 的默认设置。

SpringBoot 默认使用以下 2 种全局的配置文件，其文件名是固定的。

- application.properties
- application.yml

# YAML

application.yml 是一种使用 YAML 语言编写的文件，它与 application.properties 一样，可以在 Spring Boot 启动时被自动读取，修改 Spring Boot 自动配置的默认值。

## YAML语法

1. 使用缩进表示层级关系；
2. 缩进时不允许使用TAB键；
3. 缩进的空格数不重要，但同级元素必须左侧对齐；
4. 大小写敏感。

例如：

```yaml
spring:
  profiles: dev

  datasource:
    url: jdbc:mysql://127.0.01/banchengbang_springboot
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver
```

## YAML常用写法

YAML 支持以下三种数据结构：

- 字面量：单个的、不可拆分的值
- 对象：键值对的集合
- 数组：一组按次序排列的值

### YAML字面量写法

字面量是指单个的，不可拆分的值，例如：数字、字符串、布尔值、以及日期等。

在 YAML 中，使用“key:[空格]value**”**的形式表示一对键值对（空格不能省略），如 url: biancheng.net。

字面量直接写在键值对的“value**”**中即可，且默认情况下字符串是不需要使用单引号或双引号的。

```yaml
name: bianchengbang
```

### YAML对象写法

```YAML
website: 
  name: bianchengbang
  url: www.biancheng.net
```

### YAML数组写法

```yaml
pets:
  -dog
  -cat
  -pig
```

### 复合结构

```yaml
person:
  name: zhangsan
  age: 30
  pets:
    -dog
    -cat
    -pig
  car:
    name: QQ
  child:
    name: zhangxiaosan
    age: 2
```

## YAML 组织结构

一个 YAML 文件可以由一个或多个文档组成，文档之间使用“---**”**作为分隔符，且个文档相互独立，互不干扰。如果 YAML 文件只包含一个文档，则“---**”**分隔符可以省略。

```yaml
---
website:
  name: bianchengbang
  url: www.biancheng.net
---
website: {name: bianchengbang,url: www.biancheng.net}

pets:
  -dog
  -cat
  -pig

---
pets: [dog,cat,pig]

name: "zhangsan \n lisi"

---
name: 'zhangsan \n lisi'
```

# 配置绑定

所谓“配置绑定”就是把配置文件中的值与JavaBean中对应的属性进行绑定。

Spring Boot提供了一下2种方式进行配置绑定：

1. 使用`@ConfigrutionProperties`注解；
2. 使用@Value注解。

## @ConfigrutionProperties配置绑定

注意：

- 只有在容器中的组件，才会拥有 SpringBoot 提供的强大功能。如果我们想要使用 @ConfigurationProperties 注解进行配置绑定，那么首先就要保证该对 JavaBean 对象在 IoC 容器中，所以需要用到 @Component 注解来添加组件到容器中。
- JavaBean 上使用了注解 @ConfigurationProperties(prefix = "person") ，它表示将这个 JavaBean 中的所有属性与配置文件中以“person”为前缀的配置进行绑定。

## @Value获取一个特定配置

```yaml
    @Value("${person.lastName}")
    private String lastName;
```

## @Value 与 @ConfigurationProperties 对比

1. 使用位置不同
2. 功能不同（应用场景不同）



## @PropertySource

如果将所有的配置都集中到 application.properties 或 application.yml 中，那么这个配置文件会十分的臃肿且难以维护，因此我们通常会将与 Spring Boot 无关的配置（例如自定义配置）提取出来，写在一个单独的配置文件中，并在对应的 JavaBean 上使用 @PropertySource 注解指向该配置文件。

![SpringBoot person.properties](https://gitee.com/huawesome/my-picture/raw/master/img/202109281741574.png)

```java
@PropertySource(value = "classpath:person.properties")//指向对应的配置文件
@Component
@ConfigurationProperties(prefix = "person")
```

# Spring Boot默认配置文件及加载优先级

Spring Boot 启动时会扫描以下 5 个位置的 application.properties 或 apllication.yml 文件，并将它们作为 Spring boot 的默认配置文件。

1. file:./config/
2. file:./config/*/
3. file:./
4. classpath:/config/
5. classpath:/

> 注：file: 指当前项目根目录；classpath: 指当前项目的类路径，即 resources 目录。

以上所有配置文件都会被加载，且序号越小优先级越高。其次，对于想同位置的Application，properties的优先级高于application,yml。

![Spring Boot 配置文件加载顺序](https://gitee.com/huawesome/my-picture/raw/master/img/202109282119666.png)

# Spring配置加载顺序

以下是常用的Spring Boot配置形式及其加载顺序（优先级由高到低）：

1. 命令行参数
2. 来自java:com/env的JNDI属性
3. Java系统属性（System.getProperties()）
4. 操作系统环境变量
5. RandomValuePropertySource配置的random.*属性值
6. 配置文件（YAML文件、Properties文件）
7. @Configuration注解上的@PropertySource指定的配置文件
8. 通过SpringApplication.setDefaultProperties制定的默认属性

以上所有形式的配置都会加载，当存在相同配置内容时，高优先级的配置会覆盖低优先级的配置；存在不同的配置时，取并集，共同生效，形成互补配置

# Spring Boot自动配置原理

## Spring Factories 机制

Spring Boot的自动配置是基于Spring Factories机制实现的。

Spring Factories机制是Spring Boot中的一种服务发现机制，这种扩展机制与Java SPI机制十分相似。Spring Boot会自动扫描所有Jar包类路径下META-INF/spring.factories文件，并读取其中的内容，进行实例化，这种机制也是Spring Boot Starter的基础。

## Spring Factories实现原理

spring-core 包里定义了 SpringFactoriesLoader 类，这个类会扫描所有 Jar 包类路径下的 META-INF/spring.factories 文件，并获取指定接口的配置。在 SpringFactoriesLoader 类中定义了两个对外的方法，如下表。

| 返回值       | 方法                                                         | 描述                                                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| <T> List<T>  | loadFactories(Class<T> factoryType, @Nullable ClassLoader classLoader) | 静态方法； 根据接口获取其实现类的实例； 该方法返回的是实现类对象列表。 |
| List<String> | loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) | 公共静态方法； 根据接口l获取其实现类的名称； 该方法返回的是实现类的类名的列表 |

两个方法的关键都是从指定的ClassLoader中获取spring.factories文件，并解析得到类名列表。

### loadFactories()

loadFactories()方法能够获取指定接口的实现类对象，具体代码如下：

```java
public static <T> List<T> loadFactories(Class<T> factoryType, @Nullable ClassLoader classLoader) {
    Assert.notNull(factoryType, "'factoryType' must not be null");
    ClassLoader classLoaderToUse = classLoader;
    if (classLoader == null) {
        classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
    }
    // 调用loadFactoryNames获取接口的实现类
    List<String> factoryImplementationNames = loadFactoryNames(factoryType, classLoaderToUse);
    if (logger.isTraceEnabled()) {
        logger.trace("Loaded [" + factoryType.getName() + "] names: " + factoryImplementationNames);
    }
    // 遍历 factoryNames 数组，创建实现类的对象
    List<T> result = new ArrayList(factoryImplementationNames.size());
    Iterator var5 = factoryImplementationNames.iterator();
    //排序
    while(var5.hasNext()) {
        String factoryImplementationName = (String)var5.next();
        result.add(instantiateFactory(factoryImplementationName, factoryType, classLoaderToUse));
    }

    AnnotationAwareOrderComparator.sort(result);
    return result;
}
```

### loadFactoryNames()

loadFactoryNames() 方法能够根据接口获取其实现类类名的集合，具体代码如下。

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
    ClassLoader classLoaderToUse = classLoader;
    if (classLoader == null) {
        classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
    }

    String factoryTypeName = factoryType.getName();
    //获取自动配置类
    return (List)loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
}
```

### loadSpringFactories() 

loadSpringFactories() 方法能够读取该项目中所有 Jar 包类路径下 META-INF/spring.factories 文件的配置内容，并以 Map 集合的形式返回，具体代码如下。

```java
private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
    Map<String, List<String>> result = (Map)cache.get(classLoader);
    if (result != null) {
        return result;
    } else {
        HashMap result = new HashMap();

    try {
        //扫描所有 Jar 包类路径下的 META-INF/spring.factories 文件
        Enumeration urls = classLoader.getResources("META-INF/spring.factories");

        while(urls.hasMoreElements()) {
                URL url = (URL)urls.nextElement();
                UrlResource resource = new UrlResource(url);
                //将扫描到的 META-INF/spring.factories 文件中内容包装成 properties 对象
                Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                Iterator var6 = properties.entrySet().iterator();

                while(var6.hasNext()) {
                    Map.Entry<?, ?> entry = (Map.Entry)var6.next();
                    //提取 properties 对象中的 key 值
                    String factoryTypeName = ((String)entry.getKey()).trim();
                    //提取 proper 对象中的 value 值（多个类的完全限定名使用逗号连接的字符串）
                    // 使用逗号为分隔符转换为数组，数组内每个元素都是配置类的完全限定名
                    String[] factoryImplementationNames = StringUtils.commaDelimitedListToStringArray((String)entry.getValue());
                    String[] var10 = factoryImplementationNames;
                    int var11 = factoryImplementationNames.length;
                    //遍历配置类数组，并将数组转换为 list 集合
                    for(int var12 = 0; var12 < var11; ++var12) {
                        String factoryImplementationName = var10[var12];
                        ((List)result.computeIfAbsent(factoryTypeName, (key) -> {
                            return new ArrayList();
                        })).add(factoryImplementationName.trim());
                    }
                }
            }
            //将 propertise 对象的 key 与由配置类组成的 List 集合一一对应存入名为 result 的 Map 中
            result.replaceAll((factoryType, implementations) -> {
                return (List)implementations.stream().distinct().collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList));
            });
            cache.put(classLoader, result);
            //返回 result
            return result;
        } catch (IOException var14) {
            throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var14);
        }
     }
}
```

## 自动配置的加载

在 spring-boot-autoconfigure-xxx.jar 类路径下的 META-INF/spring.factories 中设置了 Spring Boot 自动配置的内容。

```
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
...
```

以上配置中，value取值是由多个xxxAutoConfiguration(使用逗号分隔)组成，每个xxxAutoConfiguration都是一个自动配置类。Spring Boot启动时，会利用Spring-Factories机制，将这些xxxAutoConfiguration实例化并作为组件加入到容器中，以实现Spring Boot的自动配置。

## @SpringBootApplication 注解

所有 Spring Boot 项目的主启动程序类上都使用了一个 @SpringBootApplication 注解，该注解是 Spring Boot 中最重要的注解之一 ，也是 Spring Boot 实现自动化配置的关键。 

@SpringBootApplication 是一个组合元注解，其主要包含两个注解：@SpringBootConfiguration 和 @EnableAutoConfiguration，其中 @EnableAutoConfiguration 注解是 SpringBoot 自动化配置的核心所在。

![@SpringBootApplication 注解](https://gitee.com/huawesome/my-picture/raw/master/img/202109282219751.png)

## @EnableAutoConfiguration 注解

@EnableAutoConfiguration 注解用于开启 Spring Boot 的自动配置功能， 它使用 Spring 框架提供的 @Import 注解通过 AutoConfigurationImportSelector类（选择器）给容器中导入自动配置组件。

![AutoConfigurationImportSelector 类](https://gitee.com/huawesome/my-picture/raw/master/img/202109282220599.png)

## AutoConfigurationImportSelector类

AutoConfigurationImportSelector 类实现了 DeferredImportSelector 接口，AutoConfigurationImportSelector 中还包含一个静态内部类 AutoConfigurationGroup，它实现了 DeferredImportSelector 接口的内部接口 Group（Spring 5 新增）。

AutoConfigurationImportSelector 类中包含 3 个方法，如下表。

| 返回值                 | 方法声明                                                     | 描述                                                         | 内部类方法 | 内部类                 |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------- | ---------------------- |
| Class<? extends Group> | getImportGroup()                                             | 该方法获取实现了 Group 接口的类，并实例化                    | 否         |                        |
| void                   | process(AnnotationMetadata annotationMetadata, DeferredImportSelector deferredImportSelector) | 该方法用于引入自动配置的集合                                 | 是         | AutoConfigurationGroup |
| Iterable<Entry>        | selectImports()                                              | 遍历自动配置类集合（Entry 类型的集合），并逐个解析集合中的配置类 | 是         | AutoConfigurationGroup |

### getImportGroup() 方法

AutoConfigurationImportSelector 类中 getImportGroup() 方法主要用于获取实现了 DeferredImportSelector.Group 接口的类，代码如下。

```java
    public Class<? extends Group> getImportGroup() {
        //获取实现了 DeferredImportSelector.Gorup 接口的 AutoConfigurationImportSelector.AutoConfigurationGroup 类
        return AutoConfigurationImportSelector.AutoConfigurationGroup.class;
    }
```

### process() 方法

静态内部类 AutoConfigurationGroup 中的核心方法是 process()，该方法通过调用 getAutoConfigurationEntry() 方法读取 spring.factories 文件中的内容，获得自动配置类的集合，代码如下 。

```java
public void process(AnnotationMetadata annotationMetadata, DeferredImportSelector deferredImportSelector) {
    Assert.state(deferredImportSelector instanceof AutoConfigurationImportSelector, () -> {
        return String.format("Only %s implementations are supported, got %s", AutoConfigurationImportSelector.class.getSimpleName(), deferredImportSelector.getClass().getName());
    });
    //拿到 META-INF/spring.factories中的EnableAutoConfiguration，并做排除、过滤处理
    //AutoConfigurationEntry里有需要引入配置类和排除掉的配置类，最终只要返回需要配置的配置类
    AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry =((AutoConfigurationImportSelector)deferredImportSelector).getAutoConfigurationEntry(annotationMetadata);
    //加入缓存,List<AutoConfigurationEntry>类型
    this.autoConfigurationEntries.add(autoConfigurationEntry);
    Iterator var4 = autoConfigurationEntry.getConfigurations().iterator();

    while(var4.hasNext()) {
        String importClassName = (String)var4.next();
        //加入缓存，Map<String, AnnotationMetadata>类型
        this.entries.putIfAbsent(importClassName, annotationMetadata);
    }
}
```

#### getAutoConfigurationEntry()

getAutoConfigurationEntry() 方法通过调用 getCandidateConfigurations() 方法来获取自动配置类的完全限定名，并在经过排除、过滤等处理后，将其缓存到成员变量中，具体代码如下。

```java
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    } else {
        //获取注解元数据中的属性设置
        AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
        //获取自动配置类
        List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
        //删除list 集合中重复的配置类
        configurations = this.removeDuplicates(configurations);
        //获取飘出导入的配置类
        Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
        //检查是否还存在排除配置类
        this.checkExcludedClasses(configurations, exclusions);
        //删除排除的配置类
        configurations.removeAll(exclusions);
        //获取过滤器，过滤配置类
        configurations = this.getConfigurationClassFilter().filter(configurations);
        //触发自动化配置导入事件
        this.fireAutoConfigurationImportEvents(configurations, exclusions);
        return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
    }
}
```

#### getCandidateConfigurations()

在 getCandidateConfigurations() 方法中，根据 Spring Factories 机制调用 SpringFactoriesLoader 的 loadFactoryNames() 方法，根据 EnableAutoConfiguration.class （自动配置接口）获取其实现类（自动配置类）的类名的集合，如下图。

![getCandidateConfigurations 方法](https://gitee.com/huawesome/my-picture/raw/master/img/202109282236432.png)

### selectImports()

以上所有方法执行完成后，AutoConfigurationImportSelector.AutoConfigurationGroup#selectImports() 会将 process() 方法处理后得到的自动配置类，进行过滤、排除，最后将所有自动配置类添加到容器中。

```java
public Iterable<DeferredImportSelector.Group.Entry> selectImports() {
    if (this.autoConfigurationEntries.isEmpty()) {
        return Collections.emptyList();
    } else {
        //获取所有需要排除的配置类
        Set<String> allExclusions = (Set)this.autoConfigurationEntries.stream().
                map(AutoConfigurationImportSelector.AutoConfigurationEntry::getExclusions).flatMap(Collection::stream).collect(Collectors.toSet());
        //获取所有经过自动化配置过滤器的配置类
        Set<String> processedConfigurations = (Set)this.autoConfigurationEntries.stream().map(AutoConfigurationImportSelector.
                AutoConfigurationEntry::getConfigurations).flatMap(Collection::stream).collect(Collectors.toCollection(LinkedHashSet::new));
        //排除过滤后配置类中需要排除的类
        processedConfigurations.removeAll(allExclusions);
        return (Iterable)this.sortAutoConfigurations(processedConfigurations,
                this.getAutoConfigurationMetadata()).stream().map((importClassName) -> {
            return new DeferredImportSelector.Group.Entry((AnnotationMetadata)this.entries.get(importClassName), importClassName);
        }).collect(Collectors.toList());
    }
}
```

## 自动配置的生效和修改

spring.factories 文件中的所有自动配置类（xxxAutoConfiguration），都是必须在一定的条件下才会作为组件添加到容器中，配置的内容才会生效。这些限制条件在 Spring Boot 中以 @Conditional 派生注解的形式体现，如下表。

| 注解                            | 生效条件                                                   |
| ------------------------------- | ---------------------------------------------------------- |
| @ConditionalOnJava              | 应用使用指定的 Java 版本时生效                             |
| @ConditionalOnBean              | 容器中存在指定的 Bean 时生效                               |
| @ConditionalOnMissingBean       | 容器中不存在指定的 Bean 时生效                             |
| @ConditionalOnExpression        | 满足指定的 SpEL 表达式时生效                               |
| @ConditionalOnClass             | 存在指定的类时生效                                         |
| @ConditionalOnMissingClass      | 不存在指定的类时生效                                       |
| @ConditionalOnSingleCandidate   | 容器中只存在一个指定的 Bean 或这个 Bean 为首选 Bean 时生效 |
| @ConditionalOnProperty          | 系统中指定属性存在指定的值时生效                           |
| @ConditionalOnResource          | 类路径下存在指定的资源文件时生效                           |
| @ConditionalOnWebApplication    | 当前应用是 web 应用时生效                                  |
| @ConditionalOnNotWebApplication | 当前应用不是 web 应用生效                                  |

## 总结（太长不看）

`@SpringBootApplication`等同于下面三个注解：

- `@SpringBootConfiguration`
- `@EnableAutoConfiguration`
- `@ComponentScan`

其中`@EnableAutoConfiguration`是关键(启用自动配置)，内部实际上就去加载`META-INF/spring.factories`文件的信息，然后筛选出以`EnableAutoConfiguration`为key的数据，加载到IOC容器中，实现自动配置功能！

[参考](https://zhuanlan.zhihu.com/p/55637237)

## 示例

### ServletWebServerFactoryAutoConfiguration

```java
@Configuration(   //表示这是一个配置类，与 xml 配置文件等价，也可以给容器中添加组件
    proxyBeanMethods = false
)
@AutoConfigureOrder(-2147483648)
@ConditionalOnClass({ServletRequest.class})//判断当前项目有没有 ServletRequest 这个类
@ConditionalOnWebApplication(// 判断当前应用是否是 web 应用，如果是，当前配置类生效 
type = Type.SERVLET
)
@EnableConfigurationProperties({ServerProperties.class})
//启动指定类的属性配置（ConfigurationProperties）功能；将配置文件中对应的值和 ServerProperties 绑定起来；并把 ServerProperties 加入到ioc容器中
@Import({ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class, EmbeddedTomcat.class, EmbeddedJetty.class, EmbeddedUndertow.class})
public class ServletWebServerFactoryAutoConfiguration {
    public ServletWebServerFactoryAutoConfiguration() {
    }

    @Bean //给容器中添加一个组件，这个组件的某些值需要从properties中获取
    public ServletWebServerFactoryCustomizer servletWebServerFactoryCustomizer(ServerProperties serverProperties, ObjectProvider<WebListenerRegistrar> webListenerRegistrars) {
        return new ServletWebServerFactoryCustomizer(serverProperties, (List) webListenerRegistrars.orderedStream().collect(Collectors.toList()));
    }

    @Bean
    @ConditionalOnClass(
            name = {"org.apache.catalina.startup.Tomcat"}
    )
    public TomcatServletWebServerFactoryCustomizer tomcatServletWebServerFactoryCustomizer(ServerProperties serverProperties) {
        return new TomcatServletWebServerFactoryCustomizer(serverProperties);
    }

    @Bean
    @ConditionalOnMissingFilterBean({ForwardedHeaderFilter.class})
    @ConditionalOnProperty(
            value = {"server.forward-headers-strategy"},
            havingValue = "framework"
    )
    public FilterRegistrationBean<ForwardedHeaderFilter> forwardedHeaderFilter() {
        ForwardedHeaderFilter filter = new ForwardedHeaderFilter();
        FilterRegistrationBean<ForwardedHeaderFilter> registration = new FilterRegistrationBean(filter, new ServletRegistrationBean[0]);
        registration.setDispatcherTypes(DispatcherType.REQUEST, new DispatcherType[]{DispatcherType.ASYNC, DispatcherType.ERROR});
        registration.setOrder(-2147483648);
        return registration;
    }

    public static class BeanPostProcessorsRegistrar implements ImportBeanDefinitionRegistrar, BeanFactoryAware {
        private ConfigurableListableBeanFactory beanFactory;

        public BeanPostProcessorsRegistrar() {
        }

        public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
            if (beanFactory instanceof ConfigurableListableBeanFactory) {
                this.beanFactory = (ConfigurableListableBeanFactory) beanFactory;
            }

        }

        public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
            if (this.beanFactory != null) {
                this.registerSyntheticBeanIfMissing(registry, "webServerFactoryCustomizerBeanPostProcessor", WebServerFactoryCustomizerBeanPostProcessor.class, WebServerFactoryCustomizerBeanPostProcessor::new);
                this.registerSyntheticBeanIfMissing(registry, "errorPageRegistrarBeanPostProcessor", ErrorPageRegistrarBeanPostProcessor.class, ErrorPageRegistrarBeanPostProcessor::new);
            }
        }

        private <T> void registerSyntheticBeanIfMissing(BeanDefinitionRegistry registry, String name, Class<T> beanClass, Supplier<T> instanceSupplier) {
            if (ObjectUtils.isEmpty(this.beanFactory.getBeanNamesForType(beanClass, true, false))) {
                RootBeanDefinition beanDefinition = new RootBeanDefinition(beanClass, instanceSupplier);
                beanDefinition.setSynthetic(true);
                registry.registerBeanDefinition(name, beanDefinition);
            }

        }
    }
}
```

该类使用了以下注解：

- @Configuration：用于定义一个配置类，可用于替换 Spring 中的 xml 配置文件；
- @Bean：被 @Configuration 注解的类内部，可以包含有一个或多个被 @Bean 注解的方法，用于构建一个 Bean，并添加到 Spring 容器中；该注解与 spring 配置文件中 <bean> 等价，方法名与 <bean> 的 id 或 name 属性等价，方法返回值与 class 属性等价；


除了 @Configuration 和 @Bean 注解外，该类还使用 5 个 @Conditional 衍生注解：

- @ConditionalOnClass({ServletRequest.class})：判断当前项目是否存在 ServletRequest 这个类，若存在，则该配置类生效。
- @ConditionalOnWebApplication(type = Type.SERVLET)：判断当前应用是否是 Web 应用，如果是的话，当前配置类生效。
- @ConditionalOnClass(name = {"org.apache.catalina.startup.Tomcat"})：判断是否存在 Tomcat 类，若存在则该方法生效。
- @ConditionalOnMissingFilterBean({ForwardedHeaderFilter.class})：判断容器中是否有 ForwardedHeaderFilter 这个过滤器，若不存在则该方法生效。
- @ConditionalOnProperty(value = {"server.forward-headers-strategy"},havingValue = "framework")：判断配置文件中是否存在 server.forward-headers-strategy = framework，若不存在则该方法生效。



### ServerProperties

ServletWebServerFactoryAutoConfiguration类还使用了一个@EnableConfigurationProperties注解，通过该注解导入了一个Serverproperties类，其部分源码如下

```java
@ConfigurationProperties(
    prefix = "server",
    ignoreUnknownFields = true
)
public class ServerProperties {
    private Integer port;
    private InetAddress address;
    @NestedConfigurationProperty
    private final ErrorProperties error = new ErrorProperties();
    private ServerProperties.ForwardHeadersStrategy forwardHeadersStrategy;
    private String serverHeader;
    private DataSize maxHttpHeaderSize = DataSize.ofKilobytes(8L);
    private Shutdown shutdown;
    @NestedConfigurationProperty
    private Ssl ssl;
    @NestedConfigurationProperty
    private final Compression compression;
    @NestedConfigurationProperty
    private final Http2 http2;
    private final ServerProperties.Servlet servlet;
    private final ServerProperties.Tomcat tomcat;
    private final ServerProperties.Jetty jetty;
    private final ServerProperties.Netty netty;
    private final ServerProperties.Undertow undertow;

    public ServerProperties() {
        this.shutdown = Shutdown.IMMEDIATE;
        this.compression = new Compression();
        this.http2 = new Http2();
        this.servlet = new ServerProperties.Servlet();
        this.tomcat = new ServerProperties.Tomcat();
        this.jetty = new ServerProperties.Jetty();
        this.netty = new ServerProperties.Netty();
        this.undertow = new ServerProperties.Undertow();
    }
    ....
}
```

我们看到，ServletWebServerFactoryAutoConfiguration 使用了一个 @EnableConfigurationProperties 注解，而 ServerProperties 类上则使用了一个 @ConfigurationProperties 注解。这其实是 Spring Boot 自动配置机制中的通用用法。

Spring Boot 中为我们提供了大量的自动配置类 XxxAutoConfiguration 以及 XxxProperties，每个自动配置类 XxxAutoConfiguration 都使用了 @EnableConfigurationProperties 注解，而每个 XxxProperties 上都使用 @ConfigurationProperties 注解。

@ConfigurationProperties 注解的作用，是将这个类的所有属性与配置文件中相关的配置进行绑定，以便于获取或修改配置，但是 @ConfigurationProperties 功能是由容器提供的，被它注解的类必须是容器中的一个组件，否则该功能就无法使用。而 @EnableConfigurationProperties 注解的作用正是将指定的类以组件的形式注入到 IOC 容器中，并开启其 @ConfigurationProperties 功能。因此，@ConfigurationProperties + @EnableConfigurationProperties 组合使用，便可以为 XxxProperties 类实现配置绑定功能。

自动配置类 XxxAutoConfiguration 负责使用 XxxProperties 中属性进行自动配置，而 XxxProperties 则负责将自动配置属性与配置文件的相关配置进行绑定，以便于用户通过配置文件修改默认的自动配置。也就是说，真正“限制”我们可以在配置文件中配置哪些属性的类就是这些 XxxxProperties 类，它与配置文件中定义的 prefix 关键字开头的一组属性是唯一对应的。

> 注意：XxxAutoConfiguration 与 XxxProperties 并不是一一对应的，大多数情况都是多对多的关系，即一个 XxxAutoConfiguration 可以同时使用多个 XxxProperties 中的属性，一个 XxxProperties 类中属性也可以被多个 XxxAutoConfiguration 使用。







