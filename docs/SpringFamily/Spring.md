### 1.Spring是什么？

​		Spring是一种轻量级开发框架，官网：http://spring.io/；而我们常说的Spring是指Spring Framework，它是很多模块的集合，如核心容器、数据访问/集成、Web、AOP、工具、消息和测试模块。比如：Core Container中的Core组件是Spring所有组件的核心，Beans组件和Context组件是实现IOC和依赖注入的基础，AOP组件用来实现面向切面编程。

​		Spring的6个特征：

- **核心技术**：依赖注入（DI），AOP，事件（events），资源，i18n，验证，数据绑定，类型转换
- **测试**：模拟对象，TestContext框架，Spring MVC测试，WebTestClient
- **数据访问**：事务，DAO支持，JDBC，ORM，编组XML
- **Web支持**：Spring MVC和Spring WebFlux Web框架
- **集成**：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存
- **语言**：Kotlin，Groovy，动态语言

### 2.IOC

​		IOC（Inspection of Control：控制反转）是一种设计思想，也被称为DI（Dependency Injection），即由Spring IOC容器来负责对象的声明周期和对象之间的关系。

​		Spring IOC的初始化过程：https://javadoop.com/post/spring-ioc

### 3.AOP

​		AOP（Aspect-Oriented Programming：面向切面编程）主要解决一些系统层面上的问题，如日志，事务，权限等，即将那些与业务无关，却为业务模块所共同调用的逻辑和责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，提高了可操作性和可维护性。

### 4.Spring Bean

（1）Spring中的bean的作用域有哪些？

- singleton：唯一bean实例，Spring中的bean默认都是单例的
- prototype：每次请求都会创建一个新的bean实例
- request：每一次Http请求都会产生一个新的bean，该bean仅在当前Http request内有效
- session：每一次Http请求都会产生一个新的bean，该bean仅在当前Http session内有效
- globalsession：全局session作用域，仅仅在基于protlet的web应用中才有意义，Spring5已经取消。

（2）Spring Bean的单例Bean的线程安全问题

​		当多个线程同时操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

​		常见的两种解决办法：

1. 在Bean对象中尽量笔迷那定义可变的成员变量（不现实）
2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在ThreadLocal中（推荐）

（3）@Component和@Bean的区别

1. 作用对象不同：@Component注解作用于类，@Bean作用于方法
2. @Component通常是通过类路径扫描来自动检测以及自动装配到Spring的Bean容器中；@Bean注解通常是我们在标有该注解的方法中定义产生这个bean，@Bean告诉了Spring这是某个类的示例。
3. @Bean注解比@Component注解的自定义性更强，有些地方只能用@Bean，比如引入第三方库的类需要装配到Spring容器时。

@Bean注解使用示例：

```
@Configuration
public class AppConfig{
	@Bean
	public TransferService transferService(){
		return new TransferServiceImpl();
	}
}
```

> 只要是被@Component修饰的类或注解都可以定义@Bean，都可以注册进Spring容器里面，如@Repository、@Service、@Controller @Configuration

### 5.Spring Bean的声明周期

实例化->属性赋值->（前置处理）->初始化->（后置处理）->销毁

https://www.jianshu.com/p/1dec08d290c1

1. **实例化Bean**：对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。 对于ApplicationContext容器，当容器启动结束后，便实例化所有的bean。 容器通过获取BeanDefinition对象中的信息进行实例化。并且这一步仅仅是简单的实例化，并未进行依赖注入。 实例化对象被包装在BeanWrapper对象中，BeanWrapper提供了设置对象属性的接口，从而避免了使用反射机制设置属性。

2. **设置对象属性（依赖注入）**：实例化后的对象被封装在BeanWrapper对象中，并且此时对象仍然是一个原生的状态，并没有进行依赖注入。紧接着，Spring根据BeanDefinition中的信息进行依赖注入。并且通过BeanWrapper提供的设置属性的接口完成依赖注入。

3. **注入Aware接口**：Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给bean。

4. **BeanPostProcessor**：经过上述几个步骤后，bean对象已经被正确构造，但如果你想要对象被使用前再进行一些自定义的处理，就可以通过BeanPostProcessor接口实现。

   该接口提供了两个函数：

   - postProcessBeforeInitialzation( Object bean, String beanName ) 当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 这个函数会先于InitialzationBean执行，因此称为**前置处理**。 所有Aware接口的注入就是在这一步完成的。
   - postProcessAfterInitialzation( Object bean, String beanName ) 
     当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
     这个函数会在InitialzationBean完成后执行，因此称为**后置处理**。

5. **InitializingBean与init-method**：当BeanPostProcessor的前置处理完成后就会进入本阶段。 InitializingBean接口只有一个函数：afterPropertiesSet()。

   ​		这一阶段也可以在bean正式构造完成前增加我们自定义的逻辑，但它与前置处理不同，由于该函数并不会把当前bean对象传进来，因此在这一步没办法处理对象本身，只能增加一些额外的逻辑。 
   若要使用它，我们需要让bean实现该接口，并把要增加的逻辑写在该函数中。然后Spring会在前置处理完成后检测当前bean是否实现了该接口，并执行afterPropertiesSet函数。

   ​		当然，Spring为了降低对客户代码的侵入性，给bean的配置提供了init-method属性，该属性指定了在这一阶段需要执行的函数名。Spring便会在初始化阶段执行我们设置的函数。init-method本质上仍然使用了InitializingBean接口。

6. **DisposableBean和destroy-method**：通过给destroy-method指定函数，就可以在bean销毁前执行指定的逻辑。

### 5.Spring事务

​		Spring事务分为编程式事务（硬编码，不推荐）和声明式事务（配置文件配置，推荐），声明式事务可用XML或注解进行配置

（1）**事务的特性**

- **原子性（Atomicity）**：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
- **一致性（Consistency）：**一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
- **隔离性（Isolation）**：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。
- **持久性（Durability）**：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。

（2）**事务传播特性**

- propagation_requierd：如果当前没有事务，就新建一个事务，如果已存在一个事务中，加入到这个事务中，这是最常见的选择。
- propagation_supports：支持当前事务，如果没有当前事务，就以非事务方法执行。
- propagation_mandatory：使用当前事务，如果没有当前事务，就抛出异常。
- propagation_required_new：新建事务，如果当前存在事务，把当前事务挂起。新建的事务将和被挂起的事务没有任何关系，是两个独立的事务，外层事务失败回滚之后，不能回滚内层事务执行的结果，内层事务失败抛出异常，外层事务捕获，也可以不处理回滚操作。
- propagation_not_supported：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- propagation_never：以非事务方式执行操作，如果当前事务存在则抛出异常。
- propagation_nested：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与propagation_required类似的操作。它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。内部事务的回滚不会对外部事务造成影响。它只对DataSourceTransactionManager事务管理器生效。

（3）**事务隔离级别**

​		TransactionDefinition接口中定义了5个表示隔离级别的常量：

- TransactionDefinition.ISOLATION_DEFAULT:使用后端数据库默认的隔离级别，Mysql默认采用的REPEATABLE_READ隔离级别，Oracle默认采用的READ_COMMITTED隔离级别
- TransactionDefinition.ISOLATION_READ_UNCOMMITTED:读未提交，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**。
- TransactionDefinition.ISOLATION_READ_COMMITTED:读已提交，允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**。
- TransactionDefinition.ISOLATION_REPEATABLE_READ:对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生**。
- TransactionDefinition.ISOLATION_SERIALIZABLE:可序列化，最高的隔离级别，完全符合ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，该级别可以**防止脏读、幻读和不可重复读**，但严重影响程序性能，通常不会使用。

（4）**@Transactional（rollbackFor=Exception.class）**

​		Exception分为运行时异常和非运行时异常，当@Transactional注解作用于类上，该类的所有**public**方法都将具有该类型的事务属性，同时，在方法加此注解可以覆盖类级别的定义。如果类或者方法上加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。

​		在@Transactional注解中如果不配置rollbackFor属性，那么事务只会在遇到Rxception的时候才会回滚，加上rollbackFor=Exception.class，可以让事务在遇到非运行时异常时也回滚。

​		@Transactional注解的属性信息

- name:当在配置文件中有多个TransactionManager，可以用该属性指定选择哪个事务管理器
- propagation：事务的传播行为，默认值未REQUIRED
- isolation:事务的隔离度，默认值采用DEFAULT
- timeout:事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务
- read-only：指定事务是否为只读事务，默认为false；为了忽略那些不需要事务的方法，比如读取事务，可以设置read-only为true
- rollback-for：用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔
- no-rollback-for：抛出no-rollback-for指定的异常类型，不回滚事务























