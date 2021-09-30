---
title: springboot之mybatis自动配置
date: 2021-09-29 15:38:06
---

了解springboot自动配置之前需要先了解mybatis的使用，以及mybatis如何集成到spring，还要了解一下工厂方法设计模式。

### 1.工厂方法
复杂对象的实例化交给工厂，消费者（使用方）只需要在工厂的中获取对象，不需要关心对象的生产过程。解决了多个类之前的强耦合（由多对多变为多对一），这样某个类的创建有修改时只需要更改工厂中的实现即可；
**开闭原则：**
* 工厂抽象为接口，这样增加一个类时只需要增加一个具体工厂即可；
* 工厂中增加配置属性，而配置可以通过文件加载，这样更改对象参数只需要修改配置文件即可；

mybatis和spring使用了大量的工厂方法设计模式。

### 2.mybatis使用
可以回想一下使用jdbc操作数据库的代码，mybatis对它做了很好的封装，使用mybatis主要是创建SqlSessionFactory这个工厂，通过在工厂中获取sqlsession操作数据库，工厂的配置是通过读取配置文件获取。
1. 创建SqlSessionFactory

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
----------java config
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
```
2. 创建mapper文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
````

3. 使用sqlsession执行sql

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
  Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
}
```
**mybatis使用代理MapperProxy操作数据库方式**
> 1、2同上

3. 创建mapper接口

```java
package org.mybatis.example;
public interface BlogMapper {
  Blog selectBlog(int id);
}
```
4. 创建代理类执行sql

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
}
```

[官方文档](https://mybatis.org/mybatis-3/zh/index.html)
### 3.spring集成mybatis-spring
集成的方案主要是将SqlSessionFactory和Mapper的创建交给spring的工厂。

1. 配置SqlSessionFactoryBean
此类实现了FactoryBean接口，意味着通过它创建的对象实际上是SqlSessionFactory

```java
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
</bean>
----------------java config
@Configuration
public class MyBatisConfig {
  @Bean
  public SqlSessionFactory sqlSessionFactory() throws Exception {
    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
    factoryBean.setDataSource(dataSource());
    return factoryBean.getObject();
  }
}
```
2. 创建mapper接口

```java
public interface UserMapper {
  @Select("SELECT * FROM users WHERE id = #{userId}")
  User getUser(@Param("userId") String userId);
}

<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
  <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
-------java config
@Configuration
public class MyBatisConfig {
  @Bean
  public UserMapper userMapper() throws Exception {
    SqlSessionTemplate sqlSessionTemplate = new SqlSessionTemplate(sqlSessionFactory());
    return sqlSessionTemplate.getMapper(UserMapper.class);
  }
}
```

3. 注入maoper操作数据库

```java
public class FooServiceImpl implements FooService {

  private final UserMapper userMapper;

  public FooServiceImpl(UserMapper userMapper) {
    this.userMapper = userMapper;
  }

  public User doSomeBusinessStuff(String userId) {
    return this.userMapper.getUser(userId);
  }
}
```

[官方文档](http://mybatis.org/spring/zh/getting-started.html)


### 4.springboot自动配置mybatis

依赖于springboot的自动配置原理，启动时会扫描META-INF/spring.factories下的文件，反射获取类信息，并完成自动配置。
打开mybatis自动配置jar包下的文件内容如下：
```xml
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration
```
找到自动配置的类MybatisAutoConfiguration，
```
@org.springframework.context.annotation.Configuration
@ConditionalOnClass({ SqlSessionFactory.class, SqlSessionFactoryBean.class })
@ConditionalOnBean(DataSource.class)
@EnableConfigurationProperties(MybatisProperties.class)
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
public class MybatisAutoConfiguration {
	...
}
```
通过注解可以看到配置的条件，以及`MybatisProperties`配置文件的信息,之前需要手动创建的sqlSessionFactory、sqlSessionTemplate都已经帮我们做好了配置，修改配置信息只需要修改配置文件即可
```
@Bean
@ConditionalOnMissingBean
public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
	...
}
@Bean
@ConditionalOnMissingBean
public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
	ExecutorType executorType = this.properties.getExecutorType();
	if (executorType != null) {
	return new SqlSessionTemplate(sqlSessionFactory, executorType);
	} else {
	return new SqlSessionTemplate(sqlSessionFactory);
	}
}
```
完成了sqlSessionFactory的创建，那么Mapper的接口是如何创建的呢？
```
@org.springframework.context.annotation.Configuration
@Import({ AutoConfiguredMapperScannerRegistrar.class })
@ConditionalOnMissingBean(MapperFactoryBean.class)
public static class MapperScannerRegistrarNotFoundConfiguration {

	@PostConstruct
	public void afterPropertiesSet() {
	logger.debug("No {} found.", MapperFactoryBean.class.getName());
	}
}
```
可见这也是个配置类，spring工厂中没有这个MapperFactoryBean实例时，会自动导入AutoConfiguredMapperScannerRegistrar进行自动配置，
```
public static class AutoConfiguredMapperScannerRegistrar
implements BeanFactoryAware, ImportBeanDefinitionRegistrar, ResourceLoaderAware {

	private BeanFactory beanFactory;

	private ResourceLoader resourceLoader;

	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

		logger.debug("Searching for mappers annotated with @Mapper");

		ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);

		try {
			if (this.resourceLoader != null) {
				scanner.setResourceLoader(this.resourceLoader);
			}

			List<String> packages = AutoConfigurationPackages.get(this.beanFactory);
			if (logger.isDebugEnabled()) {
				for (String pkg : packages) {
					logger.debug("Using auto-configuration base package '{}'", pkg);
				}
			}

			scanner.setAnnotationClass(Mapper.class);
			scanner.registerFilters();
			scanner.doScan(StringUtils.toStringArray(packages));
		} catch (IllegalStateException ex) {
			logger.debug("Could not determine auto-configuration package, automatic mapper scanning disabled.", ex);
		}
}
```
可见它会扫描BasePackages下@Mapper注解的类实现


























































