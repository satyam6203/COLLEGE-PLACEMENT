# Chapter 9: Spring Framework Fundamentals

**Chapter goal:** Explain Spring's IoC container, bean lifecycle, dependency resolution, component scanning, configuration classes, and AOP/proxy behavior — including the traps behind `@Transactional`, self-invocation, and prototype beans.

**Topics covered:** IoC and DI · `BeanFactory` vs `ApplicationContext` · injection types · autowiring, `@Primary`, `@Qualifier` · bean scopes and lifecycle · `BeanPostProcessor` vs `BeanFactoryPostProcessor` · component scanning and stereotypes · `@Bean` vs `@Component` · `@Configuration` proxy modes · circular dependencies · `@Lazy`, `ObjectProvider`, `Optional` · `@Value` and `@ConfigurationProperties` · `@Conditional` and profiles · Spring AOP · JDK vs CGLIB proxies · advice types and ordering · application events · `FactoryBean` · an `@Around` audit aspect.

**Cross-references:** Spring Boot auto-configuration is in [Chapter 10](10-Spring-Boot.md). REST APIs and validation are in [Chapter 11](11-REST-API-Development.md). Transaction proxies are in [Chapter 12](12-Transactions-and-Scheduling.md).

---

## Part A — IoC container and dependency injection

### Question 1: What are IoC and dependency injection in Spring?

**1. Interview-ready answer**

Inversion of Control means the framework owns object creation and wiring instead of application code doing `new` everywhere. Dependency injection is the mechanism: Spring creates beans and supplies their collaborators. This makes classes loosely coupled, easier to test, and centrally configurable.

**2. Detailed explanation**

Without IoC, a service constructs its own repository:

```java
OrderRepository repository = new JdbcOrderRepository();
```

With DI, the service declares what it needs:

```java
private final OrderRepository repository;

OrderService(OrderRepository repository) {
    this.repository = repository;
}
```

Spring decides which implementation to inject based on configuration and bean definitions.

**3. Internal working**

The container reads configuration (annotations, XML, or Java config), creates bean definitions, instantiates beans, resolves dependencies, applies post-processors, initializes them, and stores them in a bean factory. Dependencies are injected by constructor, setter, or field.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;
import org.springframework.stereotype.Service;

interface PriceGateway {

    BigDecimal price(String sku);
}

@Service
class DatabasePriceGateway implements PriceGateway {

    @Override
    public BigDecimal price(String sku) {
        return BigDecimal.valueOf(100);
    }
}

@Service
class OrderService {

    private final PriceGateway priceGateway;

    OrderService(PriceGateway priceGateway) {
        this.priceGateway = priceGateway;
    }

    public BigDecimal total(String sku, int quantity) {
        return priceGateway.price(sku).multiply(BigDecimal.valueOf(quantity));
    }
}
```

**5. Real-world use case**

A Spring service injects a `PaymentGateway` interface. Production uses Stripe; tests inject a fake gateway without changing service code.

**6. Interview follow-up questions**

- **What is a bean?** An object managed by the Spring container.
- **Is DI the same as IoC?** DI is one form of IoC.
- **Can a bean inject itself?** Technically yes, but usually a design smell.
- **Does Spring inject all fields automatically?** Only annotated constructor parameters/fields/setters or auto-injected constructor in some cases.
- **What is the benefit for testing?** Dependencies can be mocked independently.

**7. Common mistakes**

- Calling `new` for collaborators inside services.
- Using field injection because it is shorter.
- Depending on concrete classes instead of interfaces.
- Treating every helper object as a Spring bean.

**8. Interview difficulty**

**Beginner**

---

### Question 2: What is the difference between `BeanFactory` and `ApplicationContext`?

**1. Interview-ready answer**

`BeanFactory` is the core IoC container with basic bean creation and lookup. `ApplicationContext` extends it and adds enterprise features: event publishing, resource loading, message sources, environment/profile support, AOP integration, and automatic `BeanPostProcessor` registration. In Spring Boot, the context you normally use is an `ApplicationContext`.

**2. Detailed explanation**

| Feature | `BeanFactory` | `ApplicationContext` |
|---|---|---|
| Bean creation/lookup | Yes | Yes |
| Lazy bean creation by default | Yes | Eager singleton pre-instantiation |
| Events | No | Yes |
| Resource loading | Basic | Enhanced `ResourceLoader` |
| Environment/profiles | No | Yes |
| `BeanPostProcessor` auto-detection | Manual | Automatic |
| Internationalization | No | Yes |

**3. Internal working**

`ApplicationContext` wraps a `DefaultListableBeanFactory` and adds infrastructure beans. During refresh, it loads bean definitions, registers post-processors, initializes message sources and event multicasters, and pre-instantiates non-lazy singletons.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Configuration;

@Configuration
class AppConfig {
}

public class ContextDemo {

    public static void main(String[] args) {
        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(AppConfig.class)) {
            System.out.println(context.getBeanNamesForType(AppConfig.class).length);
            System.out.println(context.getClass().getSimpleName());
        }
    }
}
```

Output:

```text
1
AnnotationConfigApplicationContext
```

**5. Real-world use case**

Spring Boot creates an `AnnotationConfigApplicationContext` or reactive variant at startup; understanding refresh explains why `@PostConstruct` and bean post-processors run before the application accepts traffic.

**6. Interview follow-up questions**

- **Which one should I use?** `ApplicationContext`.
- **Does ApplicationContext extend BeanFactory?** Yes.
- **Are beans lazy in Spring Boot?** Singletons are eager by default unless `@Lazy` is used.
- **What is context refresh?** The startup phase that builds the container.
- **Can there be parent contexts?** Yes, e.g. Spring MVC child context.

**7. Common mistakes**

- Using raw BeanFactory and expecting events or AOP to work automatically.
- Assuming all beans are lazy.
- Creating multiple application contexts unnecessarily in tests.

**8. Interview difficulty**

**Beginner**

---

### Question 3: Compare constructor, setter, and field injection

**1. Interview-ready answer**

Constructor injection is recommended: it makes dependencies mandatory, supports `final` fields, preserves immutability, and keeps beans valid after construction. Setter injection is useful for optional or reconfigurable dependencies. Field injection is concise but hides dependencies, prevents final fields, and makes tests harder.

**2. Detailed explanation**

| Injection type | Pros | Cons |
|---|---|---|
| Constructor | Immutable, testable, fails fast | Many parameters can signal too many dependencies |
| Setter | Optional/replaceable dependencies | Object can be partially configured |
| Field | Short code | Hidden, reflection-based, not final |

Spring 4.3+ automatically uses a single constructor without `@Autowired`. Multiple constructors require `@Autowired` or explicit configuration.

**3. Internal working**

Constructor injection happens during bean instantiation. Setter/field injection occurs in the population phase, before initialization callbacks and `BeanPostProcessor.afterInitialization`.

**4. Practical Java/Spring Boot example**

```java
import java.util.Optional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

interface AuditClient {

    void record(String event);
}

@Service
class OrderService {

    private final PaymentClient paymentClient;
    private AuditClient auditClient;

    OrderService(PaymentClient paymentClient) {
        this.paymentClient = paymentClient;
    }

    @Autowired
    void setAuditClient(Optional<AuditClient> auditClient) {
        this.auditClient = auditClient.orElse(event -> { });
    }

    void checkout(String orderId) {
        paymentClient.charge(orderId);
        auditClient.record("checkout:" + orderId);
    }
}
```

**5. Real-world use case**

A service requires `PaymentClient` but has an optional metrics client; constructor injection enforces the required dependency while setter injection handles the optional one.

**6. Interview follow-up questions**

- **Why prefer constructor injection?** Immutability and explicit dependencies.
- **What if there are two constructors?** Annotate one with `@Autowired`.
- **Is field injection bad?** It works, but hides coupling and complicates tests.
- **Can final fields be setter-injected?** No.
- **What is circular constructor injection?** A cycle that cannot be constructed.

**7. Common mistakes**

- Mixing all three styles in one class.
- Using field injection in unit tests that do not load Spring.
- Treating optional dependencies as required.
- Hiding ten dependencies behind field injection.

**8. Interview difficulty**

**Beginner**

---

### Question 4: How does Spring resolve dependencies when multiple beans match?

**1. Interview-ready answer**

Spring first resolves by type. If multiple candidates exist, it applies primary/qualifier rules, then bean name matching, then priority annotations. `@Primary` marks the default; `@Qualifier` selects a named bean; `@Resource` resolves by name first (JSR-250 style). If ambiguity remains, startup fails with `NoUniqueBeanDefinitionException`.

**2. Detailed explanation**

Resolution order conceptually:

1. Collect all beans assignable to the required type.
2. Remove beans not eligible for autowiring.
3. Apply `@Primary`.
4. Apply `@Qualifier` / custom qualifiers.
5. Match parameter/field name to bean name.
6. Apply `@Priority` where relevant.
7. Fail if not exactly one candidate.

**3. Internal working**

`DefaultListableBeanFactory.resolveDependency` delegates to `resolveNamedBean`, which checks primary candidates and qualifier metadata from `AutowireCandidateResolver`.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Service;

interface PaymentGateway {

    void charge(String orderId);
}

@Service
@Primary
class StripeGateway implements PaymentGateway {

    @Override
    public void charge(String orderId) {
        System.out.println("stripe " + orderId);
    }
}

@Service("paypalGateway")
class PayPalGateway implements PaymentGateway {

    @Override
    public void charge(String orderId) {
        System.out.println("paypal " + orderId);
    }
}

@Service
class CheckoutService {

    private final PaymentGateway defaultGateway;
    private final PaymentGateway paypalGateway;

    CheckoutService(PaymentGateway defaultGateway,
                    @Qualifier("paypalGateway") PaymentGateway paypalGateway) {
        this.defaultGateway = defaultGateway;
        this.paypalGateway = paypalGateway;
    }

    void checkout(String orderId) {
        defaultGateway.charge(orderId);
        paypalGateway.charge(orderId);
    }
}
```

Output when called with `ORD-1`:

```text
stripe ORD-1
paypal ORD-1
```

**5. Real-world use case**

A multi-tenant application has two `RestTemplate` beans and uses `@Qualifier` to select the one configured for an external tax service.

**6. Interview follow-up questions**

- **What happens with two beans and no primary?** NoUniqueBeanDefinitionException.
- **Does bean name really matter?** Yes, as a fallback when parameter names are available.
- **@Qualifier vs @Resource?** Qualifier is type-first with qualifier; Resource is name-first.
- **Can @Primary be on multiple beans?** Yes, but then ambiguity returns.
- **What is a custom qualifier?** Your own annotation meta-annotated with `@Qualifier`.

**7. Common mistakes**

- Depending on parameter names compiled without `-parameters`.
- Marking too many beans @Primary.
- Misspelling qualifier values.
- Using `@Autowired` on fields in tests without context.

**8. Interview difficulty**

**Intermediate**

---

### Question 5: What bean scopes does Spring provide?

**1. Interview-ready answer**

The main scopes are `singleton` (one instance per container, default), `prototype` (new instance per request), `request`, `session`, `application`, and `websocket` for web contexts. Singleton beans are stateless and shared; prototype beans must be handled carefully because Spring does not manage their full lifecycle after creation.

**2. Detailed explanation**

| Scope | Lifetime |
|---|---|
| `singleton` | One bean per `ApplicationContext` |
| `prototype` | New bean each injection/lookup |
| `request` | One bean per HTTP request |
| `session` | One bean per HTTP session |
| `application` | One bean per `ServletContext` |
| `websocket` | One bean per WebSocket session |

Web scopes require an active web request context. Prototype beans get construction and initialization callbacks, but `@PreDestroy` is not invoked by the container.

**3. Internal working**

Singleton beans are cached in `singletonObjects` after creation. Prototype beans are created through `createBean` each time and not cached. Request/session scopes store beans in request/session attributes via a `Scope` implementation.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Service;

@Service
class ReportBuilder {
}

@Scope("prototype")
class ReportRequestContext {
}

@Service
class ReportService {

    private final ObjectProvider<ReportRequestContext> contextProvider;

    ReportService(ObjectProvider<ReportRequestContext> contextProvider) {
        this.contextProvider = contextProvider;
    }

    void build() {
        ReportRequestContext first = contextProvider.getObject();
        ReportRequestContext second = contextProvider.getObject();

        System.out.println(first == second);
    }
}
```

Output:

```text
false
```

Each `getObject()` creates a new prototype instance.

**5. Real-world use case**

A report service needs a fresh mutable context per export job but should remain a singleton; `ObjectProvider` avoids permanently injecting one prototype into a singleton.

**6. Interview follow-up questions**

- **Are singleton beans thread-safe?** Spring does not make them safe; your fields must be.
- **Does prototype destroy callbacks run?** No.
- **How do web scopes work in tests?** Use `RequestContextHolder` or MockMvc.
- **Can a singleton depend on a request-scoped bean?** Yes through a scoped proxy or provider.
- **Is prototype a performance problem?** Creation cost can be high.

**7. Common mistakes**

- Storing request state in singleton services.
- Expecting `@PreDestroy` on prototype beans.
- Injecting prototype directly into singleton and expecting new instances per call.
- Using web scopes outside a web context.

**8. Interview difficulty**

**Intermediate**

---

---

## Part B — Bean lifecycle and configuration

### Question 6: Describe the Spring bean lifecycle

**1. Interview-ready answer**

Spring creates a bean by instantiating it, populating properties/dependencies, invoking Aware callbacks, running `BeanPostProcessor` pre-initialization hooks and initialization callbacks, running post-initialization hooks, using the bean, and finally invoking destruction callbacks on context shutdown. `@PostConstruct`, `InitializingBean`, and custom `init-method` are initialization hooks; `@PreDestroy`, `DisposableBean`, and custom `destroy-method` are destruction hooks.

**2. Detailed explanation**

Typical order:

```text
Bean definition loaded
 -> instantiate
 -> populate properties / inject dependencies
 -> BeanNameAware / BeanFactoryAware / ApplicationContextAware
 -> BeanPostProcessor.beforeInitialization
 -> @PostConstruct
 -> InitializingBean.afterPropertiesSet
 -> custom init-method
 -> BeanPostProcessor.afterInitialization
 -> bean ready for use
 -> @PreDestroy
 -> DisposableBean.destroy
 -> custom destroy-method
```

**3. Internal working**

`AbstractAutowireCapableBeanFactory.createBean` drives most of this flow. `BeanPostProcessor` hooks are applied to every bean instance, while initialization callbacks are bean-specific. AOP proxies are commonly created in `postProcessAfterInitialization`, which is why self-invocation bypasses advice.

**4. Practical Java/Spring Boot example**

```java
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.stereotype.Component;

@Component
class LifecycleBean implements InitializingBean, DisposableBean {

    LifecycleBean() {
        System.out.println("1 constructor");
    }

    @PostConstruct
    void postConstruct() {
        System.out.println("2 postConstruct");
    }

    @Override
    public void afterPropertiesSet() {
        System.out.println("3 afterPropertiesSet");
    }

    @PreDestroy
    void preDestroy() {
        System.out.println("4 preDestroy");
    }

    @Override
    public void destroy() {
        System.out.println("5 destroy");
    }
}
```

On shutdown, the destruction methods run in the declared order.

**5. Real-world use case**

A cache warmer fills local state in `@PostConstruct`, and a connection pool is closed in `@PreDestroy`.

**6. Interview follow-up questions**

- **Where are AOP proxies created?** Usually in post-process-after-initialization.
- **Which initialization hook is preferred?** `@PostConstruct` for application code.
- **Are prototype destroy methods invoked?** No.
- **What runs before dependency injection?** Construction; injection happens in populate.
- **Can BeanPostProcessor change the bean?** Yes, it can return a proxy/wrapper.

**7. Common mistakes**

- Doing heavy I/O in constructors.
- Assuming field injection is complete in the constructor.
- Depending on exact ordering of multiple init mechanisms.
- Forgetting graceful shutdown for background resources.

**8. Interview difficulty**

**Intermediate**

---

### Question 7: What is the difference between `BeanPostProcessor` and `BeanFactoryPostProcessor`?

**1. Interview-ready answer**

A `BeanFactoryPostProcessor` runs after bean definitions are loaded but before beans are instantiated; it can change metadata such as property values or scope. A `BeanPostProcessor` runs around bean initialization on actual instances; it can modify or wrap beans, which is how Spring AOP and many annotations work.

**2. Detailed explanation**

| Feature | BeanFactoryPostProcessor | BeanPostProcessor |
|---|---|---|
| Operates on | Bean definitions | Bean instances |
| Timing | Before instantiation | During initialization |
| Can change bean class metadata | Yes | No |
| Can return proxy | No | Yes |
| Common use | Property placeholder config | AOP, validation, injection annotations |

`ConfigurationPropertiesBindingPostProcessor` and `AutowiredAnnotationBeanPostProcessor` are examples of bean post-processors.

**3. Internal working**

`ApplicationContext.refresh()` invokes registered `BeanFactoryPostProcessor`s first. Later, for each bean, `getBean` invokes all applicable `BeanPostProcessor` callbacks.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Component;

@Component
class DefinitionModifier implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory factory)
            throws BeansException {
        BeanDefinition definition = factory.getBeanDefinition("orderService");
        definition.setLazyInit(true);
    }
}

@Component
class InstanceLogger implements BeanPostProcessor {

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName)
            throws BeansException {
        if (beanName.equals("orderService")) {
            System.out.println("initialized " + beanName);
        }
        return bean;
    }
}
```

**5. Real-world use case**

A framework sets `lazyInit` on selected definitions via a BFPP, while an AOP auto-proxy creator wraps repository beans via a BPP.

**6. Interview follow-up questions**

- **Can BeanFactoryPostProcessor see instances?** No.
- **Can BeanPostProcessor modify BeanDefinition?** No.
- **Why are BPPs applied to every bean?** They implement cross-cutting behavior.
- **Do BPPs apply to prototype beans?** Yes, on each creation.
- **Can BPP order matter?** Yes; implement `Ordered` or use `@Order`.

**7. Common mistakes**

- Injecting beans into a BPP too early.
- Modifying definitions after instantiation.
- Forgetting to return the bean from BPP callbacks.
- Ignoring BPP ordering with proxies.

**8. Interview difficulty**

**Advanced**

---

### Question 8: What are component scanning and stereotype annotations?

**1. Interview-ready answer**

Component scanning tells Spring which packages to search for classes annotated with `@Component` and its specializations. `@Repository`, `@Service`, `@Controller`, and `@Configuration` are meta-annotated with `@Component` and add semantic meaning or extra behavior, such as repository exception translation. `@SpringBootApplication` enables scanning from its package by default.

**2. Detailed explanation**

Stereotypes:

| Annotation | Meaning |
|---|---|
| `@Component` | Generic managed component |
| `@Repository` | Persistence layer; exception translation |
| `@Service` | Business service layer |
| `@Controller` | MVC controller |
| `@RestController` | `@Controller` + `@ResponseBody` |
| `@Configuration` | Source of `@Bean` definitions |

Scanning can be customized with `basePackages`, `includeFilters`, and `excludeFilters`.

**3. Internal working**

`ClassPathScanningCandidateComponentProvider` scans classpath metadata for candidate components and registers `AnnotatedBeanDefinition`s. Meta-annotations are discovered through annotation metadata, so a custom `@Service`-like annotation can itself be a component.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication
@ComponentScan(basePackages = {"com.example.orders", "com.example.common"})
public class OrdersApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrdersApplication.class, args);
    }
}
```

`com.example.orders` and `com.example.common` are scanned for stereotyped classes.

**5. Real-world use case**

A shared library's components live outside the application's main package, so the application explicitly adds the library package to component scanning.

**6. Interview follow-up questions**

- **Does @SpringBootApplication include @ComponentScan?** Yes.
- **What is the default scan package?** The package of the `@SpringBootApplication` class and subpackages.
- **Is @Repository only semantic?** It also enables persistence exception translation.
- **Can you scan a jar dependency?** Yes, if it is on the classpath.
- **How do you exclude a component?** `excludeFilters` or `@Conditional`.

**7. Common mistakes**

- Placing the main class too high or too low in the package tree.
- Scanning too broadly and loading unintended beans.
- Forgetting to scan shared modules.
- Using `@Component` for everything.

**8. Interview difficulty**

**Beginner**

---

### Question 9: When do you use `@Bean` vs `@Component`?

**1. Interview-ready answer**

Use `@Component` when you control the class and want it discovered by scanning. Use `@Bean` when you need explicit factory logic, want to configure a third-party class, need conditional creation, or must choose among multiple implementations. `@Bean` gives you full control over construction; `@Component` is simpler for application-owned classes.

**2. Detailed explanation**

| Aspect | `@Component` | `@Bean` |
|---|---|---|
| Applied to | Class | Method in `@Configuration` |
| Discovery | Component scanning | Configuration parsing |
| Third-party classes | Cannot annotate | Ideal |
| Conditional creation | Limited | `@Conditional*` easy |
| Multiple instances | One class definition | Multiple methods possible |
| Constructor logic | Class constructor | Factory method |

**3. Internal working**

A `@Component` class becomes an `AnnotatedGenericBeanDefinition`. A `@Bean` method becomes a `ConfigurationClassBeanDefinition` whose factory bean is the configuration class and whose factory method is the annotated method.

**4. Practical Java/Spring Boot example**

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Component;

@Component
class OrderMapper {
}

@Configuration
class JacksonConfig {

    @Bean
    ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

`OrderMapper` is discovered by scanning; `ObjectMapper` is registered explicitly.

**5. Real-world use case**

An application creates a configured `RestTemplate`, `ObjectMapper`, or Kafka producer through `@Bean` because those classes come from libraries and need custom setup.

**6. Interview follow-up questions**

- **What is the default @Bean name?** The method name.
- **Can @Bean methods have parameters?** Yes; Spring injects them.
- **Can a class have both @Component and @Bean methods?** If annotated `@Configuration`, yes.
- **Which is better for interfaces?** `@Bean` when choosing implementation dynamically.
- **Can @Bean be used in @Component class?** Yes, in lite configuration mode.

**7. Common mistakes**

- Annotating third-party classes you do not own.
- Using component scanning for conditional beans.
- Assuming @Bean methods are ordinary calls in full config mode.
- Forgetting `@Configuration` for interception.

**8. Interview difficulty**

**Beginner**

---

### Question 10: What does `@Configuration(proxyBeanMethods = ...)` do?

**1. Interview-ready answer**

With `proxyBeanMethods = true` (the default), Spring CGLIB-proxies the `@Configuration` class so calls between `@Bean` methods return the same singleton from the container. Setting it to false uses lite mode: no proxy, direct method calls create new objects. Lite mode starts faster and is recommended when beans do not call each other.

**2. Detailed explanation**

Full mode:

```java
@Configuration(proxyBeanMethods = true)
class FullConfig {

    @Bean
    PriceClient priceClient() {
        return new PriceClient();
    }

    @Bean
    PricingService pricingService() {
        return new PricingService(priceClient());
    }
}
```

`pricingService()` receives the container-managed singleton `PriceClient`.

Lite mode:

```java
@Configuration(proxyBeanMethods = false)
class LiteConfig {

    @Bean
    PriceClient priceClient() {
        return new PriceClient();
    }

    @Bean
    PricingService pricingService(PriceClient client) {
        return new PricingService(client);
    }
}
```

Dependencies are injected as method parameters instead of calling another bean method.

**3. Internal working**

Full configuration classes are subclassed with CGLIB; intercepted `@Bean` methods call `beanFactory.getBean` to enforce singleton semantics. Lite mode avoids class generation and proxy overhead.

**4. Practical Java/Spring Boot example**

```java
class PriceClient {
}

class PricingService {

    PricingService(PriceClient client) {
    }
}

@Configuration(proxyBeanMethods = false)
class AppConfig {

    @Bean
    PriceClient priceClient() {
        return new PriceClient();
    }

    @Bean
    PricingService pricingService(PriceClient priceClient) {
        return new PricingService(priceClient);
    }
}
```

This is the recommended style for most Spring Boot configuration classes.

**5. Real-world use case**

Large Boot applications use `proxyBeanMethods = false` on configuration classes to reduce startup time and CGLIB class generation.

**6. Interview follow-up questions**

- **Why is proxyBeanMethods true by default?** Backward compatibility and inter-bean calls.
- **Does lite mode break singletons?** No, if dependencies are injected as parameters.
- **Can @Bean be in a non-@Configuration class?** Yes, but lite mode applies.
- **Does proxyBeanMethods affect all beans?** Only the configuration class itself.
- **Is CGLIB available in Spring Boot?** Yes, via spring-core.

**7. Common mistakes**

- Setting false while still calling @Bean methods directly.
- Assuming lite mode changes bean scopes.
- Using final @Bean methods in full mode.
- Overusing full mode for simple configs.

**8. Interview difficulty**

**Advanced**

---

---

## Part C — Resolution, conditions, and AOP foundations

### Question 11: How do circular dependencies happen and how do you fix them?

**1. Interview-ready answer**

A circular dependency occurs when bean A needs B and bean B needs A. Constructor cycles cannot be satisfied because neither object can be created first; Spring Boot 2.6+ disables circular references by default. Setter/field cycles were historically tolerated through early singleton exposure, but can yield partially initialized beans. The best fix is to redesign the dependency direction, not to re-enable the flag.

**2. Detailed explanation**

Example cycle:

```text
OrderService -> PaymentService -> OrderService
```

Common fixes:

1. Extract the shared logic into a third bean both depend on.
2. Use events to invert one direction.
3. Inject `ObjectProvider<B>` or `@Lazy` to delay resolution.
4. Depend on an interface implemented by one side.
5. Use setter injection only as a last resort.

**3. Internal working**

Singleton creation uses "currently in creation" registries and early singleton objects. Constructor injection cannot expose an early reference because the object does not exist yet. Setter injection can expose a partially created instance, which is why Spring now fails fast by default.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Service;

@Service
class OrderService {

    private final PaymentService paymentService;

    OrderService(@Lazy PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    void checkout(String orderId) {
        paymentService.charge(orderId);
    }
}

@Service
class PaymentService {

    private final OrderService orderService;

    PaymentService(OrderService orderService) {
        this.orderService = orderService;
    }

    void charge(String orderId) {
        orderService.markPaid(orderId);
    }

    void markPaidUnused() {
    }
}
```

`OrderService` receives a lazy proxy for `PaymentService`, breaking the constructor cycle. Refactoring is still preferable.

**5. Real-world use case**

A notification service depends on a user service that publishes events back to the notification service; replacing the direct callback with `ApplicationEventPublisher` removes the cycle.

**6. Interview follow-up questions**

- **Why do constructor cycles fail?** No instance can be created before the other.
- **What does spring.main.allow-circular-references do?** Re-enables legacy behavior; not recommended.
- **Does @Lazy always work?** It works by proxying, but hides design problems.
- **What is an early singleton?** A partially initialized bean exposed to break a cycle.
- **How do events fix cycles?** One bean publishes; the other listens without a direct dependency.

**7. Common mistakes**

- Re-enabling circular references in production.
- Using field injection to hide the cycle.
- Creating two beans that call each other synchronously.
- Assuming @Lazy removes the dependency.

**8. Interview difficulty**

**Intermediate**

---

### Question 12: How do `@Lazy`, `ObjectProvider`, and `Optional` help with dependency injection?

**1. Interview-ready answer**

`@Lazy` postpones bean creation until first use or creates a lazy proxy for injection. `ObjectProvider<T>` gives lookup-on-demand methods such as `getObject`, `getIfAvailable`, and `orderedStream`. `Optional<T>` expresses that a dependency may be absent. They solve different problems: lazy initialization, dynamic lookup, and optionality.

**2. Detailed explanation**

| Mechanism | Purpose |
|---|---|
| `@Lazy` on bean | Delay creation |
| `@Lazy` on injection point | Inject lazy proxy |
| `ObjectProvider<T>` | Programmatic lookup, multiple candidates, optional get |
| `Optional<T>` | Dependency may be missing |

**3. Internal working**

`ObjectProvider` is backed by the bean factory and resolves at call time. An `Optional` dependency becomes `empty` when no candidate exists. A lazy proxy is a generated proxy that resolves the target on first method invocation.

**4. Practical Java/Spring Boot example**

```java
import java.util.Optional;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Service;

interface FeatureFlagClient {

    boolean enabled(String flag);
}

@Service
class ExpensiveReportService {

    ExpensiveReportService() {
        System.out.println("created");
    }
}

@Configuration
class LazyConfig {

    @Bean
    @Lazy
    ExpensiveReportService expensiveReportService() {
        return new ExpensiveReportService();
    }
}

@Service
class FeatureService {

    private final ObjectProvider<FeatureFlagClient> clientProvider;
    private final Optional<AuditSink> auditSink;

    FeatureService(ObjectProvider<FeatureFlagClient> clientProvider,
                   Optional<AuditSink> auditSink) {
        this.clientProvider = clientProvider;
        this.auditSink = auditSink;
    }

    void check() {
        FeatureFlagClient client = clientProvider.getIfAvailable();
        if (client != null && client.enabled("new-checkout")) {
            auditSink.ifPresent(sink -> sink.record("enabled"));
        }
    }
}
```

**5. Real-world use case**

A startup-critical web service injects an optional audit sink and lazily resolves a report generator that is used only by an admin endpoint.

**6. Interview follow-up questions**

- **Does Optional avoid startup failure for required beans?** Only if the dependency is declared Optional.
- **Is ObjectProvider lazy?** It defers lookup until method calls.
- **Does @Lazy on a singleton change its scope?** No.
- **What if multiple candidates for ObjectProvider?** `getObject` may throw; use `orderedStream`.
- **Can @Lazy break proxies?** It adds another proxy layer.

**7. Common mistakes**

- Using Optional for genuinely required dependencies.
- Calling `getObject()` too early.
- Believing @Lazy makes a singleton prototype.
- Hiding missing configuration with Optional.

**8. Interview difficulty**

**Intermediate**

---

### Question 13: How do `@Value` and `@ConfigurationProperties` differ?

**1. Interview-ready answer**

`@Value` injects a single resolved property value using SpEL/placeholder syntax. `@ConfigurationProperties` binds a whole typed group of properties to an object with relaxed binding, conversion, and validation. Use `@ConfigurationProperties` for cohesive configuration groups and `@Value` for simple one-off values.

**2. Detailed explanation**

| Aspect | `@Value` | `@ConfigurationProperties` |
|---|---|---|
| Scope | Single value | Group of properties |
| Type safety | Weak | Strong |
| Relaxed binding | Limited | Yes |
| Validation | Manual | `@Validated` + constraints |
| Metadata | No | IDE support via processor |

**3. Internal working**

`PropertySourcesPlaceholderConfigurer` resolves `${...}` placeholders. `ConfigurationPropertiesBindingPostProcessor` binds `Environment` values to bean setters/fields using `Binder`.

**4. Practical Java/Spring Boot example**

```java
import jakarta.validation.constraints.Min;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
import org.springframework.validation.annotation.Validated;

@Component
@Validated
@ConfigurationProperties(prefix = "orders")
class OrderProperties {

    @Min(1)
    private int timeoutSeconds = 5;

    private String currency = "INR";

    public int getTimeoutSeconds() {
        return timeoutSeconds;
    }

    public void setTimeoutSeconds(int timeoutSeconds) {
        this.timeoutSeconds = timeoutSeconds;
    }

    public String getCurrency() {
        return currency;
    }

    public void setCurrency(String currency) {
        this.currency = currency;
    }
}

@Component
class TaxClient {

    private final String endpoint;

    TaxClient(@Value("${tax.endpoint:http://localhost:8081}") String endpoint) {
        this.endpoint = endpoint;
    }
}
```

Properties `orders.timeout-seconds` and `orders.currency` bind to `OrderProperties`.

**5. Real-world use case**

A service binds a `kafka.consumer.*` property group into a validated configuration class instead of scattering `@Value` fields.

**6. Interview follow-up questions**

- **What is relaxed binding?** `timeout-seconds`, `timeoutSeconds`, and `TIMEOUT_SECONDS` can bind.
- **Can @Value use defaults?** Yes: `${key:default}`.
- **How do you validate config?** `@Validated` plus Jakarta constraints.
- **Where do properties come from?** application.yml, env vars, system props, command line, custom sources.
- **Which is better for tests?** ConfigurationProperties objects are easy to instantiate directly.

**7. Common mistakes**

- Using @Value for 20 related properties.
- Forgetting getters/setters for binding.
- Spelling prefixes incorrectly.
- Assuming @Value validates types automatically.

**8. Interview difficulty**

**Beginner**

---

### Question 14: How do `@Profile` and `@Conditional` work?

**1. Interview-ready answer**

`@Profile` activates beans only when a named profile is active. `@Conditional` is a lower-level mechanism that evaluates a `Condition` at bean-definition registration time; Spring Boot adds conveniences such as `@ConditionalOnProperty`, `@ConditionalOnClass`, and `@ConditionalOnMissingBean`. Conditions are evaluated before bean instantiation.

**2. Detailed explanation**

Common conditions:

| Annotation | Activates when |
|---|---|
| `@Profile("dev")` | dev profile active |
| `@ConditionalOnProperty` | Property has expected value |
| `@ConditionalOnClass` | Class is on classpath |
| `@ConditionalOnMissingClass` | Class absent |
| `@ConditionalOnMissingBean` | No matching bean exists |
| `@ConditionalOnBean` | Matching bean exists |

**3. Internal working**

`ConditionEvaluator.shouldSkip` checks metadata during configuration class parsing. Because it runs before bean creation, condition methods must not rely on application beans being instantiated.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;

interface MessageSender {

    void send(String message);
}

@Configuration
class MessagingConfig {

    @Bean
    @Profile("dev")
    MessageSender logSender() {
        return System.out::println;
    }

    @Bean
    @ConditionalOnProperty(name = "messaging.kafka.enabled", havingValue = "true")
    MessageSender kafkaSender() {
        return message -> System.out.println("kafka:" + message);
    }
}
```

Activate with `--spring.profiles.active=dev` or set `messaging.kafka.enabled=true`.

**5. Real-world use case**

A service registers a real payment bean in production and an in-memory fake under the `test` profile, without changing business code.

**6. Interview follow-up questions**

- **Are conditions evaluated at runtime per request?** No, at startup/definition time.
- **Can @Profile be on a component class?** Yes.
- **Can condition order matter?** Yes, especially OnBean/OnMissingBean.
- **Why is OnMissingBean important in Boot?** It lets user beans override auto-configuration.
- **Can conditions inspect bean instances?** No, only definitions/metadata.

**7. Common mistakes**

- Using conditions inside business methods.
- Forgetting to activate the profile.
- Depending on beans inside custom Condition.
- Creating duplicate beans with overlapping conditions.

**8. Interview difficulty**

**Intermediate**

---

### Question 15: What are the core concepts of Spring AOP?

**1. Interview-ready answer**

Spring AOP adds cross-cutting behavior — logging, metrics, security, transactions — without putting it in every business method. Key terms: join point (a method execution), pointcut (expression selecting join points), advice (code to run), aspect (pointcut + advice), target (advised object), and proxy (the wrapper the caller actually invokes).

**2. Detailed explanation**

| Term | Meaning |
|---|---|
| Join point | Candidate method execution |
| Pointcut | Predicate selecting join points |
| Advice | Action: before/after/around |
| Aspect | Module combining pointcuts and advice |
| Target | Original bean |
| Proxy | Bean exposed to callers |
| Weaving | Linking advice to target; Spring AOP uses runtime proxies |

Spring AOP is proxy-based and advises public method calls on Spring-managed beans. It does not instrument arbitrary object construction or private method calls.

**3. Internal working**

`AnnotationAwareAspectJAutoProxyCreator` is a `BeanPostProcessor` that checks whether a bean matches any advisor and returns a proxy if needed. Calls go through an interceptor chain before reaching the target method.

**4. Practical Java/Spring Boot example**

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
class TimingAspect {

    @Around("execution(* com.example.orders..*Service.*(..))")
    Object measure(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.nanoTime();
        try {
            return joinPoint.proceed();
        } finally {
            System.out.println(joinPoint.getSignature().getName()
                    + " took " + (System.nanoTime() - start) + " ns");
        }
    }
}
```

**5. Real-world use case**

A payment service measures every service method and emits latency metrics without adding timer code to each method.

**6. Interview follow-up questions**

- **Does Spring AOP use AspectJ weaving?** By default, no; it uses proxies with AspectJ pointcut syntax.
- **Can it advise final methods?** No with CGLIB; JDK proxies only advise interface methods.
- **Can it advise private methods?** No.
- **What is an advisor?** A pointcut plus one advice.
- **How do you enable AOP?** `spring-boot-starter-aop` and `@EnableAspectJAutoProxy` outside Boot.

**7. Common mistakes**

- Expecting aspect to run on self-invocation.
- Applying broad pointcuts to every method.
- Swallowing exceptions in `@Around`.
- Forgetting to return `proceed()` result.

**8. Interview difficulty**

**Intermediate**

---

---

## Part D — Proxies, events, and coding

### Question 16: Explain JDK dynamic proxies vs CGLIB proxies and self-invocation

**1. Interview-ready answer**

JDK dynamic proxies implement the target's interfaces and require interface-based calls. CGLIB creates a subclass and can proxy classes without interfaces, but cannot proxy final classes or final methods. Spring Boot uses CGLIB-style class proxies by default. Self-invocation — calling one method on `this` from another method of the same bean — bypasses the proxy, so AOP advice such as `@Transactional` does not run.

**2. Detailed explanation**

| Proxy | Requires | Can proxy final class | Spring Boot default |
|---|---|---|---|
| JDK dynamic | Interfaces | n/a | No |
| CGLIB subclass | Non-final class/method | No | Yes |

Fix self-invocation by:

1. Moving the method into another bean.
2. Calling through the injected proxy.
3. Using `AopContext.currentProxy()` with `exposeProxy = true` (last resort).
4. Using AspectJ weaving instead of proxy AOP.

**3. Internal working**

A JDK proxy's invocation handler forwards calls through the advisor chain before reflecting on the target. A CGLIB proxy overrides methods in a generated subclass. In both cases, advice runs only when the call enters through the proxy object.

**4. Practical Java/Spring Boot example**

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;

@Aspect
@Component
class CallAspect {

    @Before("execution(* com.example.orders.OrderService.applyDiscount(..))")
    void beforeDiscount() {
        System.out.println("advice");
    }
}

@Service
class OrderService {

    public void placeOrder(String orderId) {
        applyDiscount(orderId);
    }

    public void applyDiscount(String orderId) {
        System.out.println("discount " + orderId);
    }
}
```

Calling `placeOrder` prints only `discount ORD-1`; the advice is skipped because `applyDiscount` was called on `this`.

**5. Real-world use case**

A service calls its own `@Transactional` method and expects a new transaction; no new transaction starts because the proxy is bypassed.

**6. Interview follow-up questions**

- **Which proxy type for a class with no interface?** CGLIB.
- **Can CGLIB proxy final methods?** No.
- **How do you force JDK proxies?** `spring.aop.proxy-target-class=false`.
- **Does @Async self-invocation work?** No, same reason.
- **How do you detect proxy type?** `AopUtils.isCglibProxy(bean)` / `isJdkDynamicProxy(bean)`.

**7. Common mistakes**

- Expecting advice on private/self-called methods.
- Making proxied service methods final.
- Injecting the target instead of the proxy.
- Using AopContext without exposing the proxy.

**8. Interview difficulty**

**Intermediate**

---

### Question 17: What are the advice types and how is advice ordering controlled?

**1. Interview-ready answer**

`@Before` runs before the method, `@After` runs after any outcome, `@AfterReturning` runs only on success, `@AfterThrowing` only on exception, and `@Around` wraps the call and can alter arguments, result, or exceptions. Ordering uses `@Order` or `Ordered`; lower values run first for before advice and typically last for after advice.

**2. Detailed explanation**

Execution order for successful Around + Before:

```text
@Around enter
@Before
target method
@AfterReturning
@After
@Around exit
```

On exception, `@AfterThrowing` replaces `@AfterReturning`.

**3. Internal working**

Each matching advisor contributes an interceptor to a chain. `@Around` is the most powerful because it calls `ProceedingJoinPoint.proceed()`. If it does not call proceed, the target never executes.

**4. Practical Java/Spring Boot example**

```java
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Aspect
@Component
@Order(1)
class SecurityAspect {

    @Before("execution(* com.example.orders.*Service.*(..))")
    void checkAccess() {
        System.out.println("security");
    }
}

@Aspect
@Component
@Order(2)
class MetricsAspect {

    @AfterReturning("execution(* com.example.orders.*Service.*(..))")
    void recordSuccess() {
        System.out.println("metrics");
    }

    @After("execution(* com.example.orders.*Service.*(..))")
    void cleanup() {
        System.out.println("cleanup");
    }
}
```

Security advice runs before metrics advice because of `@Order(1)` vs `@Order(2)`.

**5. Real-world use case**

Security advice must run before business validation, while metrics advice should record the final outcome after execution.

**6. Interview follow-up questions**

- **Which advice can suppress an exception?** `@Around` and `@AfterThrowing` by handling it.
- **Does @After see the return value?** No.
- **What happens if Around skips proceed?** Target method does not run.
- **How does @Order work with two @Before?** Lower value executes first.
- **Can advice change arguments?** Only `@Around`.

**7. Common mistakes**

- Forgetting `proceed()`.
- Broad pointcuts matching infrastructure beans.
- Relying on undefined ordering without @Order.
- Swallowing exceptions silently.

**8. Interview difficulty**

**Intermediate**

---

### Question 18: How do application events work in Spring?

**1. Interview-ready answer**

Spring's event mechanism lets one bean publish an event with `ApplicationEventPublisher` while listeners react with `@EventListener`. Events are synchronous by default and run in the publisher's thread; add `@Async` for asynchronous listeners or `@TransactionalEventListener` to react after a transaction phase. Events decouple producers from consumers and help break dependency cycles.

**2. Detailed explanation**

| Annotation | Purpose |
|---|---|
| `@EventListener` | Listen for any application event |
| `@TransactionalEventListener` | Listen at a transaction phase |
| `@Order` | Listener ordering |
| `@Async` | Run listener asynchronously |
| `@ConditionalOn...` | Conditional listeners in Boot |

A listener can declare the specific event type as its parameter. Spring finds listeners through `ApplicationListener` detection and `EventListenerMethodProcessor`.

**3. Internal working**

`publishEvent` delegates to `ApplicationEventMulticaster`, which invokes matching listeners. By default, exceptions in listeners propagate to the publisher unless the listener handles them.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;

record OrderCreatedEvent(String orderId) {
}

@Service
class OrderService {

    private final ApplicationEventPublisher publisher;

    OrderService(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    void create(String orderId) {
        publisher.publishEvent(new OrderCreatedEvent(orderId));
    }
}

@Component
class OrderAuditListener {

    @EventListener
    void onOrderCreated(OrderCreatedEvent event) {
        System.out.println("audit " + event.orderId());
    }
}
```

Publishing `ORD-1` prints `audit ORD-1`.

**5. Real-world use case**

After an order is created, listeners send confirmation email, update search index, and publish a Kafka event without the order service depending on all consumers.

**6. Interview follow-up questions**

- **Are events synchronous?** By default, yes.
- **Can a listener start its own transaction?** Yes with `@Transactional`.
- **What does @TransactionalEventListener do by default?** Listens after commit.
- **What if listener throws?** It can fail the publisher unless handled.
- **How do you order listeners?** `@Order` or `Ordered`.

**7. Common mistakes**

- Treating events as durable messaging.
- Using events when immediate consistency is required.
- Forgetting async executor configuration for `@Async` listeners.
- Publishing before the entity is flushed when listener needs it.

**8. Interview difficulty**

**Intermediate**

---

### Question 19: What is a `FactoryBean` in Spring?

**1. Interview-ready answer**

A `FactoryBean<T>` is a bean that produces another object. Callers injecting `T` receive the produced object; prefixing the bean name with `&` retrieves the factory itself. It is useful for complex third-party object construction where a normal `@Bean` method is not flexible enough.

**2. Detailed explanation**

Methods:

- `T getObject()` — creates the product.
- `Class<?> getObjectType()` — declares the product type.
- `boolean isSingleton()` — whether the product is cached.

Spring itself uses FactoryBeans internally for proxies, JPA EntityManagerFactory, and other infrastructure objects.

**3. Internal working**

During `getBean("client")`, Spring sees that the definition is a FactoryBean. If the name has no `&`, it calls `getObject`; with `&client`, it returns the FactoryBean instance. Singleton products are cached by the factory.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.beans.factory.FactoryBean;
import org.springframework.stereotype.Component;

class ApiClient {

    ApiClient(String endpoint) {
    }
}

@Component("apiClient")
class ApiClientFactoryBean implements FactoryBean<ApiClient> {

    @Override
    public ApiClient getObject() {
        return new ApiClient("https://api.example.com");
    }

    @Override
    public Class<?> getObjectType() {
        return ApiClient.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

Injecting `ApiClient` gives the produced client; injecting `@Qualifier("&apiClient") ApiClientFactoryBean` would be a type mismatch, so use `BeanFactory.getBean("&apiClient")` to retrieve the factory.

**5. Real-world use case**

A library creates clients whose configuration depends on runtime metadata and exposes them through FactoryBeans.

**6. Interview follow-up questions**

- **How do you get the FactoryBean itself?** Prefix bean name with `&`.
- **Is FactoryBean the same as BeanFactory?** No; opposite roles.
- **When prefer @Bean?** Most application cases.
- **Can getObject return null?** Avoid it; resolution may fail.
- **Does isSingleton false mean new object each get?** Yes.

**7. Common mistakes**

- Confusing FactoryBean with BeanFactory.
- Forgetting `getObjectType`.
- Returning prototypes while claiming singleton.
- Using FactoryBean where @Bean is simpler.

**8. Interview difficulty**

**Advanced**

---

### Question 20: Coding challenge — implement an audit aspect with `@Around`

**1. Interview-ready answer**

I define a custom `@Audit` annotation, apply it to service methods, and write an `@Around` aspect that records method name, arguments, outcome, and duration. The advice calls `proceed()` in a try/finally block so the business method always runs and timing is always recorded, while exceptions are rethrown unchanged.

**2. Detailed explanation**

**Problem statement:** Create reusable audit logging for selected service methods without modifying their bodies.

**Approach:**
1. Define `@Audit(action)` with `@Target(METHOD)` and `@Retention(RUNTIME)`.
2. Create an `@Aspect` with `@Around("@annotation(audit)")`.
3. Log entry, call `proceed`, log success/failure, and log duration in `finally`.
4. Annotate a service method.

**3. Internal working**

The annotation is read from `MethodSignature.getMethod()`. The proxy invokes the advice chain before the target. Because the advice wraps the call, it can observe both normal return and exception paths.

**4. Practical Java solution**

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.Arrays;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@interface Audit {

    String action();
}

@Aspect
@Component
class AuditAspect {

    @Around("@annotation(audit)")
    Object audit(ProceedingJoinPoint joinPoint, Audit audit) throws Throwable {
        String method = joinPoint.getSignature().toShortString();
        System.out.println("start " + audit.action() + " " + method
                + " args=" + Arrays.toString(joinPoint.getArgs()));

        long start = System.nanoTime();
        try {
            Object result = joinPoint.proceed();
            System.out.println("success " + audit.action());
            return result;
        } catch (Throwable failure) {
            System.out.println("failure " + audit.action() + " " + failure.getMessage());
            throw failure;
        } finally {
            System.out.println("durationNs=" + (System.nanoTime() - start));
        }
    }
}

@Service
class CheckoutService {

    @Audit(action = "ORDER_CHECKOUT")
    public String checkout(String orderId) {
        return "confirmed:" + orderId;
    }
}
```

Calling `checkout("ORD-1")` prints:

```text
start ORDER_CHECKOUT CheckoutService.checkout(String) args=[ORD-1]
success ORDER_CHECKOUT
durationNs=...
```

**5. Dry run**

| Step | Action | Result |
|---|---|---|
| 1 | Proxy receives checkout call | Aspect matched by @Audit |
| 2 | Advice logs start | audit entry emitted |
| 3 | `proceed()` | target returns `confirmed:ORD-1` |
| 4 | Advice logs success and duration | caller receives result |

**6. Complexity**

- Time: O(1) advice overhead plus target execution.
- Space: O(1) besides log output.

**7. Edge cases**

- Exception from target → failure logged and rethrown.
- Large arguments → cap or redact logged values.
- Self-invocation → advice is skipped.
- Async method → advice sees Future/CompletableFuture return, not eventual result.
- Sensitive data → never log passwords/tokens.

**8. Real-world use case**

A Spring service audits checkout, refund, and payment capture calls with one aspect instead of duplicating logging in every method.

**9. Interview follow-up questions**

- **Why @Around instead of @Before/@After?** It observes return, exception, and duration in one advice.
- **What happens if proceed is omitted?** Target method never executes.
- **How do you avoid logging secrets?** Mask arguments by type/name.
- **Why does self-invocation skip this?** The call does not pass through the proxy.
- **How do you send audit events asynchronously?** Publish an event inside the aspect.

**10. Common mistakes**

- Swallowing exceptions instead of rethrowing.
- Logging full request DTOs with PII.
- Forgetting `@EnableAspectJAutoProxy` outside Boot.
- Applying the aspect to too many methods.
- Reading annotations from the proxy class instead of target method.

**11. Interview difficulty**

**Intermediate**

---

## Chapter 9 Verification

| Syllabus topic | Question(s) |
|---|---|
| IoC and dependency injection | Q1, Q3, Q4 |
| BeanFactory vs ApplicationContext | Q2 |
| Bean scopes | Q5 |
| Bean lifecycle and callbacks | Q6 |
| BeanPostProcessor vs BeanFactoryPostProcessor | Q7 |
| Component scanning and stereotypes | Q8 |
| @Bean vs @Component | Q9 |
| @Configuration full/lite modes | Q10 |
| Circular dependencies | Q11 |
| @Lazy, ObjectProvider, Optional | Q12 |
| Environment and configuration binding | Q13 |
| @Conditional and profiles | Q14 |
| Spring AOP concepts | Q15, Q17, Q20 |
| JDK vs CGLIB proxies and self-invocation | Q16 |
| Application events | Q18 |
| FactoryBean | Q19 |

**Total interview questions: 20** — all with the required 8-part format; coding questions include problem statement, approach, Java solution, dry run, complexity, and edge cases.

**Next:** [Chapter 10 — Spring Boot](10-Spring-Boot.md). Say `NEXT CHAPTER` to continue.
