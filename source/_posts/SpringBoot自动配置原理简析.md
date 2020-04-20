---
title: SpringBoot自动配置原理简析
date: 2020-04-30 23:18:10
categories:
tags:
---

简单叙述下SpringBoot自动配置的原理，之后再回头梳理验证

<!-- more -->

# SpingBoot自动配置原理简析

## 0.自动配置简述

​	Spring Boot有默认的全局配置文件，名叫`application.properties`或`application.yml`。我们可以在这个配置文件里写一大堆属性进行配置，例如：server.port、logging.level等等。

​	除此之外还能配那些呢，可参考[Spring Boot的官方文档介绍](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-application-properties.html#web-properties)

​	闲话说完，进入正题。要探究Spring Boot的自动配置原理，就绕不开一个注解`@SpringBootApplication`，这是Spring Boot项目必不可少的注解。该注解的源码如下：

```java
@Target({ElementType.TYPE})    //可以给一个类型进行注解，比如类、接口、枚举
@Retention(RetentionPolicy.RUNTIME)    //可以保留到程序运行的时候，它会被加载进入到 JVM 中
@Documented    //将注解中的元素包含到 Javadoc 中去。
@Inherited    //继承，比如A类上有该注解，B类继承A类，B类就也拥有该注解

@SpringBootConfiguration

@EnableAutoConfiguration

/*
*创建一个配置类，在配置类上添加 @ComponentScan 注解。
*该注解默认会扫描该类所在的包下所有的配置类，相当于之前的 <context:component-scan>。
*/
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication
```

​	可以看到，除了`@Target`、`@Retention`、`@Documented `和`@Inherited`外，还有几个注解，其中的`@EnableAutoConfiguration`注解便是Spring Boot能自动配置的关键，该注解源码如下：

```java
 @Target({ElementType.TYPE})
 @Retention(RetentionPolicy.RUNTIME)
 @Documented
 @Inherited

 @AutoConfigurationPackage
 @Import({AutoConfigurationImportSelector.class})

 public @interface EnableAutoConfiguration
```

​	嗯，`@EnableAutoConfiguration`这个注解也是个复合注解，也被多个注解所标注，那就接着来看它们的作用。



​	首先`@AutoConfigurationPackage`，中文直译：自动配置包，它的相关源码如下：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited

@Import({Registrar.class})
public @interface AutoConfigurationPackage
```

​	哎呀，又套了一堆注解，但跟上面的`@AutoConfigurationPackage`所用的注解类似，除了基本的注解外，二者都有一个`@Import`。

​	`@AutoConfigurationPackage`里的这个`@Import({Registrar.class})`，
​	该注解表示org.springframework.boot.autoconfigure.AutoConfigurationPackages.Registrar，就这个类，它会将**主配置类（也就是@SpringBootApplication标注的类）的所在包**及其下所有子包里面的所有组件注册到Spring容器当中。

​	而`@EnableAutoConfiguration`里的`@Import({AutoConfigurationImportSelector.class})`，表示通过`@Import`导入了一个类`AutoConfigurationImportSelector`。

​	AutoConfigurationImportSelector类里面有这么几个方法，如下：

```java
/**
* 方法用于给容器中导入组件
**/
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
    AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
        .loadMetadata(this.beanClassLoader);
    AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(
        autoConfigurationMetadata, annotationMetadata);  // 获取自动配置项
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}

// 获取自动配置项
protected AutoConfigurationEntry getAutoConfigurationEntry(
    AutoConfigurationMetadata autoConfigurationMetadata,
    AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    }
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    List < String > configurations = getCandidateConfigurations(annotationMetadata,
        attributes);  //  获取一个自动配置 List ，这个 List 就包含了所有自动配置的类名
    configurations = removeDuplicates(configurations);
    Set < String > exclusions = getExclusions(annotationMetadata, attributes);
    checkExcludedClasses(configurations, exclusions);
    configurations.removeAll(exclusions);
    configurations = filter(configurations, autoConfigurationMetadata);
    fireAutoConfigurationImportEvents(configurations, exclusions);
    return new AutoConfigurationEntry(configurations, exclusions);
}

//   获取一个自动配置 List ，这个 List 就包含了所有的自动配置的类名
protected List < String > getCandidateConfigurations(AnnotationMetadata metadata,
    AnnotationAttributes attributes) {
    // 通过 getSpringFactoriesLoaderFactoryClass 获取默认的 EnableAutoConfiguration.class 类名，传入 loadFactoryNames 方法
    List < String > configurations = SpringFactoriesLoader.loadFactoryNames(
        getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
    Assert.notEmpty(configurations,
        "No auto configuration classes found in META-INF/spring.factories. If you " +
        "are using a custom packaging, make sure that file is correct.");
    return configurations;
}

// 默认的 EnableAutoConfiguration.class 类名
protected Class<?> getSpringFactoriesLoaderFactoryClass() {
    return EnableAutoConfiguration.class;
}
```

步骤简述：

1. 通过方法**selectImports**给容器导入了组件。往里深入就能发现，该方法的方法体中调用了同类的第二个方法getAutoConfigurationEntry (获取自动配置项)。

2. 方法**getAutoConfigurationEntry**，是用来获取自动配置项的。往里走，发现该方法调用了上述的第三个方法getCandidateConfigurations，进而拿到了一个List，这个List里有所有自动配置类的类名，

3. 方法**getCandidateConfigurations**，用来获取自动配置的List的，该List包含了所有自动配置类的类名。而该方法中的List是通过调用**SpringFactoriesLoader类**的`loadFactoryNames()`方法得到的。

4. `loadFactoryNames()`方法源码如下：

   ```java
   public static List < String > loadFactoryNames(Class < ? > factoryClass, @Nullable ClassLoader classLoader) {
       String factoryClassName = factoryClass.getName();
       return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
   }

   private static Map < String, List < String >> loadSpringFactories(@Nullable ClassLoader classLoader) {
       MultiValueMap < String, String > result = cache.get(classLoader);
       if (result != null) {
           return result;
       }

       try {
           // 扫描所有 jar 包类路径下  META-INF/spring.factories
           Enumeration < URL > urls = (classLoader != null ?
               classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
               ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
           result = new LinkedMultiValueMap < > ();
           while (urls.hasMoreElements()) {
               URL url = urls.nextElement();
               UrlResource resource = new UrlResource(url);
               // 把扫描到的这些文件的内容包装成 properties 对象
               Properties properties = PropertiesLoaderUtils.loadProperties(resource);
               for (Map.Entry < ? , ? > entry : properties.entrySet()) {
                   String factoryClassName = ((String) entry.getKey()).trim();
                   for (String factoryName: StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
                       // 从 properties 中获取到 EnableAutoConfiguration.class 类（类名）对应的值，然后把他们添加在容器中
                       result.add(factoryClassName, factoryName.trim());
                   }
               }
           }
           cache.put(classLoader, result);
           return result;
       } catch (IOException ex) {
           throw new IllegalArgumentException("Unable to load factories from location [" +
               FACTORIES_RESOURCE_LOCATION + "]", ex);
       }
   }
   ```

   ​	然后经过上述巴拉巴拉一系列操作，简单说就是利用PropertiesLoaderUtils 把 ClassLoader 扫描到的这些文件的内容包装成 properties 对象，然后从 properties 中获取到 EnableAutoConfiguration.class 类（类名）对应的值，最后把他们添加在容器中。

   而上述源码中的FACTORIES_RESOURCE_LOCATION是啥？

   ```java
   public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
   ```

   可以看到，它指明了一个文件位置，META-INF/spring.factories。而之前的操作就是将类路径下的spring.factories文件里面配置的所有 EnableAutoConfiguration 的值加入到了容器中。

   ​	所有的 EnableAutoConfiguration 如下所示：注意到 EnableAutoConfiguration 有一个 = 号，= 号后面那一串就是这个项目需要用到的自动配置类。

   ```properties
   # Auto Configure
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
   org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
   org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
   org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
   org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
   org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
   org.springframework.boot.autoconfigure.cloud.CloudServiceConnectorsAutoConfiguration,\
   org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
   org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
   org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
   org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
   org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
   org.springframework.boot.autoconfigure.elasticsearch.rest.RestClientAutoConfiguration,\
   org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
   org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
   org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
   org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
   org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
   org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
   org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
   org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
   org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
   org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
   org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
   org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
   org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
   org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
   org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
   org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
   org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
   org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
   org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
   org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
   org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
   org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
   org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
   org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
   org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
   org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
   org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
   org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
   org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
   org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
   org.springframework.boot.autoconfigure.reactor.core.ReactorCoreAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.servlet.SecurityRequestMatcherProviderAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
   org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
   org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
   org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
   org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
   org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
   org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
   org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
   org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
   org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
   org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
   org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
   org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
   org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
   org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
   ```

   上面这一百多个**自动配置类**就加到了容器中了，至于生不生效，满足什么条件才会自动进行配置呢？

   ## 1. 自动配置生效条件

   每一个XxxxAutoConfiguration自动配置类都是**在某些条件之下才会生效**的。这些条件限制在Spring Boot中以注解的形式体现，常见的**条件注解**有如下几项：

   ```java
   @ConditionalOnBean// 当容器里有指定的bean的条件下。

   @ConditionalOnMissingBean// 当容器里不存在指定bean的条件下。

   @ConditionalOnClass// 当类路径下有指定类的条件下。

   @ConditionalOnMissingClass// 当类路径下不存在指定类的条件下。

   @ConditionalOnProperty// 指定的属性是否有指定的值，比如@ConditionalOnProperties(prefix=”xxx.xxx”, value=”enable”, matchIfMissing=true),代表当xxx.xxx为enable时条件的布尔值为true，如果没有设置的情况下也为true。
   ```

   以**HttpEncodingAutoConfiguration**为例解释自动配置原理，源码如下：

   ```java
   @Configuration   //表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
   @EnableConfigurationProperties(HttpEncodingProperties.class)  //启动指定类的ConfigurationProperties功能；将配置文件中对应的值和HttpEncodingProperties绑定起来；并把HttpEncodingProperties加入到ioc容器中

   @ConditionalOnWebApplication //Spring底层@Conditional注解（Spring注解版），根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；    判断当前应用是否是web应用，如果是，当前配置类生效

   @ConditionalOnClass(CharacterEncodingFilter.class)  //判断当前项目有没有这个类CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；

   @ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)  //判断配置文件中是否存在某个配置  spring.http.encoding.enabled；如果不存在，判断也是成立的
   //即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
   public class HttpEncodingAutoConfiguration {
     
     	private final HttpEncodingProperties properties;
     
      //只有一个有参构造器的情况下，参数的值就会从容器中拿
     	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
   		this.properties = properties;
   	}
     
       @Bean   //给容器中添加一个组件，这个组件的某些值需要从properties中获取
   	@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？
   	public CharacterEncodingFilter characterEncodingFilter() {
   		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
   		filter.setEncoding(this.properties.getCharset().name());
   		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
   		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
   		return filter;
   	}
   ```

   HttpEncodingAutoConfiguration类上有个**@EnableConfigurationProperties**注解，参数为HttpEncodingProperties.class，而HttpEncodingProperties的一部分源码如下：

   ```java
   @ConfigurationProperties(prefix = "spring.http.encoding")  //从配置文件中获取指定的值和bean的属性进行绑定
   public class HttpEncodingProperties {

      public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
   ```

   ​	该类上有个**@ConfigurationProperties**注解，该注解的作用就是**从配置文件中绑定属性到bean上**，，而**@EnableConfigurationProperties**注解的作用就是将绑定了属性的bean注入到Spring容器中。

   ​	诸多的XxxxAutoConfiguration自动配置类，就是Spring容器的JavaConfig形式，作用就是为Spring 容器导入bean，而所有导入的bean所需要的属性都通过xxxxProperties的bean来获得。

   ### ​简单总结

   - Spring Boot启动的时候会通过**@EnableAutoConfiguration注解**找到META-INF/spring.factories配置文件中的所有自动配置类，并对其进行加载。
   - 而这些自动配置类都是以AutoConfiguration结尾来命名的，它实际上就是一个JavaConfig形式的Spring容器配置类，它能通过以Properties结尾命名的类中取得在全局配置文件中配置的属性如：server.port，而XxxxProperties类是通过**@ConfigurationProperties**注解与全局配置文件中对应的属性进行绑定的。

   ​

   ​



