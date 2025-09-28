# Spring框架的设计哲学与源码解析

## 引言

Spring框架是Java企业级开发的事实标准，它不仅仅是一个框架，更是一套完整的企业级应用开发解决方案。从最初的IoC容器到如今的微服务生态，Spring框架体现了"简化Java开发"的核心哲学。在这篇文章中，我将深入剖析Spring框架的设计哲学、核心实现原理以及源码解析。

## 1. Spring框架概述与设计哲学

### 1.1 Spring框架的核心价值

```java
// Spring框架的核心价值体现
public class SpringCoreValues {
    // 1. 简化Java开发
    // 2. 提高开发效率
    // 3. 降低开发复杂度
    // 4. 提供企业级解决方案

    // 传统Java EE开发
    public void traditionalJavaEE() {
        // 复杂的配置和样板代码
        Context context = new InitialContext();
        DataSource dataSource = (DataSource) context.lookup("java:comp/env/jdbc/DataSource");
        UserTransaction transaction = (UserTransaction) context.lookup("java:comp/UserTransaction");

        try {
            transaction.begin();
            // 业务逻辑
            transaction.commit();
        } catch (Exception e) {
            transaction.rollback();
        }
    }

    // Spring方式开发
    @Service
    public class UserService {
        @Autowired
        private DataSource dataSource;

        @Transactional
        public void createUser(User user) {
            // 简洁的业务逻辑
            // 事务管理由AOP处理
        }
    }

    // Spring的核心优势：
    // 1. 控制反转（IoC）
    // 2. 面向切面编程（AOP）
    // 3. 声明式事务
    // 4. 简化的配置
    // 5. 丰富的集成能力
}
```

### 1.2 Spring的设计哲学

```java
// Spring框架的设计原则
public class SpringDesignPhilosophy {
    // 1. 轻量级和最小侵入性
    @Component
    public class LightweightComponent {
        // Spring对业务代码的侵入性极小
        // 只需要简单的注解即可
    }

    // 2. 松耦合
    @Service
    public class OrderService {
        // 通过依赖注入实现松耦合
        private final PaymentService paymentService;
        private final InventoryService inventoryService;

        @Autowired
        public OrderService(PaymentService paymentService, InventoryService inventoryService) {
            this.paymentService = paymentService;
            this.inventoryService = inventoryService;
        }
    }

    // 3. 面向接口编程
    public interface UserRepository {
        User findById(Long id);
        void save(User user);
    }

    @Repository
    public class JpaUserRepository implements UserRepository {
        @Override
        public User findById(Long id) {
            // JPA实现
            return null;
        }

        @Override
        public void save(User user) {
            // JPA实现
        }
    }

    // 4. 可测试性
    @Service
    public class TestableService {
        private final Dependency dependency;

        @Autowired
        public TestableService(Dependency dependency) {
            this.dependency = dependency;
        }

        // 依赖注入使得单元测试变得简单
        public void businessMethod() {
            dependency.execute();
        }
    }

    // 5. 声明式编程
    @Service
    @Transactional
    @Cacheable("userCache")
    public class DeclarativeService {
        // 使用注解声明事务和缓存
        // 而不是编写模板代码
    }
}
```

## 2. IoC容器深度解析

### 2.1 BeanFactory和ApplicationContext

```java
import org.springframework.beans.factory.*;
import org.springframework.context.*;
import org.springframework.core.io.*;

public class IoCContainerDeepDive {
    // BeanFactory - 基础IoC容器
    public void beanFactoryExample() {
        // 基础的BeanFactory使用
        Resource resource = new ClassPathResource("applicationContext.xml");
        BeanFactory beanFactory = new XmlBeanFactory(resource);

        // 获取Bean
        MyBean myBean = (MyBean) beanFactory.getBean("myBean");
        myBean.execute();
    }

    // ApplicationContext - 高级IoC容器
    public void applicationContextExample() {
        // ApplicationContext提供更多企业级功能
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        // 获取Bean
        MyBean myBean = context.getBean(MyBean.class);
        myBean.execute();

        // ApplicationContext的额外功能：
        // 1. 国际化（i18n）
        String message = context.getMessage("welcome", null, Locale.getDefault());

        // 2. 事件发布
        context.publishEvent(new MyEvent("Hello"));

        // 3. 资源加载
        Resource resource = context.getResource("classpath:config.properties");

        // 4. 环境抽象
        String property = context.getEnvironment().getProperty("app.name");
    }

    // BeanFactory vs ApplicationContext
    public void containerComparison() {
        /*
        BeanFactory:
        - 基础的IoC容器
        - 懒加载（延迟初始化）
        - 提供基本的依赖注入功能
        - 较少的内存占用

        ApplicationContext:
        - 高级的IoC容器
        - 立即加载（默认）
        - 提供企业级功能
        - 更多的内存占用
        */

        // BeanFactory - 适合资源受限的环境
        BeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("simple-context.xml"));

        // ApplicationContext - 适合企业级应用
        ApplicationContext appContext = new ClassPathXmlApplicationContext("enterprise-context.xml");
    }
}
```

### 2.2 Bean的生命周期

```java
import org.springframework.beans.factory.*;
import org.springframework.context.*;
import org.springframework.beans.factory.support.*;

public class BeanLifecycle {
    // 自定义Bean的生命周期回调
    @Component
    public class LifecycleBean implements InitializingBean, DisposableBean {
        private String name;

        // 1. 构造器实例化
        public LifecycleBean() {
            System.out.println("1. Bean实例化");
        }

        // 2. 属性注入
        @Autowired
        public void setDependency(Dependency dependency) {
            System.out.println("2. 依赖注入");
        }

        // 3. BeanNameAware接口
        @Override
        public void setBeanName(String name) {
            System.out.println("3. BeanNameAware: " + name);
        }

        // 4. BeanFactoryAware接口
        @Override
        public void setBeanFactory(BeanFactory beanFactory) {
            System.out.println("4. BeanFactoryAware");
        }

        // 5. ApplicationContextAware接口
        @Override
        public void setApplicationContext(ApplicationContext applicationContext) {
            System.out.println("5. ApplicationContextAware");
        }

        // 6. @PostConstruct注解
        @PostConstruct
        public void postConstruct() {
            System.out.println("6. @PostConstruct");
        }

        // 7. InitializingBean接口
        @Override
        public void afterPropertiesSet() {
            System.out.println("7. InitializingBean");
        }

        // 8. 自定义init-method
        public void customInit() {
            System.out.println("8. 自定义init-method");
        }

        // 9. Bean就绪，可以使用了
        public void execute() {
            System.out.println("9. Bean就绪");
        }

        // 10. @PreDestroy注解
        @PreDestroy
        public void preDestroy() {
            System.out.println("10. @PreDestroy");
        }

        // 11. DisposableBean接口
        @Override
        public void destroy() {
            System.out.println("11. DisposableBean");
        }

        // 12. 自定义destroy-method
        public void customDestroy() {
            System.out.println("12. 自定义destroy-method");
        }
    }

    // Bean生命周期配置
    @Configuration
    public class LifecycleConfig {
        @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
        public LifecycleBean lifecycleBean() {
            return new LifecycleBean();
        }
    }

    // BeanPostProcessor接口
    @Component
    public class CustomBeanPostProcessor implements BeanPostProcessor {
        @Override
        public Object postProcessBeforeInitialization(Object bean, String beanName) {
            System.out.println("BeanPostProcessor: " + beanName + " 初始化前");
            return bean;
        }

        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) {
            System.out.println("BeanPostProcessor: " + beanName + " 初始化后");
            return bean;
        }
    }

    // BeanFactoryPostProcessor接口
    @Component
    public class CustomBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
        @Override
        public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
            System.out.println("BeanFactoryPostProcessor: BeanFactory初始化完成");
        }
    }
}
```

### 2.3 依赖注入机制

```java
import org.springframework.beans.factory.annotation.*;
import org.springframework.context.annotation.*;
import org.springframework.stereotype.*;

public class DependencyInjectionMechanism {
    // 1. 构造器注入（推荐）
    @Service
    public class ConstructorInjectionService {
        private final Dependency dependency1;
        private final AnotherDependency dependency2;

        @Autowired
        public ConstructorInjectionService(Dependency dependency1, AnotherDependency dependency2) {
            this.dependency1 = dependency1;
            this.dependency2 = dependency2;
        }

        // 构造器注入的优势：
        // - 不可变性
        // - 强制依赖
        // - 易于测试
        // - 避免循环依赖
    }

    // 2. Setter注入
    @Service
    public class SetterInjectionService {
        private Dependency dependency;

        @Autowired
        public void setDependency(Dependency dependency) {
            this.dependency = dependency;
        }

        // Setter注入的优势：
        // - 可选依赖
        // - 灵活性
        // - 易于重新注入
    }

    // 3. 字段注入（不推荐）
    @Service
    public class FieldInjectionService {
        @Autowired
        private Dependency dependency;

        // 字段注入的劣势：
        // - 隐藏依赖
        // - 难以测试
        // - 违反单一职责原则
    }

    // 4. 方法注入
    @Service
    public class MethodInjectionService {
        private Dependency dependency;

        @Autowired
        public void setup(Dependency dependency, AnotherDependency another) {
            this.dependency = dependency;
            // 可以执行其他初始化逻辑
        }
    }

    // 5. @Qualifier注解
    @Service
    public class QualifierExample {
        @Autowired
        @Qualifier("primaryDataSource")
        private DataSource dataSource;

        @Autowired
        @Qualifier("secondaryDataSource")
        private DataSource secondaryDataSource;
    }

    // 6. @Primary注解
    @Configuration
    public class DataSourceConfig {
        @Bean
        @Primary
        public DataSource primaryDataSource() {
            return createDataSource("primary");
        }

        @Bean
        public DataSource secondaryDataSource() {
            return createDataSource("secondary");
        }

        private DataSource createDataSource(String name) {
            // 创建数据源
            return null;
        }
    }

    // 7. @Resource注解
    @Service
    public class ResourceExample {
        @Resource(name = "myBean")
        private MyBean myBean;

        // @Resource是JSR-250标准，默认按名称注入
    }

    // 8. @Value注解
    @Service
    public class ValueExample {
        @Value("${app.name}")
        private String appName;

        @Value("#{systemProperties['os.name']}")
        private String osName;

        @Value("#{T(Math).random() * 100}")
        private double randomNumber;
    }

    // 9. 自定义注解注入
    @Target({ElementType.FIELD, ElementType.PARAMETER})
    @Retention(RetentionPolicy.RUNTIME)
    @Autowired
    public @interface MyInject {
        String value() default "";
    }

    @Service
    public class CustomAnnotationExample {
        @MyInject("myService")
        private MyService myService;
    }

    // 10. 循环依赖处理
    @Service
    public class ServiceA {
        @Autowired
        private ServiceB serviceB;

        public void executeA() {
            System.out.println("ServiceA executing");
        }
    }

    @Service
    public class ServiceB {
        @Autowired
        private ServiceA serviceA;

        public void executeB() {
            System.out.println("ServiceB executing");
        }
    }

    // Spring如何解决循环依赖：
    // 1. 使用三级缓存
    // 2. 提前暴露未完全初始化的Bean
    // 3. 只能解决setter注入的循环依赖
    // 4. 构造器注入的循环依赖无法解决
}
```

## 3. AOP深度解析

### 3.1 AOP核心概念

```java
import org.aspectj.lang.*;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.*;

// AOP核心概念演示
@Aspect
@Component
public class AOPConcepts {
    // 1. 切面（Aspect）
    // 横切关注点的模块化
    // 这个类本身就是一个切面

    // 2. 连接点（Join Point）
    // 程序执行过程中的特定点
    // 如方法调用、异常抛出等

    // 3. 切入点（Pointcut）
    // 匹配连接点的表达式
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceLayer() {}

    @Pointcut("within(@org.springframework.stereotype.Service)")
    public void serviceClass() {}

    @Pointcut("@annotation(org.springframework.transaction.annotation.Transactional)")
    public void transactionalMethod() {}

    // 4. 通知（Advice）
    // 在特定连接点执行的动作

    // 前置通知（Before Advice）
    @Before("serviceLayer()")
    public void beforeAdvice(JoinPoint joinPoint) {
        System.out.println("Before: " + joinPoint.getSignature().getName());
        // 可以修改参数
        Object[] args = joinPoint.getArgs();
        // 参数预处理
    }

    // 后置通知（After Advice）
    @After("serviceLayer()")
    public void afterAdvice(JoinPoint joinPoint) {
        System.out.println("After: " + joinPoint.getSignature().getName());
        // 清理资源
    }

    // 返回通知（After Returning Advice）
    @AfterReturning(pointcut = "serviceLayer()", returning = "result")
    public void afterReturningAdvice(JoinPoint joinPoint, Object result) {
        System.out.println("AfterReturning: " + joinPoint.getSignature().getName() + " returned: " + result);
        // 可以修改返回值
    }

    // 异常通知（After Throwing Advice）
    @AfterThrowing(pointcut = "serviceLayer()", throwing = "ex")
    public void afterThrowingAdvice(JoinPoint joinPoint, Exception ex) {
        System.out.println("AfterThrowing: " + joinPoint.getSignature().getName() + " threw: " + ex.getMessage());
        // 异常处理逻辑
    }

    // 环绕通知（Around Advice）
    @Around("serviceLayer()")
    public Object aroundAdvice(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("Around Before: " + joinPoint.getSignature().getName());

        try {
            // 执行目标方法
            Object result = joinPoint.proceed();

            System.out.println("Around After: " + joinPoint.getSignature().getName());
            return result;
        } catch (Exception e) {
            System.out.println("Around Exception: " + e.getMessage());
            throw e;
        }
    }

    // 5. 引入（Introduction）
    // 为现有类添加新的接口或方法
    @DeclareParents(value = "com.example.service.*+", defaultImpl = DefaultTracing.class)
    public static Tracing tracing;

    // 6. 目标对象（Target Object）
    // 被一个或多个切面通知的对象
    // 即被增强的业务对象

    // 7. 代理对象（Proxy）
    // AOP框架创建的对象，用于实现切面功能
}

// 被增强的业务类
@Service
public class BusinessService {
    public void performOperation() {
        System.out.println("BusinessService: Performing operation");
    }

    public String processData(String input) {
        System.out.println("BusinessService: Processing " + input);
        return "Processed: " + input;
    }

    public void riskyOperation() throws Exception {
        System.out.println("BusinessService: Risky operation");
        throw new Exception("Something went wrong");
    }
}

// 引入的接口
public interface Tracing {
    void startTracing();
    void stopTracing();
}

// 引入的实现
@Component
public class DefaultTracing implements Tracing {
    @Override
    public void startTracing() {
        System.out.println("Tracing started");
    }

    @Override
    public void stopTracing() {
        System.out.println("Tracing stopped");
    }
}
```

### 3.2 AOP实现原理

```java
import org.springframework.aop.*;
import org.springframework.aop.framework.*;
import org.springframework.aop.support.*;

public class AOPImplementation {
    // 1. 代理模式实现AOP
    public class ProxyBasedAOP {
        public static void main(String[] args) {
            // 创建目标对象
            TargetService target = new TargetServiceImpl();

            // 创建代理对象
            TargetService proxy = (TargetService) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new LoggingInvocationHandler(target)
            );

            // 使用代理对象
            proxy.execute();
        }
    }

    // 目标接口
    public interface TargetService {
        void execute();
    }

    // 目标实现
    public static class TargetServiceImpl implements TargetService {
        @Override
        public void execute() {
            System.out.println("TargetServiceImpl: 执行业务逻辑");
        }
    }

    // 自定义InvocationHandler
    public static class LoggingInvocationHandler implements InvocationHandler {
        private final Object target;

        public LoggingInvocationHandler(Object target) {
            this.target = target;
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            // 前置处理
            System.out.println("LoggingInvocationHandler: 方法调用前 - " + method.getName());

            try {
                // 调用目标方法
                Object result = method.invoke(target, args);

                // 后置处理
                System.out.println("LoggingInvocationHandler: 方法调用后 - " + method.getName());

                return result;
            } catch (Exception e) {
                // 异常处理
                System.out.println("LoggingInvocationHandler: 方法调用异常 - " + e.getMessage());
                throw e;
            }
        }
    }

    // 2. Spring AOP的底层实现
    public class SpringAOPImplementation {
        // Spring AOP使用动态代理或CGLIB
        public static void demonstrateSpringAOP() {
            // 如果目标对象实现了接口，使用JDK动态代理
            // 如果目标对象没有实现接口，使用CGLIB

            // JDK动态代理示例
            ServiceInterface service = new ServiceImpl();
            ServiceInterface proxy = createJdkProxy(service);

            // CGLIB代理示例
            ServiceImpl target = new ServiceImpl();
            ServiceImpl cglibProxy = createCglibProxy(target);
        }

        private static ServiceInterface createJdkProxy(ServiceInterface target) {
            return (ServiceInterface) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new MethodInterceptor() {
                    @Override
                    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
                        System.out.println("JDK Proxy: Before " + method.getName());
                        Object result = proxy.invokeSuper(obj, args);
                        System.out.println("JDK Proxy: After " + method.getName());
                        return result;
                    }
                }
            );
        }

        private static ServiceImpl createCglibProxy(ServiceImpl target) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(ServiceImpl.class);
            enhancer.setCallback(new MethodInterceptor() {
                @Override
                public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
                    System.out.println("CGLIB Proxy: Before " + method.getName());
                    Object result = proxy.invoke(obj, args);
                    System.out.println("CGLIB Proxy: After " + method.getName());
                    return result;
                }
            });
            return (ServiceImpl) enhancer.create();
        }
    }

    // 3. 切面织入时机
    public class AspectWeaving {
        // 编译期织入
        // 使用AspectJ编译器，在编译时织入切面
        // 需要特殊的编译器支持

        // 编译后织入
        // 编译后对class文件进行增强
        // 使用aspectjweaver工具

        // 加载时织入
        // 类加载时织入切面
        // 使用Spring的LoadTimeWeaving

        // 运行时织入
        // 运行时动态创建代理
        // Spring AOP的主要方式
    }

    // 4. AOP性能考虑
    public class AOPPerformance {
        // 1. 减少切面数量
        // 合并相关的切面功能

        // 2. 优化切入点表达式
        // 使用具体的类名和方法名
        // 避免过于宽泛的匹配

        // 3. 合理使用通知类型
        // 前置通知和后置通知性能较好
        // 环绕通知性能较差

        // 4. 避免在切面中进行复杂计算
        // 切面应该是轻量级的
    }
}
```

## 4. 事务管理深度解析

### 4.1 声明式事务

```java
import org.springframework.transaction.annotation.*;
import org.springframework.transaction.support.*;
import org.springframework.jdbc.core.*;
import javax.sql.*;

// 声明式事务
@Service
@Transactional
public class DeclarativeTransactionService {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    // 方法级别的事务配置
    @Transactional(propagation = Propagation.REQUIRED)
    public void createUser(User user) {
        // 插入用户
        jdbcTemplate.update("INSERT INTO users (name, email) VALUES (?, ?)",
                          user.getName(), user.getEmail());

        // 插入用户详情
        jdbcTemplate.update("INSERT INTO user_details (user_id, phone) VALUES (?, ?)",
                          user.getId(), user.getPhone());
    }

    // 自定义事务属性
    @Transactional(
        propagation = Propagation.REQUIRED,
        isolation = Isolation.READ_COMMITTED,
        timeout = 30,
        readOnly = false,
        rollbackFor = {BusinessException.class},
        noRollbackFor = {SystemException.class}
    )
    public void updateUserInfo(User user) throws BusinessException, SystemException {
        jdbcTemplate.update("UPDATE users SET name = ?, email = ? WHERE id = ?",
                          user.getName(), user.getEmail(), user.getId());

        // 如果抛出BusinessException，会回滚
        // 如果抛出SystemException，不会回滚
    }

    // 只读事务
    @Transactional(readOnly = true)
    public User getUserById(Long id) {
        return jdbcTemplate.queryForObject(
            "SELECT * FROM users WHERE id = ?",
            new Object[]{id},
            (rs, rowNum) -> new User(
                rs.getLong("id"),
                rs.getString("name"),
                rs.getString("email")
            )
        );
    }

    // 嵌套事务
    @Transactional(propagation = Propagation.REQUIRED)
    public void outerMethod() {
        // 外部事务开始
        jdbcTemplate.update("INSERT INTO logs (message) VALUES ('outer start')");

        innerMethod();  // 内部方法

        jdbcTemplate.update("INSERT INTO logs (message) VALUES ('outer end')");
        // 外部事务提交
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void innerMethod() {
        // 新事务开始
        jdbcTemplate.update("INSERT INTO logs (message) VALUES ('inner')");

        // 内部事务提交
    }

    // 编程式事务（不推荐，但有时需要）
    @Autowired
    private PlatformTransactionManager transactionManager;

    public void programmaticTransaction() {
        TransactionDefinition definition = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(definition);

        try {
            // 执行业务逻辑
            jdbcTemplate.update("INSERT INTO logs (message) VALUES ('programmatic')");

            transactionManager.commit(status);
        } catch (Exception e) {
            transactionManager.rollback(status);
            throw e;
        }
    }
}

// 事务传播行为
@Transactional(propagation = Propagation.REQUIRED)
public void requiredMethod() {
    // 如果当前没有事务，就新建一个事务
    // 如果当前有事务，就加入当前事务
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void requiresNewMethod() {
    // 总是新建事务
    // 如果当前有事务，挂起当前事务
}

@Transactional(propagation = Propagation.SUPPORTS)
public void supportsMethod() {
    // 如果当前有事务，就加入当前事务
    // 如果当前没有事务，就以非事务方式执行
}

@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void notSupportedMethod() {
    // 总是以非事务方式执行
    // 如果当前有事务，挂起当前事务
}

@Transactional(propagation = Propagation.MANDATORY)
public void mandatoryMethod() {
    // 必须在事务中执行
    // 如果当前没有事务，抛出异常
}

@Transactional(propagation = Propagation.NEVER)
public void neverMethod() {
    // 必须在非事务中执行
    // 如果当前有事务，抛出异常
}

@Transactional(propagation = Propagation.NESTED)
public void nestedMethod() {
    // 如果当前有事务，就创建一个嵌套事务
    // 如果当前没有事务，就新建一个事务
}

// 事务隔离级别
@Transactional(isolation = Isolation.DEFAULT)
public void defaultIsolation() {
    // 使用数据库默认隔离级别
}

@Transactional(isolation = Isolation.READ_UNCOMMITTED)
public void readUncommitted() {
    // 读未提交（脏读、不可重复读、幻读都可能发生）
}

@Transactional(isolation = Isolation.READ_COMMITTED)
public void readCommitted() {
    // 读已提交（避免脏读，不可重复读、幻读可能发生）
}

@Transactional(isolation = Isolation.REPEATABLE_READ)
public void repeatableRead() {
    // 可重复读（避免脏读、不可重复读，幻读可能发生）
}

@Transactional(isolation = Isolation.SERIALIZABLE)
public void serializable() {
    // 串行化（避免脏读、不可重复读、幻读）
}

// 异常回滚配置
@Transactional(rollbackFor = {BusinessException.class})
public void rollbackForBusinessException() {
    // 抛出BusinessException时回滚
}

@Transactional(noRollbackFor = {BusinessException.class})
public void noRollbackForBusinessException() {
    // 抛出BusinessException时不回滚
}

@Transactional(rollbackForClassName = {"com.example.BusinessException"})
public void rollbackForClassName() {
    // 通过类名指定回滚异常
}
```

### 4.2 事务管理器

```java
import org.springframework.transaction.*;
import org.springframework.jdbc.datasource.*;
import javax.sql.*;

// 事务管理器配置
@Configuration
public class TransactionManagerConfig {
    // 1. DataSourceTransactionManager
    @Bean
    public PlatformTransactionManager dataSourceTransactionManager(DataSource dataSource) {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        transactionManager.setDefaultTimeout(30);
        transactionManager.setValidateExistingTransaction(true);
        return transactionManager;
    }

    // 2. JpaTransactionManager
    @Bean
    public PlatformTransactionManager jpaTransactionManager(EntityManagerFactory entityManagerFactory) {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(entityManagerFactory);
        transactionManager.setDataSource(dataSource());
        return transactionManager;
    }

    // 3. JtaTransactionManager
    @Bean
    public PlatformTransactionManager jtaTransactionManager() {
        JtaTransactionManager transactionManager = new JtaTransactionManager();
        // 配置JTA属性
        return transactionManager;
    }

    @Bean
    public DataSource dataSource() {
        // 配置数据源
        return new DriverManagerDataSource();
    }
}

// 自定义事务管理器
public class CustomTransactionManager extends AbstractPlatformTransactionManager {
    @Override
    protected Object doGetTransaction() {
        // 获取事务对象
        return new CustomTransactionObject();
    }

    @Override
    protected void doBegin(Object transaction, TransactionDefinition definition) {
        // 开始事务
        CustomTransactionObject tx = (CustomTransactionObject) transaction;
        tx.begin();
    }

    @Override
    protected void doCommit(DefaultTransactionStatus status) {
        // 提交事务
        CustomTransactionObject tx = (CustomTransactionObject) status.getTransaction();
        tx.commit();
    }

    @Override
    protected void doRollback(DefaultTransactionStatus status) {
        // 回滚事务
        CustomTransactionObject tx = (CustomTransactionObject) status.getTransaction();
        tx.rollback();
    }

    @Override
    protected boolean isExistingTransaction(Object transaction) {
        // 检查是否已存在事务
        return ((CustomTransactionObject) transaction).isActive();
    }

    // 自定义事务对象
    private static class CustomTransactionObject {
        private boolean active = false;

        public void begin() {
            this.active = true;
        }

        public void commit() {
            this.active = false;
        }

        public void rollback() {
            this.active = false;
        }

        public boolean isActive() {
            return active;
        }
    }
}

// 事务同步管理
@Component
public class TransactionSynchronizationManager {
    @Autowired
    private PlatformTransactionManager transactionManager;

    public void executeWithTransaction(Runnable task) {
        TransactionTemplate template = new TransactionTemplate(transactionManager);

        template.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus status) {
                task.run();
            }
        });
    }

    public <T> T executeWithTransactionResult(Callable<T> task) {
        TransactionTemplate template = new TransactionTemplate(transactionManager);

        return template.execute(new TransactionCallback<T>() {
            @Override
            public T doInTransaction(TransactionStatus status) {
                try {
                    return task.call();
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            }
        });
    }
}
```

## 5. Spring MVC深度解析

### 5.1 MVC架构与工作流程

```java
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.*;
import javax.servlet.http.*;

// Spring MVC工作流程
@Controller
@RequestMapping("/users")
public class UserController {
    // 1. 用户请求发送到DispatcherServlet
    // 2. DispatcherServlet查找HandlerMapping
    // 3. HandlerMapping找到对应的Controller
    // 4. DispatcherServlet调用Controller
    // 5. Controller处理请求并返回ModelAndView
    // 6. DispatcherServlet查找ViewResolver
    // 7. ViewResolver解析视图
    // 8. 返回响应给用户

    @Autowired
    private UserService userService;

    // 基本映射
    @GetMapping("/{id}")
    public String getUser(@PathVariable Long id, Model model) {
        User user = userService.findById(id);
        model.addAttribute("user", user);
        return "user/detail";  // 逻辑视图名
    }

    // 复杂映射
    @GetMapping("/search")
    public String searchUsers(
            @RequestParam(required = false) String name,
            @RequestParam(defaultValue = "1") int page,
            @RequestParam(defaultValue = "10") int size,
            Model model) {

        Page<User> users = userService.searchUsers(name, page, size);
        model.addAttribute("users", users);
        model.addAttribute("currentPage", page);
        model.addAttribute("totalPages", users.getTotalPages());

        return "user/list";
    }

    // POST请求处理
    @PostMapping("/create")
    public String createUser(@Valid User user, BindingResult result) {
        if (result.hasErrors()) {
            return "user/create";  // 返回创建表单
        }

        userService.createUser(user);
        return "redirect:/users/" + user.getId();  // 重定向到详情页
    }

    // RESTful API
    @GetMapping(produces = MediaType.APPLICATION_JSON_VALUE)
    @ResponseBody
    public List<User> getAllUsers() {
        return userService.findAll();
    }

    @GetMapping("/{id}")
    @ResponseBody
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        User user = userService.findById(id);
        if (user == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(user);
    }

    @PostMapping
    @ResponseBody
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User created = userService.createUser(user);
        return ResponseEntity.created(URI.create("/users/" + created.getId())).body(created);
    }

    @PutMapping("/{id}")
    @ResponseBody
    public ResponseEntity<User> updateUser(
            @PathVariable Long id,
            @RequestBody User user) {

        User updated = userService.updateUser(id, user);
        if (updated == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(updated);
    }

    @DeleteMapping("/{id}")
    @ResponseBody
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        if (userService.deleteUser(id)) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }

    // 异常处理
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handleUserNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                          .body(ex.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleException(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                          .body("Internal Server Error");
    }

    // 拦截器
    @ModelAttribute("currentTime")
    public long addCurrentTime() {
        return System.currentTimeMillis();
    }

    @InitBinder
    public void initBinder(WebDataBinder binder) {
        binder.registerCustomEditor(Date.class, new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"), true));
    }
}

// 全局异常处理
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handleUserNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                          .body(ex.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(
            MethodArgumentNotValidException ex) {

        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });

        return ResponseEntity.badRequest().body(errors);
    }
}

// 自定义拦截器
@Component
public class LoggingInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        long startTime = System.currentTimeMillis();
        request.setAttribute("startTime", startTime);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response,
                          Object handler, ModelAndView modelAndView) {
        long startTime = (Long) request.getAttribute("startTime");
        long endTime = System.currentTimeMillis();
        long executeTime = endTime - startTime;

        if (handler instanceof HandlerMethod) {
            HandlerMethod handlerMethod = (HandlerMethod) handler;
            String methodName = handlerMethod.getMethod().getName();
            System.out.println("Method " + methodName + " executed in " + executeTime + "ms");
        }
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                               Object handler, Exception ex) {
        if (ex != null) {
            System.out.println("Exception occurred: " + ex.getMessage());
        }
    }
}

// 配置拦截器
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Autowired
    private LoggingInterceptor loggingInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loggingInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/static/**");
    }
}
```

### 5.2 视图解析与数据绑定

```java
import org.springframework.web.servlet.*;
import org.springframework.web.servlet.view.*;
import org.springframework.ui.*;
import org.springframework.web.bind.*;

// 视图解析器配置
@Configuration
public class ViewResolverConfig {
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        resolver.setExposeContextBeansAsAttributes(true);
        resolver.setViewClass(JstlView.class);
        return resolver;
    }

    @Bean
    public ViewResolver contentNegotiatingViewResolver(
            ContentNegotiationManager contentNegotiationManager) {

        ContentNegotiatingViewResolver resolver = new ContentNegotiatingViewResolver();
        resolver.setContentNegotiationManager(contentNegotiationManager);

        // 默认视图
        resolver.setDefaultViews(Arrays.asList(new MappingJackson2JsonView()));

        // 视图解析器
        List<ViewResolver> viewResolvers = new ArrayList<>();
        viewResolvers.add(internalResourceViewResolver());
        viewResolvers.add(thymeleafViewResolver());

        resolver.setViewResolvers(viewResolvers);
        return resolver;
    }

    @Bean
    public ViewResolver thymeleafViewResolver() {
        ThymeleafViewResolver resolver = new ThymeleafViewResolver();
        resolver.setTemplateEngine(templateEngine());
        resolver.setCharacterEncoding("UTF-8");
        resolver.setContentType("text/html");
        return resolver;
    }

    @Bean
    public SpringTemplateEngine templateEngine() {
        SpringTemplateEngine engine = new SpringTemplateEngine();
        engine.setTemplateResolver(templateResolver());
        return engine;
    }

    @Bean
    public ITemplateResolver templateResolver() {
        SpringResourceTemplateResolver resolver = new SpringResourceTemplateResolver();
        resolver.setPrefix("/WEB-INF/templates/");
        resolver.setSuffix(".html");
        resolver.setTemplateMode("HTML");
        resolver.setCharacterEncoding("UTF-8");
        return resolver;
    }

    @Bean
    public ViewResolver internalResourceViewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        return resolver;
    }
}

// 数据绑定和验证
@RestController
@RequestMapping("/api/products")
public class ProductController {
    @Autowired
    private ProductService productService;

    // 自定义数据绑定
    @InitBinder
    public void initBinder(WebDataBinder binder) {
        // 注册自定义编辑器
        binder.registerCustomEditor(Date.class, new CustomDateEditor(
            new SimpleDateFormat("yyyy-MM-dd"), true));

        // 设置允许的字段
        binder.setAllowedFields("name", "price", "description", "category");

        // 设置必需字段
        binder.setRequiredFields("name", "price");
    }

    // 创建产品
    @PostMapping
    public ResponseEntity<?> createProduct(@Valid @RequestBody ProductDTO productDTO,
                                        BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            Map<String, String> errors = new HashMap<>();
            bindingResult.getAllErrors().forEach(error -> {
                String fieldName = ((FieldError) error).getField();
                String errorMessage = error.getDefaultMessage();
                errors.put(fieldName, errorMessage);
            });
            return ResponseEntity.badRequest().body(errors);
        }

        Product product = convertToEntity(productDTO);
        Product created = productService.createProduct(product);
        return ResponseEntity.ok(convertToDTO(created));
    }

    // 文件上传
    @PostMapping("/upload")
    public ResponseEntity<String> uploadFile(@RequestParam("file") MultipartFile file) {
        if (file.isEmpty()) {
            return ResponseEntity.badRequest().body("Please select a file to upload");
        }

        try {
            byte[] bytes = file.getBytes();
            Path path = Paths.get("/uploads/" + file.getOriginalFilename());
            Files.write(path, bytes);

            return ResponseEntity.ok("File uploaded successfully: " + file.getOriginalFilename());
        } catch (IOException e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                              .body("Failed to upload file");
        }
    }

    // 下载文件
    @GetMapping("/download/{filename}")
    public ResponseEntity<Resource> downloadFile(@PathVariable String filename) {
        try {
            Path path = Paths.get("/uploads/" + filename);
            Resource resource = new UrlResource(path.toUri());

            if (resource.exists() && resource.isReadable()) {
                return ResponseEntity.ok()
                        .header(HttpHeaders.CONTENT_DISPOSITION,
                                "attachment; filename=\"" + resource.getFilename() + "\"")
                        .contentType(MediaType.APPLICATION_OCTET_STREAM)
                        .body(resource);
            } else {
                return ResponseEntity.notFound().build();
            }
        } catch (MalformedURLException e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }

    private Product convertToEntity(ProductDTO dto) {
        Product product = new Product();
        product.setName(dto.getName());
        product.setPrice(dto.getPrice());
        product.setDescription(dto.getDescription());
        product.setCategory(dto.getCategory());
        return product;
    }

    private ProductDTO convertToDTO(Product product) {
        ProductDTO dto = new ProductDTO();
        dto.setId(product.getId());
        dto.setName(product.getName());
        dto.setPrice(product.getPrice());
        dto.setDescription(product.getDescription());
        dto.setCategory(product.getCategory());
        return dto;
    }
}

// RESTful响应处理
@RestControllerAdvice
public class ResponseEntityAdvice {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            "INTERNAL_SERVER_ERROR",
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                          .body(error);
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            "NOT_FOUND",
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                          .body(error);
    }

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(ValidationException ex) {
        ErrorResponse error = new ErrorResponse(
            "VALIDATION_ERROR",
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                          .body(error);
    }
}
```

## 6. Spring Boot自动配置原理

### 6.1 自动配置机制

```java
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.context.annotation.*;
import org.springframework.core.type.classreading.*;
import org.springframework.core.env.*;

// Spring Boot自动配置核心机制
@SpringBootApplication
public class SpringBootApplicationExample {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootApplicationExample.class, args);
    }
}

// 自定义自动配置类
@Configuration
@ConditionalOnClass(MyService.class)
@ConditionalOnMissingBean(MyService.class)
@ConditionalOnProperty(prefix = "my.service", name = "enabled", havingValue = "true", matchIfMissing = true)
public class MyServiceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyService myService() {
        return new MyServiceImpl();
    }

    @Bean
    @ConditionalOnBean(MyService.class)
    public MyServiceClient myServiceClient(MyService myService) {
        return new MyServiceClient(myService);
    }
}

// 自定义starter的META-INF/spring.factories
// org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
// com.example.autoconfigure.MyServiceAutoConfiguration

// 条件注解的使用
@Configuration
public class ConditionalConfiguration {
    // 1. @ConditionalOnClass - 当类存在时配置
    @Bean
    @ConditionalOnClass(RedisTemplate.class)
    public RedisService redisService() {
        return new RedisServiceImpl();
    }

    // 2. @ConditionalOnMissingBean - 当Bean不存在时配置
    @Bean
    @ConditionalOnMissingBean(DataSource.class)
    public DataSource defaultDataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .build();
    }

    // 3. @ConditionalOnProperty - 当属性存在时配置
    @Bean
    @ConditionalOnProperty(prefix = "cache", name = "type", havingValue = "redis")
    public CacheManager redisCacheManager() {
        return new RedisCacheManager();
    }

    // 4. @ConditionalOnWebApplication - 当是Web应用时配置
    @Bean
    @ConditionalOnWebApplication
    public WebSecurityConfigurerAdapter webSecurityConfigurer() {
        return new CustomWebSecurityConfigurer();
    }

    // 5. @ConditionalOnNotWebApplication - 当不是Web应用时配置
    @Bean
    @ConditionalOnNotWebApplication
    public CommandLineRunner commandLineRunner() {
        return args -> {
            System.out.println("非Web应用启动");
        };
    }

    // 6. @ConditionalOnExpression - 基于SpEL表达式
    @Bean
    @ConditionalOnExpression("${app.feature.enabled} && '${app.version}' >= '2.0'")
    public FeatureService featureService() {
        return new FeatureServiceImpl();
    }
}

// 自定义条件
public class OnCustomCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment env = context.getEnvironment();
        String property = env.getProperty("custom.condition.property");
        return "enabled".equals(property);
    }
}

@Configuration
@Conditional(OnCustomCondition.class)
public class CustomConditionConfiguration {
    @Bean
    public CustomService customService() {
        return new CustomServiceImpl();
    }
}

// 自动配置属性
@ConfigurationProperties(prefix = "my.service")
public class MyServiceProperties {
    private String name = "DefaultService";
    private int timeout = 30;
    private boolean enabled = true;
    private Map<String, String> additionalConfig = new HashMap<>();

    // getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getTimeout() { return timeout; }
    public void setTimeout(int timeout) { this.timeout = timeout; }
    public boolean isEnabled() { return enabled; }
    public void setEnabled(boolean enabled) { this.enabled = enabled; }
    public Map<String, String> getAdditionalConfig() { return additionalConfig; }
    public void setAdditionalConfig(Map<String, String> additionalConfig) { this.additionalConfig = additionalConfig; }
}

// 自动配置的启用和禁用
@Configuration
@EnableConfigurationProperties(MyServiceProperties.class)
public class MyServiceAutoConfigurationProperties {
    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyServiceProperties properties) {
        MyService service = new MyServiceImpl();
        service.setName(properties.getName());
        service.setTimeout(properties.getTimeout());
        return service;
    }
}

// Spring Boot启动过程
@SpringBootApplication
public class SpringBootStartupProcess {
    public static void main(String[] args) {
        // 1. 创建SpringApplication
        SpringApplication app = new SpringApplication(SpringBootStartupProcess.class);

        // 2. 设置启动器
        app.setAdditionalProfiles("dev");
        app.setBannerMode(Banner.Mode.OFF);

        // 3. 运行应用
        app.run(args);
    }
}

// 自定义启动监听器
@Component
public class CustomApplicationListener implements ApplicationListener<ApplicationReadyEvent> {
    @Override
    public void onApplicationEvent(ApplicationReadyEvent event) {
        System.out.println("应用启动完成，可以接受请求");
    }
}

// 自定义启动器
@Component
public class CustomApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("应用启动完成，执行初始化任务");
    }
}
```

### 6.2 外部化配置

```java
import org.springframework.boot.context.properties.*;
import org.springframework.boot.env.*;
import org.springframework.core.env.*;

// 外部化配置
@Configuration
@PropertySource("classpath:custom.properties")
@ConfigurationProperties(prefix = "app")
public class ExternalizedConfiguration {
    private String name;
    private String version;
    private Database database;
    private List<String> features;
    private Map<String, String> config;

    // getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getVersion() { return version; }
    public void setVersion(String version) { this.version = version; }
    public Database getDatabase() { return database; }
    public void setDatabase(Database database) { this.database = database; }
    public List<String> getFeatures() { return features; }
    public void setFeatures(List<String> features) { this.features = features; }
    public Map<String, String> getConfig() { return config; }
    public void setConfig(Map<String, String> config) { this.config = config; }

    public static class Database {
        private String url;
        private String username;
        private String password;

        // getters and setters
        public String getUrl() { return url; }
        public void setUrl(String url) { this.url = url; }
        public String getUsername() { return username; }
        public void setUsername(String username) { this.username = username; }
        public String getPassword() { return password; }
        public void setPassword(String password) { this.password = password; }
    }
}

// 多环境配置
@Configuration
@Profile("dev")
public class DevelopmentConfiguration {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .addScript("classpath:schema.sql")
                .build();
    }
}

@Configuration
@Profile("prod")
public class ProductionConfiguration {
    @Bean
    @ConfigurationProperties(prefix = "app.database")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }
}

@Configuration
@Profile("test")
public class TestConfiguration {
    @Bean
    public DataSource testDataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .build();
    }
}

// 动态配置刷新
@RestController
@RefreshScope
public class DynamicConfigController {
    @Value("${app.dynamic.config}")
    private String dynamicConfig;

    @GetMapping("/config")
    public String getConfig() {
        return dynamicConfig;
    }
}

// 自定义属性源
@Component
public class CustomPropertySource implements EnvironmentPostProcessor {
    @Override
    public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
        Map<String, Object> customProperties = new HashMap<>();
        customProperties.put("custom.property", "custom-value");

        MapPropertySource propertySource = new MapPropertySource("customPropertySource", customProperties);
        environment.getPropertySources().addFirst(propertySource);
    }
}

// 配置验证
@Configuration
@Validated
public class ValidatedConfiguration {
    @Min(1)
    @Max(100)
    private int threadPoolSize;

    @NotBlank
    private String applicationName;

    @Email
    private String adminEmail;

    // getters and setters
    public int getThreadPoolSize() { return threadPoolSize; }
    public void setThreadPoolSize(int threadPoolSize) { this.threadPoolSize = threadPoolSize; }
    public String getApplicationName() { return applicationName; }
    public void setApplicationName(String applicationName) { this.applicationName = applicationName; }
    public String getAdminEmail() { return adminEmail; }
    public void setAdminEmail(String adminEmail) { this.adminEmail = adminEmail; }
}

// 配置元数据
@ConfigurationProperties(prefix = "app.cache")
public class CacheProperties {
    private int maxSize = 1000;
    private Duration expireAfterWrite = Duration.ofHours(1);
    private boolean enabled = true;

    // getters and setters
    public int getMaxSize() { return maxSize; }
    public void setMaxSize(int maxSize) { this.maxSize = maxSize; }
    public Duration getExpireAfterWrite() { return expireAfterWrite; }
    public void setExpireAfterWrite(Duration expireAfterWrite) { this.expireAfterWrite = expireAfterWrite; }
    public boolean isEnabled() { return enabled; }
    public void setEnabled(boolean enabled) { this.enabled = enabled; }
}
```

## 7. Spring生态圈深度解析

### 7.1 Spring生态系统

```java
// Spring生态系统的主要组件

// 1. Spring Core - 核心容器
@SpringBootApplication
public class SpringCoreExample {
    public static void main(String[] args) {
        SpringApplication.run(SpringCoreExample.class, args);
    }
}

// 2. Spring Data - 数据访问
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // 自动实现CRUD操作
    List<User> findByName(String name);
    List<User> findByEmailContaining(String email);
}

// 3. Spring Security - 安全框架
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/public/**").permitAll()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
                .formLogin();
    }
}

// 4. Spring Batch - 批处理
@Configuration
@EnableBatchProcessing
public class BatchConfig {
    @Bean
    public Job importUserJob(JobBuilderFactory jobs, Step step1) {
        return jobs.get("importUserJob")
                .incrementer(new RunIdIncrementer())
                .flow(step1)
                .end()
                .build();
    }

    @Bean
    public Step step1(JobRepository jobRepository, PlatformTransactionManager transactionManager,
                     ItemReader<User> reader, ItemWriter<User> writer, ItemProcessor<User, User> processor) {
        return new StepBuilder("step1", jobRepository)
                .<User, User>chunk(10, transactionManager)
                .reader(reader)
                .processor(processor)
                .writer(writer)
                .build();
    }
}

// 5. Spring Integration - 集成框架
@Configuration
@EnableIntegration
public class IntegrationConfig {
    @Bean
    public MessageChannel inputChannel() {
        return new DirectChannel();
    }

    @Bean
    public IntegrationFlow fileReadingFlow() {
        return IntegrationFlows
                .from(Files.inboundAdapter(new File("input")))
                .channel(inputChannel())
                .transform(Transformers.objectToString())
                .handle(message -> {
                    System.out.println("Received: " + message);
                    return null;
                })
                .get();
    }
}

// 6. Spring Cloud - 微服务
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker
public class CloudApplication {
    public static void main(String[] args) {
        SpringApplication.run(CloudApplication.class, args);
    }
}

// 7. Spring AMQP - 消息队列
@Configuration
@EnableRabbit
public class RabbitMQConfig {
    @Bean
    public Queue queue() {
        return new Queue("myQueue", false);
    }

    @Bean
    public DirectExchange exchange() {
        return new DirectExchange("myExchange");
    }

    @Bean
    public Binding binding(Queue queue, DirectExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with("routing.key");
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        return new RabbitTemplate(connectionFactory);
    }

    @Bean
    public MessageListenerAdapter listenerAdapter(MessageListener receiver) {
        return new MessageListenerAdapter(receiver, "receiveMessage");
    }

    @Bean
    public SimpleMessageListenerContainer container(ConnectionFactory connectionFactory, MessageListenerAdapter listenerAdapter) {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.setQueueNames("myQueue");
        container.setMessageListener(listenerAdapter);
        return container;
    }
}

// 8. Spring WebSocket - 实时通信
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic");
        registry.setApplicationDestinationPrefixes("/app");
    }
}

// 9. Spring Session - 会话管理
@Configuration
@EnableRedisHttpSession
public class SessionConfig {
    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory();
    }
}

// 10. Spring Actuator - 监控和运维
@Configuration
public class ActuatorConfig {
    @Bean
    public HealthIndicator customHealthIndicator() {
        return new HealthIndicator() {
            @Override
            public Health health() {
                // 自定义健康检查
                return Health.up().withDetail("custom", "OK").build();
            }
        };
    }
}
```

### 7.2 微服务架构

```java
// Spring Cloud微服务架构示例

// 1. 服务发现
@SpringBootApplication
@EnableDiscoveryClient
public class ServiceDiscoveryApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceDiscoveryApplication.class, args);
    }
}

// 2. 配置中心
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}

// 3. API网关
@SpringBootApplication
@EnableZuulProxy
@EnableDiscoveryClient
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}

// 4. 断路器
@SpringBootApplication
@EnableCircuitBreaker
@EnableDiscoveryClient
public class CircuitBreakerApplication {
    public static void main(String[] args) {
        SpringApplication.run(CircuitBreakerApplication.class, args);
    }
}

// 5. 链路追踪
@SpringBootApplication
@EnableZipkinServer
public class ZipkinServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZipkinServerApplication.class, args);
    }
}

// 6. 消息总线
@SpringBootApplication
@EnableBus
public class MessageBusApplication {
    public static void main(String[] args) {
        SpringApplication.run(MessageBusApplication.class, args);
    }
}

// 7. 分布式事务
@SpringBootApplication
@EnableTransactionManager
public class DistributedTransactionApplication {
    public static void main(String[] args) {
        SpringApplication.run(DistributedTransactionApplication.class, args);
    }
}

// 8. 服务熔断和降级
@Service
public class OrderService {
    @Autowired
    private PaymentServiceClient paymentServiceClient;

    @CircuitBreaker(fallbackMethod = "createOrderFallback")
    public Order createOrder(Order order) {
        // 调用支付服务
        Payment payment = paymentServiceClient.processPayment(order.getPayment());
        order.setPaymentId(payment.getId());
        return orderRepository.save(order);
    }

    public Order createOrderFallback(Order order, Exception e) {
        // 降级逻辑
        order.setStatus(OrderStatus.PENDING_PAYMENT);
        return orderRepository.save(order);
    }
}

// 9. 服务限流
@Configuration
public class RateLimitConfig {
    @Bean
    public FilterRegistrationBean<RateLimitFilter> rateLimitFilter() {
        FilterRegistrationBean<RateLimitFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new RateLimitFilter());
        registration.addUrlPatterns("/api/*");
        registration.setOrder(1);
        return registration;
    }
}

// 10. 服务监控
@RestController
@RequestMapping("/actuator")
public class CustomMetricsController {
    @Autowired
    private MeterRegistry meterRegistry;

    @GetMapping("/metrics")
    public Map<String, Object> getMetrics() {
        Map<String, Object> metrics = new HashMap<>();
        metrics.put("activeThreads", Thread.activeCount());
        metrics.put("memoryUsage", Runtime.getRuntime().totalMemory() - Runtime.getRuntime().freeMemory());
        metrics.put("systemLoad", ManagementFactory.getOperatingSystemMXBean().getSystemLoadAverage());
        return metrics;
    }
}
```

## 8. Spring性能优化与最佳实践

### 8.1 性能优化策略

```java
// Spring性能优化

// 1. 懒加载
@Configuration
public class LazyLoadConfig {
    @Bean
    @Lazy
    public ExpensiveService expensiveService() {
        return new ExpensiveServiceImpl();
    }

    @Bean
    public LazyInitBeanFactoryPostProcessor lazyInitBeanFactoryPostProcessor() {
        return new LazyInitBeanFactoryPostProcessor();
    }
}

// 2. 连接池优化
@Configuration
public class DataSourceConfig {
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.hikari")
    public DataSource dataSource() {
        return DataSourceBuilder.create()
                .type(HikariDataSource.class)
                .build();
    }
}

// 3. 缓存配置
@Configuration
@EnableCaching
public class CacheConfig {
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .maximumSize(1000));
        return cacheManager;
    }
}

// 4. 异步处理
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean
    public Executor asyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(25);
        executor.setThreadNamePrefix("Async-");
        executor.initialize();
        return executor;
    }
}

// 5. 事务优化
@Configuration
@EnableTransactionManagement
public class TransactionConfig {
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        transactionManager.setDefaultTimeout(30);
        transactionManager.setValidateExistingTransaction(true);
        return transactionManager;
    }
}

// 6. AOP优化
@Aspect
@Component
public class PerformanceAspect {
    @Around("execution(* com.example.service.*.*(..))")
    public Object logPerformance(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        try {
            return joinPoint.proceed();
        } finally {
            long endTime = System.currentTimeMillis();
            long duration = endTime - startTime;
            if (duration > 1000) {
                System.out.println("Slow method: " + joinPoint.getSignature().getName() + " took " + duration + "ms");
            }
        }
    }
}

// 7. 懒加载配置
@Configuration
public class LazyLoadConfiguration {
    @Bean
    public static BeanDefinitionRegistryPostProcessor lazyInitProcessor() {
        return new BeanDefinitionRegistryPostProcessor() {
            @Override
            public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
                for (String beanName : registry.getBeanDefinitionNames()) {
                    BeanDefinition beanDefinition = registry.getBeanDefinition(beanName);
                    if (beanDefinition.getBeanClassName() != null &&
                        beanDefinition.getBeanClassName().startsWith("com.example.service")) {
                        beanDefinition.setLazyInit(true);
                    }
                }
            }
        };
    }
}

// 8. 资源清理
@Configuration
public class ResourceCleanupConfig {
    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setFetchSize(100);
        jdbcTemplate.setQueryTimeout(30);
        return jdbcTemplate;
    }
}
```

### 8.2 最佳实践

```java
// Spring最佳实践

// 1. 依赖注入最佳实践
@Service
public class BestPracticeService {
    private final Dependency dependency;

    // 构造器注入
    @Autowired
    public BestPracticeService(Dependency dependency) {
        this.dependency = dependency;
    }

    // 避免字段注入
    // @Autowired
    // private Dependency dependency;

    // 避免循环依赖
    // @Autowired
    // private AnotherService anotherService;
}

// 2. 配置最佳实践
@Configuration
@ComponentScan(basePackages = "com.example")
@EnableJpaRepositories(basePackages = "com.example.repository")
@EntityScan(basePackages = "com.example.entity")
public class AppConfig {
    // 配置类应该保持简洁
    // 避免在配置类中编写业务逻辑
}

// 3. 异常处理最佳实践
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            "INTERNAL_SERVER_ERROR",
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                          .body(error);
    }

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(BusinessException ex) {
        ErrorResponse error = new ErrorResponse(
            "BUSINESS_ERROR",
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                          .body(error);
    }
}

// 4. 事务最佳实践
@Service
@Transactional
public class TransactionalService {
    @Transactional(readOnly = true)
    public User getUserById(Long id) {
        return userRepository.findById(id).orElse(null);
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void createWithNewTransaction() {
        // 新事务
    }

    // 避免长事务
    @Transactional(timeout = 30)
    public void processLargeData() {
        // 处理大量数据
    }
}

// 5. 缓存最佳实践
@Service
public class CacheService {
    @Cacheable(value = "users", key = "#id")
    public User getUser(Long id) {
        return userRepository.findById(id).orElse(null);
    }

    @CacheEvict(value = "users", key = "#user.id")
    public void updateUser(User user) {
        userRepository.save(user);
    }

    @CacheEvict(value = "users", allEntries = true)
    public void clearCache() {
        // 清除所有缓存
    }
}

// 6. 日志最佳实践
@Service
@Slf4j
public class LoggingService {
    private static final Logger logger = LoggerFactory.getLogger(LoggingService.class);

    public void performOperation(String input) {
        logger.info("Starting operation with input: {}", input);
        try {
            // 业务逻辑
            logger.debug("Operation completed successfully");
        } catch (Exception e) {
            logger.error("Operation failed: {}", e.getMessage(), e);
            throw e;
        }
    }
}

// 7. 测试最佳实践
@SpringBootTest
@AutoConfigureMockMvc
@Transactional
public class ServiceTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    public void testGetUser() throws Exception {
        mockMvc.perform(get("/api/users/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("John Doe"));
    }
}

// 8. 安全最佳实践
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/public/**").permitAll()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .logout()
                .logoutSuccessUrl("/login")
                .and()
                .csrf().disable();
    }
}

// 9. 配置管理最佳实践
@Configuration
@ConfigurationProperties(prefix = "app")
@Data
public class AppProperties {
    private String name;
    private int version;
    private Database database;
    private Cache cache;

    @Data
    public static class Database {
        private String url;
        private String username;
        private String password;
    }

    @Data
    public static class Cache {
        private int maxSize = 1000;
        private int expireAfterWrite = 3600;
    }
}

// 10. 监控最佳实践
@RestController
@RequestMapping("/actuator")
public class CustomHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        // 自定义健康检查
        boolean isHealthy = checkApplicationHealth();
        if (isHealthy) {
            return Health.up()
                    .withDetail("custom", "All systems operational")
                    .build();
        } else {
            return Health.down()
                    .withDetail("custom", "Some issues detected")
                    .build();
        }
    }

    private boolean checkApplicationHealth() {
        // 实现健康检查逻辑
        return true;
    }
}
```

## 9. 总结与展望

### 9.1 Spring框架的设计哲学总结

```java
// Spring框架设计哲学的体现

// 1. 简化Java开发
// 通过IoC和AOP简化企业级开发
@Service
public class SimplifiedDevelopment {
    @Autowired
    private Dependency dependency;

    @Transactional
    public void businessMethod() {
        // 简化的业务逻辑
        // 事务管理由AOP处理
    }
}

// 2. 非侵入式设计
// 最小化对业务代码的侵入
@Component
public class NonIntrusiveDesign {
    // 只需要简单的注解
    // 业务代码保持纯净
}

// 3. 灵活性和可扩展性
// 通过扩展点和插件机制实现灵活扩展
@Configuration
public class ExtensibleConfiguration {
    // 可以轻松扩展和定制
}

// 4. 最佳实践引导
// 框架设计引导开发者使用最佳实践
@Service
public class BestPracticeGuidance {
    // 构造器注入
    private final Dependency dependency;

    @Autowired
    public BestPracticeGuidance(Dependency dependency) {
        this.dependency = dependency;
    }
}

// 5. 测试友好
// 良好的可测试性设计
@Service
public class TestableService {
    private final Dependency dependency;

    @Autowired
    public TestableService(Dependency dependency) {
        this.dependency = dependency;
    }

    // 易于单元测试
}
```

### 9.2 Spring生态圈的未来展望

```java
// Spring生态圈的未来发展趋势

// 1. 云原生
// Spring Cloud Native、Kubernetes集成
@SpringBootApplication
public class CloudNativeApplication {
    public static void main(String[] args) {
        SpringApplication.run(CloudNativeApplication.class, args);
    }
}

// 2. 响应式编程
// Spring WebFlux、Project Reactor
@RestController
public class ReactiveController {
    @GetMapping("/reactive")
    public Mono<String> getReactiveData() {
        return Mono.just("Reactive Data");
    }
}

// 3. 函数式编程
// Spring Functional Web Framework
@Configuration
public class FunctionalWebConfig {
    @Bean
    public RouterFunction<ServerResponse> route() {
        return RouterFunctions.route()
                .GET("/functional", req -> ServerResponse.ok().body(Mono.just("Functional Web")))
                .build();
    }
}

// 4. 微服务架构
// Spring Cloud Microservices
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker
public class MicroserviceApplication {
    public static void main(String[] args) {
        SpringApplication.run(MicroserviceApplication.class, args);
    }
}

// 5. AI/ML集成
// Spring AI、机器学习集成
@Service
public class AIService {
    @Autowired
    private Model model;

    public Prediction predict(Data input) {
        return model.predict(input);
    }
}

// 6. 低代码/无代码
// Spring低代码开发平台
@Configuration
public class LowCodeConfig {
    // 自动生成代码和配置
}

// 7. Serverless
// Spring Serverless架构
@FunctionBean
public class ServerlessFunction {
    public String handle(String input) {
        return "Processed: " + input;
    }
}

// 8. 安全增强
// Spring Security 6.0+
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    // 更强的安全功能
}

// 9. 性能优化
// Spring性能持续优化
@Configuration
public class PerformanceConfig {
    // 自动性能优化
}

// 10. 开发者体验
// Spring Boot DevTools、Hot Reload
@Configuration
public class DeveloperExperienceConfig {
    // 提升开发体验的配置
}
```

## 结语

Spring框架已经从最初的IoC容器发展成为完整的企业级应用开发生态系统。通过本文的学习，你应该掌握了：

1. Spring框架的核心设计哲学和架构
2. IoC容器和依赖注入的深度实现
3. AOP的原理和应用场景
4. 事务管理的机制和配置
5. Spring MVC的工作流程和最佳实践
6. Spring Boot自动配置的原理
7. Spring生态圈的完整体系
8. 性能优化和最佳实践

Spring框架的成功在于它始终秉持"简化Java开发"的理念，通过优雅的设计和持续的演进，为Java开发者提供了强大的开发工具。**优秀的Spring开发者不仅要会使用框架，更要理解其设计思想和实现原理。**

随着云计算、微服务、响应式编程等技术的发展，Spring框架也在不断演进。Spring Boot 3.x、Spring 6等新版本引入了更多现代化的特性，让Spring框架在新时代继续保持其领先地位。

**Spring不仅是一个框架，更是一种思维方式。理解这种思维方式，才能在复杂的业务场景中做出正确的技术选择。**

---

*这篇文章深入剖析了Spring框架的设计哲学、核心实现和源码解析。通过大量的代码示例和实际案例，展示了Spring框架如何简化企业级Java开发，以及其在现代应用开发中的重要地位。浅者可以学会基本的使用方法，深者能够理解其设计哲学和底层原理。*