# Chapter 10: Spring Boot

**Chapter goal:** Explain Spring Boot 3 auto-configuration, starters, startup, embedded servers, externalized configuration, Actuator, logging, graceful shutdown, fat jars, and production-ready configuration.

**Topics covered:** Spring Boot overview · `@SpringBootApplication` · auto-configuration internals · starters · startup lifecycle · embedded servers · properties vs YAML · configuration precedence · profiles · `@ConfigurationProperties` · Actuator · logging · environment variables · graceful shutdown · Spring Boot 3 changes · executable jars · conditional auto-configuration · testing slices · production config · custom auto-configuration.

**Cross-references:** Core IoC and AOP are in [Chapter 9](09-Spring-Framework.md). REST APIs are in [Chapter 11](11-REST-API-Development.md). Transactions and scheduling are in [Chapter 12](12-Transactions-and-Scheduling.md).

---

## Part A — Boot foundations

### Question 1: What is Spring Boot and what problems does it solve?

**1. Interview-ready answer**

Spring Boot is an opinionated layer on top of Spring that enables standalone, production-ready applications with minimal configuration. It solves dependency management through starters, wires beans automatically based on the classpath via auto-configuration, embeds a server, externalizes configuration, and provides Actuator for operations.

**2. Detailed explanation**

Problems solved:

- No manual XML/Java wiring for common infrastructure.
- No external WAR deployment requirement.
- Consistent dependency versions via the parent BOM.
- Sensible defaults that can be overridden.
- Production endpoints for health, metrics, and info.

**3. Internal working**

Boot reads the classpath, evaluates auto-configuration conditions, creates missing infrastructure beans, and starts an embedded server. Application properties can override defaults before and during context refresh.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
class OrdersApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrdersApplication.class, args);
    }

    @GetMapping("/health-check")
    String health() {
        return "UP";
    }
}
```

Running the class starts a web server on port 8080 with no external container.

**5. Real-world use case**

A team ships a `java -jar orders-service.jar` artifact with Tomcat embedded, instead of deploying a WAR to a separately managed application server.

**6. Interview follow-up questions**

- **Is Spring Boot different from Spring?** Boot builds on Spring; it is not a replacement.
- **Does Boot generate code?** No; it registers beans at runtime.
- **Can you disable auto-configuration?** Yes with `exclude` or `excludeName`.
- **Does Boot require the parent POM?** No, but the BOM is strongly recommended.
- **Can Boot build non-web apps?** Yes.

**7. Common mistakes**

- Treating Boot as a framework separate from Spring.
- Fighting Boot defaults without understanding conditions.
- Adding dependencies whose versions conflict with the BOM.
- Using Boot for a tiny library where plain Spring is enough.

**8. Interview difficulty**

**Beginner**

---

### Question 2: What does `@SpringBootApplication` include?

**1. Interview-ready answer**

`@SpringBootApplication` is a composed annotation containing `@SpringBootConfiguration`, `@EnableAutoConfiguration`, and `@ComponentScan` with type-based exclude filters. In practice it marks the configuration class, enables auto-configuration, and scans the main class's package and subpackages for components.

**2. Detailed explanation**

| Meta-annotation | Purpose |
|---|---|
| `@SpringBootConfiguration` | Specialized `@Configuration` |
| `@EnableAutoConfiguration` | Imports auto-configuration candidates |
| `@ComponentScan` | Scans components from the main package |

The default scan starts at the package of the class annotated with `@SpringBootApplication`. Placing it too deep leaves beans undiscovered; placing it too high scans unrelated packages.

**3. Internal working**

`@EnableAutoConfiguration` imports `AutoConfigurationImportSelector`, which reads candidate classes from `AutoConfiguration.imports`, filters them by conditions, and registers eligible configuration classes.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;

@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class OrdersApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrdersApplication.class, args);
    }
}
```

This disables JDBC datasource auto-configuration when the service does not need a database.

**5. Real-world use case**

A pure messaging microservice excludes `DataSourceAutoConfiguration` to avoid failing startup when no JDBC URL is configured.

**6. Interview follow-up questions**

- **Can you use multiple @SpringBootApplication classes?** Not normally in one app context.
- **How do you scan another package?** `scanBasePackages` or `@ComponentScan`.
- **What is the difference from `@Configuration`?** Boot adds auto-configuration and scanning.
- **How do you exclude by class name?** `excludeName`.
- **Does it enable AOP?** Only if AOP starter/classes are present.

**7. Common mistakes**

- Moving the main class into a deep package.
- Excluding auto-configuration blindly.
- Duplicating scan annotations.
- Assuming `@SpringBootApplication` imports every dependency automatically.

**8. Interview difficulty**

**Beginner**

---

### Question 3: How does Spring Boot auto-configuration work internally?

**1. Interview-ready answer**

Auto-configuration is conditional configuration loaded at startup. Boot reads candidate classes from `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`, applies conditions such as `@ConditionalOnClass`, `@ConditionalOnMissingBean`, and `@ConditionalOnProperty`, then registers only the configurations whose conditions pass. User-defined beans usually win because `@ConditionalOnMissingBean` backs off.

**2. Detailed explanation**

Boot 3 changes:

- Auto-configuration classes are listed in `AutoConfiguration.imports`.
- `spring.factories` is still used for listeners, initializers, and other extension points, but not for auto-configuration imports.
- `@AutoConfiguration` replaces plain `@Configuration` for auto-config classes.

Key conditions:

- `@ConditionalOnClass` — class is present.
- `@ConditionalOnMissingClass` — class absent.
- `@ConditionalOnBean` / `@ConditionalOnMissingBean` — bean presence.
- `@ConditionalOnProperty` — property matches.
- `@ConditionalOnWebApplication` — web app type.

**3. Internal working**

`AutoConfigurationImportSelector` loads imports, sorts them using `@AutoConfigureOrder`, `@AutoConfigureBefore`, and `@AutoConfigureAfter`, removes duplicates, then evaluates conditions via the Spring `ConditionEvaluator`. Conditions run before bean instantiation.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.boot.autoconfigure.AutoConfiguration;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;

@AutoConfiguration
@ConditionalOnProperty(name = "audit.enabled", havingValue = "true")
class AuditAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    AuditClient auditClient() {
        return new ConsoleAuditClient();
    }
}
```

The bean exists only when `audit.enabled=true` and no user-provided `AuditClient` exists.

**5. Real-world use case**

Adding `spring-boot-starter-data-redis` puts Redis classes on the classpath, so Boot auto-configures a `RedisTemplate` unless the application defines its own.

**6. Interview follow-up questions**

- **How do you debug auto-configuration?** Run with `--debug` or `DEBUG` condition evaluation report.
- **How do you override an auto-configured bean?** Define your own bean; `@ConditionalOnMissingBean` backs off.
- **When are conditions evaluated?** During configuration parsing, before bean creation.
- **Why @AutoConfiguration instead of @Configuration?** It is optimized and ordered for auto-config.
- **Can auto-configuration depend on a bean instance?** Avoid it; use definitions/metadata.

**7. Common mistakes**

- Using `spring.factories` for Boot 3 auto-config imports.
- Ignoring condition reports.
- Defining beans with ambiguous types that block backoff.
- Forgetting `@ConditionalOnMissingBean`.

**8. Interview difficulty**

**Advanced**

---

### Question 4: How do Spring Boot starters work?

**1. Interview-ready answer**

A starter is a curated POM that groups dependencies for a capability, such as web, JPA, or security. It contains little or no code; it brings libraries with compatible versions, and those libraries trigger auto-configuration through the classpath. Starters simplify dependency management but do not replace understanding what is included.

**2. Detailed explanation**

Common starters:

| Starter | Brings |
|---|---|
| `spring-boot-starter-web` | Spring MVC, Jackson, Tomcat |
| `spring-boot-starter-data-jpa` | JPA, Hibernate, Spring Data |
| `spring-boot-starter-security` | Spring Security |
| `spring-boot-starter-test` | JUnit, Mockito, AssertJ, Testcontainers support |
| `spring-boot-starter-actuator` | Production endpoints |

**3. Internal working**

The Boot parent/BOM manages versions. Starter POMs declare transitive dependencies. Auto-configuration classes check for those dependency classes and activate.

**4. Practical Java/Spring Boot example**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Adding `starter-web` triggers `WebMvcAutoConfiguration` and embedded Tomcat configuration.

**5. Real-world use case**

A service adds `spring-boot-starter-webflux` instead of `starter-web` to build a non-blocking reactive API with Netty.

**6. Interview follow-up questions**

- **Do starters contain auto-configuration code?** Usually no; they are dependency descriptors.
- **Can you use individual dependencies instead?** Yes, but you lose convenience.
- **Why not include all starters?** Classpath bloat and conflicting auto-configuration.
- **What does starter-test include?** JUnit 5, Mockito, AssertJ, Spring Test, Hamcrest, JSONassert.
- **How do you see the dependency tree?** `mvn dependency:tree`.

**7. Common mistakes**

- Adding both web and webflux starters accidentally.
- Duplicate versions overriding the BOM.
- Assuming a starter guarantees a specific implementation.
- Including test starters in production scope.

**8. Interview difficulty**

**Beginner**

---

### Question 5: Describe the Spring Boot startup lifecycle

**1. Interview-ready answer**

`SpringApplication.run` creates a `SpringApplication`, infers application type, loads initializers and listeners, prepares the `Environment`, creates the appropriate `ApplicationContext`, refreshes it, calls `ApplicationRunner`/`CommandLineRunner` beans, and publishes readiness events. Embedded servers start during context refresh.

**2. Detailed explanation**

Main phases:

1. Create `SpringApplication` and determine web type: SERVLET, REACTIVE, or NONE.
2. Load `ApplicationContextInitializer`s and listeners.
3. Prepare `Environment` with properties, profiles, and command-line args.
4. Print banner.
5. Create bean factory/context.
6. Refresh context: bean definitions, beans, post-processors, embedded server.
7. Call runners.
8. Publish `ApplicationReadyEvent`.

**3. Internal working**

`SpringApplicationRefreshApplication` and `ApplicationContextInitializer` hooks customize the context before refresh. `ApplicationRunner` and `CommandLineRunner` execute after the context is ready but before the application is considered fully ready in some lifecycle views; `ApplicationReadyEvent` is the usual signal that the app can accept traffic.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationListener;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.stereotype.Component;

@SpringBootApplication
public class OrdersApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrdersApplication.class, args);
    }
}

@Component
class WarmupRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) {
        System.out.println("warming cache");
    }
}

@Component
class ReadyLogger implements ApplicationListener<ApplicationReadyEvent> {

    @Override
    public void onApplicationEvent(ApplicationReadyEvent event) {
        System.out.println("application ready");
    }
}
```

Output:

```text
warming cache
application ready
```

**5. Real-world use case**

A service warms a local price cache in an `ApplicationRunner` before marking itself ready for Kubernetes traffic.

**6. Interview follow-up questions**

- **When does Tomcat start?** During context refresh, via `WebServerStartStopLifecycle`.
- **Runner vs ApplicationReadyEvent?** Runners are beans with simple hooks; events support listeners and ordering.
- **Can you access command-line args?** Yes via `ApplicationArguments`.
- **How do you fail startup fast?** Throw from a runner or validate configuration early.
- **What is an initializer?** A hook that customizes the context before refresh.

**7. Common mistakes**

- Doing long blocking work in constructors.
- Starting background threads before context readiness.
- Confusing runner order with readiness.
- Reading properties before environment is prepared.

**8. Interview difficulty**

**Intermediate**

---

---

## Part B — Configuration and embedded runtime

### Question 6: How do embedded servers work in Spring Boot?

**1. Interview-ready answer**

A servlet-based Boot app includes an embedded Tomcat by default through `spring-boot-starter-web`. During context refresh, Boot creates a `WebServer` and connector, registers the `DispatcherServlet`, and starts the server on `server.port`. You can switch to Jetty or Undertow by excluding Tomcat and adding the desired starter; WebFlux uses Netty by default.

**2. Detailed explanation**

| Web type | Default server | Starter |
|---|---|---|
| Servlet MVC | Tomcat | `spring-boot-starter-web` |
| Servlet Jetty | Jetty | `spring-boot-starter-jetty` |
| Servlet Undertow | Undertow | `spring-boot-starter-undertow` |
| Reactive WebFlux | Netty | `spring-boot-starter-webflux` |

Common settings: `server.port`, `server.address`, `server.tomcat.threads.max`, `server.tomcat.accept-count`, `server.servlet.context-path`, SSL properties.

**3. Internal working**

`ServletWebServerApplicationContext` creates the web server during `onRefresh`. Auto-configurations such as `ServletWebServerFactoryAutoConfiguration` and `DispatcherServletAutoConfiguration` provide the factory and servlet registration.

**4. Practical Java/Spring Boot example**

```yaml
server:
  port: 8081
  servlet:
    context-path: /orders
  tomcat:
    threads:
      max: 200
      min-spare: 20
    accept-count: 100
```

Switch to Jetty:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

**5. Real-world use case**

A high-concurrency service switches from Tomcat to Undertow after load testing shows better memory behavior for its workload.

**6. Interview follow-up questions**

- **How do you run on a random port?** `server.port=0`.
- **How do you disable the web server?** Set WebApplicationType.NONE or remove web starter.
- **Where is DispatcherServlet registered?** By Boot auto-configuration.
- **How do you configure SSL?** `server.ssl.*` properties.
- **Can two connectors be configured?** Yes with a custom `WebServerFactoryCustomizer`.

**7. Common mistakes**

- Adding two servlet server starters.
- Tuning Tomcat threads without measuring.
- Forgetting context path in health checks.
- Using MVC annotations in a pure WebFlux app.

**8. Interview difficulty**

**Intermediate**

---

### Question 7: What is the difference between `application.properties` and `application.yml`?

**1. Interview-ready answer**

Both define external configuration and are loaded by Spring Boot. `.properties` uses flat `key=value` lines; YAML uses structured indentation and supports lists and multi-document profiles more readably. YAML is converted into the same property sources, so there is no behavioral advantage beyond readability.

**2. Detailed explanation**

| Aspect | properties | YAML |
|---|---|---|
| Syntax | Flat keys | Hierarchical indentation |
| Lists | Indexed keys | Native list syntax |
| Multi-document profiles | Separate files | `---` documents |
| Tab indentation | N/A | Not allowed |
| Runtime behavior | Same | Same after flattening |

**3. Internal working**

Boot uses `PropertiesPropertySourceLoader` and `YamlPropertySourceLoader`. YAML is flattened into dot-separated keys such as `orders.regions[0]=us-east-1`.

**4. Practical Java/Spring Boot example**

Properties:

```properties
orders.timeout-seconds=30
orders.regions[0]=us-east-1
orders.regions[1]=eu-west-1
```

YAML:

```yaml
orders:
  timeout-seconds: 30
  regions:
    - us-east-1
    - eu-west-1
```

Both bind to the same `OrderProperties` object.

**5. Real-world use case**

Teams use YAML for readable environment configuration and `.properties` for small overrides or when tooling expects flat keys.

**6. Interview follow-up questions**

- **Can both files exist?** Yes; properties files take precedence when both are found in the same location.
- **Are YAML anchors supported?** SnakeYAML supports them, but they are rarely recommended.
- **Does YAML support placeholders?** Yes, `${...}` works.
- **Why do tabs break YAML?** Indentation is syntactically significant.
- **Can profiles be in one YAML file?** Yes with `---` and `spring.config.activate.on-profile`.

**7. Common mistakes**

- Mixing tabs and spaces.
- Assuming YAML supports types Spring will always preserve perfectly.
- Duplicate keys in YAML.
- Putting secrets in committed YAML.

**8. Interview difficulty**

**Beginner**

---

### Question 8: What is the Spring Boot configuration property precedence?

**1. Interview-ready answer**

Boot uses layered property sources; later/higher-priority sources override earlier ones. Command-line arguments outrank environment variables, which outrank application properties inside the jar, which outrank default properties. Profile-specific files override non-profile files for the active profile.

**2. Detailed explanation**

Simplified order, highest first:

1. Command-line arguments.
2. `SPRING_APPLICATION_JSON`.
3. Servlet init parameters and context parameters.
4. Java system properties (`-D`).
5. OS environment variables.
6. Profile-specific config outside the jar.
7. Profile-specific config inside the jar.
8. Non-profile config outside the jar.
9. Non-profile config inside the jar.
10. `@PropertySource` annotations.
11. `SpringApplication.setDefaultProperties`.

**3. Internal working**

All sources are added to the `Environment` in order. `PropertySourcesPropertyResolver` reads the first matching source. `@Value` and `@ConfigurationProperties` both resolve through this environment abstraction.

**4. Practical Java/Spring Boot example**

```bash
java -jar orders.jar \
  --server.port=9090 \
  --orders.timeout-seconds=15
```

These command-line values override `application.yml` and environment variables.

Environment variable equivalent:

```bash
export SERVER_PORT=9090
export ORDERS_TIMEOUT_SECONDS=15
```

**5. Real-world use case**

A Docker image has safe defaults in `application.yml`; Kubernetes overrides them with environment variables, and an operator can override again with command-line args.

**6. Interview follow-up questions**

- **Which wins: env var or application.yml?** Environment variable.
- **How are env var names mapped?** Uppercase with underscores.
- **Can you add a custom property source?** Yes via `EnvironmentPostProcessor`.
- **Do profile properties override command-line args?** No.
- **How do you inspect the final value?** Use Actuator `/env` when secured, or logs/debug.

**7. Common mistakes**

- Guessing precedence instead of checking.
- Putting secrets in inside-jar config.
- Using inconsistent naming.
- Exposing Actuator `/env` publicly.

**8. Interview difficulty**

**Intermediate**

---

### Question 9: How do Spring profiles and profile groups work?

**1. Interview-ready answer**

Profiles let you activate different bean definitions and property files for environments such as dev, test, and prod. Set them with `spring.profiles.active`, environment variables, or command-line flags. Profile groups, added in Boot 2.4, let one profile activate several others, such as `prod` activating `production` and `metrics`.

**2. Detailed explanation**

Ways to activate:

- `spring.profiles.active=dev`
- `--spring.profiles.active=prod`
- `SPRING_PROFILES_ACTIVE=prod`
- `SpringApplication.setAdditionalProfiles`
- Profile groups in config

Profile-specific files:

- `application-dev.yml`
- `application-prod.properties`
- Multi-document YAML with `spring.config.activate.on-profile`

**3. Internal working**

`Environment` holds active profiles. `Profiles` predicates are evaluated during configuration parsing and property loading. `@Profile` beans are skipped when inactive.

**4. Practical Java/Spring Boot example**

```yaml
spring:
  profiles:
    active: prod
    group:
      prod:
        - database
        - monitoring

---
spring:
  config:
    activate:
      on-profile: database
orders:
  timeout-seconds: 30

---
spring:
  config:
    activate:
      on-profile: monitoring
management:
  endpoints:
    web:
      exposure:
        include: health,metrics
```

Activating `prod` also activates `database` and `monitoring`.

**5. Real-world use case**

A single deployment artifact uses profiles to switch between local H2, staging PostgreSQL, and production PostgreSQL without rebuilding.

**6. Interview follow-up questions**

- **Can multiple profiles be active?** Yes.
- **Can @Profile be on @Configuration classes?** Yes.
- **What replaced `spring.profiles` in YAML documents?** `spring.config.activate.on-profile`.
- **Can profile groups be nested?** Yes.
- **How do you default a profile?** `spring.profiles.default`.

**7. Common mistakes**

- Putting secrets in profile files committed to Git.
- Assuming inactive profile beans are merely disabled at runtime — they are not created.
- Mixing old and new profile activation syntax.
- Forgetting to activate the intended profile in tests.

**8. Interview difficulty**

**Beginner**

---

### Question 10: How do you use `@ConfigurationProperties` effectively?

**1. Interview-ready answer**

`@ConfigurationProperties` binds a typed prefix from the `Environment` into an object, with relaxed binding, conversion, nested objects, collections, and validation. In Boot 3, records work naturally with constructor binding. Register the class with `@EnableConfigurationProperties` or `@ConfigurationPropertiesScan`.

**2. Detailed explanation**

Benefits:

- Type safety and IDE completion with configuration metadata.
- Group related properties.
- Validation with Jakarta Bean Validation.
- Relaxed binding across YAML, properties, env vars, and system props.
- Easier testing by instantiating the properties object directly.

**3. Internal working**

`ConfigurationPropertiesBindingPostProcessor` uses `Binder` after the bean is instantiated. For constructor binding, it matches constructor parameters to properties by name. Metadata can be generated by `spring-boot-configuration-processor`.

**4. Practical Java/Spring Boot example**

```java
import java.time.Duration;
import java.util.List;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.validation.annotation.Validated;

@Validated
@ConfigurationProperties(prefix = "orders")
public record OrderProperties(
        @NotNull Duration timeout,
        @NotBlank String currency,
        List<String> regions) {
}

@Configuration
@EnableConfigurationProperties(OrderProperties.class)
class PropertiesConfig {
}
```

Configuration:

```yaml
orders:
  timeout: 30s
  currency: INR
  regions:
    - us-east-1
    - eu-west-1
```

**5. Real-world use case**

A service binds all Kafka consumer settings into one validated record and logs it at startup with secrets masked.

**6. Interview follow-up questions**

- **How does `30s` become Duration?** Boot's conversion service supports duration suffixes.
- **How do you bind a Map?** `Map<String, String> headers`.
- **Can you validate nested properties?** Yes with `@Valid` on nested objects.
- **How do you generate metadata?** Add `spring-boot-configuration-processor`.
- **Does relaxed binding work for env vars?** Yes.

**7. Common mistakes**

- Forgetting to register the properties class.
- Missing setters for non-record classes.
- Using primitive defaults that hide missing config.
- Logging raw properties containing passwords.

**8. Interview difficulty**

**Intermediate**

---

---

## Part C — Operations and production behavior

### Question 11: What is Spring Boot Actuator and how do you use it safely?

**1. Interview-ready answer**

Actuator exposes production endpoints such as health, info, metrics, beans, env, and HTTP traces. Add `spring-boot-starter-actuator`, expose only required endpoints over HTTP, secure them, and use `health` for liveness/readiness probes. Sensitive endpoints such as `env` and `heapdump` should never be public.

**2. Detailed explanation**

Common endpoints:

| Endpoint | Purpose |
|---|---|
| `/actuator/health` | Application health |
| `/actuator/info` | Build/git metadata |
| `/actuator/metrics` | Micrometer metrics |
| `/actuator/prometheus` | Prometheus scrape format |
| `/actuator/loggers` | Runtime log levels |
| `/actuator/env` | Property sources |
| `/actuator/beans` | Bean list |

By default only `health` is exposed over HTTP.

**3. Internal working**

Endpoints are `@Endpoint` beans exposed through web or JMX technology by `EndpointWebMvcAutoConfiguration`. `HealthIndicator` beans contribute to the aggregate `Health` result with statuses UP, DOWN, OUT_OF_SERVICE, or UNKNOWN.

**4. Practical Java/Spring Boot example**

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when_authorized
  health:
    diskspace:
      enabled: false
```

Custom health indicator:

```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
class PaymentGatewayHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        boolean reachable = checkGateway();
        return reachable
                ? Health.up().withDetail("latencyMs", 12).build()
                : Health.down().withDetail("error", "timeout").build();
    }

    private boolean checkGateway() {
        return true;
    }
}
```

**5. Real-world use case**

Kubernetes uses `/actuator/health/liveness` and `/actuator/health/readiness`, while Prometheus scrapes `/actuator/prometheus` from an internal network.

**6. Interview follow-up questions**

- **What is the difference between liveness and readiness?** Liveness restarts a broken app; readiness controls traffic.
- **How do you secure Actuator?** Spring Security, network policy, or separate management port.
- **How do you add custom metrics?** Micrometer `Counter`, `Timer`, `Gauge`.
- **Can you change log level at runtime?** Yes via `/actuator/loggers` if exposed and secured.
- **Is `/actuator/env` safe?** No; it can leak secrets.

**7. Common mistakes**

- Exposing all endpoints publicly.
- Using health for business validation.
- Returning UP even when dependencies are unusable.
- Putting secrets in `/info`.

**8. Interview difficulty**

**Intermediate**

---

### Question 12: How does logging work in Spring Boot?

**1. Interview-ready answer**

Spring Boot uses SLF4J as the facade and Logback as the default backend. It provides default console patterns, log levels, rotating file support, and profile-specific logging configuration. Application code should log through SLF4J loggers rather than `System.out`.

**2. Detailed explanation**

Default levels: ERROR, WARN, INFO, DEBUG, TRACE. Configure with:

- `logging.level.root`
- `logging.level.com.example.orders`
- `logging.file.name`
- `logging.pattern.console`
- `logback-spring.xml` for advanced setup

Boot also configures logging for Spring Framework, Hibernate, and Tomcat.

**3. Internal working**

`LoggingApplicationListener` initializes the logging system early during startup so framework logs use the same configuration. `logback-spring.xml` supports Spring profile tags such as `<springProfile>`.

**4. Practical Java/Spring Boot example**

```yaml
logging:
  level:
    root: INFO
    com.example.orders: DEBUG
    org.hibernate.SQL: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{36} - %msg%n"
```

Java usage:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

@Service
class PaymentService {

    private static final Logger log = LoggerFactory.getLogger(PaymentService.class);

    void charge(String paymentId) {
        log.info("charging payment {}", paymentId);
    }
}
```

**5. Real-world use case**

A production service logs JSON to stdout so a log shipper can parse fields, while local development uses colored console output.

**6. Interview follow-up questions**

- **Why use parameterized logging?** It avoids string concatenation when level is disabled.
- **Can you switch to Log4j2?** Yes by excluding Logback and adding a starter.
- **How do you change level at runtime?** `/actuator/loggers` or JMX.
- **Why avoid println?** No levels, no correlation, no configuration.
- **How do you add trace IDs?** Micrometer Tracing with MDC.

**7. Common mistakes**

- Logging passwords or tokens.
- DEBUG logging in production.
- Using string concatenation in hot paths.
- Creating multiple logging backends accidentally.

**8. Interview difficulty**

**Beginner**

---

### Question 13: How do environment variables and containers fit into Boot configuration?

**1. Interview-ready answer**

Boot maps OS environment variables into property sources using relaxed binding: `SPRING_DATASOURCE_URL` becomes `spring.datasource.url`. In Docker and Kubernetes, environment variables or mounted secret files commonly override application defaults. Secrets should come from a secret manager or mounted volume, not a committed config file.

**2. Detailed explanation**

Mapping examples:

| Property | Environment variable |
|---|---|
| `server.port` | `SERVER_PORT` |
| `orders.timeout-seconds` | `ORDERS_TIMEOUT_SECONDS` |
| `spring.datasource.url` | `SPRING_DATASOURCE_URL` |

Boot also reads `SPRING_APPLICATION_JSON` for structured external config.

**3. Internal working**

`SystemEnvironmentPropertySource` normalizes env var names to match relaxed property names. `ConfigurationPropertySourcesPropertySource` adapts the environment for binder lookups.

**4. Practical Java/Spring Boot example**

```bash
docker run --rm \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e SERVER_PORT=8080 \
  -e SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/orders \
  orders-service:1.0
```

Kubernetes manifest fragment:

```yaml
env:
  - name: SPRING_DATASOURCE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: orders-db
        key: password
```

**5. Real-world use case**

The same immutable container image runs in staging and production with only environment variables and secrets changed.

**6. Interview follow-up questions**

- **Do hyphens become underscores?** Yes in env var names.
- **Can env vars override YAML?** Yes.
- **How do you pass complex lists?** Indexed env vars or JSON via `SPRING_APPLICATION_JSON`.
- **How do you mount secrets as files?** Kubernetes Secret volume; reference with `file:` or property path.
- **Why not commit prod passwords?** Security and audit risk.

**7. Common mistakes**

- Using wrong env var casing.
- Expecting dots in env var names.
- Logging the resolved datasource password.
- Baking environment-specific config into the image.

**8. Interview difficulty**

**Intermediate**

---

### Question 14: How do you implement graceful shutdown in Spring Boot?

**1. Interview-ready answer**

Set `server.shutdown=graceful` and configure `spring.lifecycle.timeout-per-shutdown-phase`. On SIGTERM, the embedded server stops accepting new requests, waits for in-flight requests to finish, and then closes the application context and beans. Kubernetes should allow enough `terminationGracePeriodSeconds`.

**2. Detailed explanation**

Shutdown sequence:

1. Kubernetes sends SIGTERM.
2. Boot stops accepting new traffic.
3. Web server waits for active requests up to the timeout.
4. SmartLifecycle beans stop in reverse order.
5. `@PreDestroy` / `DisposableBean` callbacks run.
6. Context closes and process exits.

**3. Internal working**

`GracefulShutdown` tracks active requests using `Connection: close` responses and latches. Bean destruction is managed by `ConfigurableApplicationContext.close`.

**4. Practical Java/Spring Boot example**

```yaml
server:
  shutdown: graceful

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

Kubernetes:

```yaml
terminationGracePeriodSeconds: 45
```

**5. Real-world use case**

During a rolling deployment, an in-flight payment capture finishes successfully instead of being cut off mid-request.

**6. Interview follow-up questions**

- **What happens if timeout expires?** Remaining requests are aborted and context closes.
- **How do readers stop traffic before shutdown?** Readiness probe fails first.
- **Does graceful shutdown wait for async tasks?** Only if they are managed by Spring lifecycle or tracked explicitly.
- **How do you close a custom executor?** Implement `SmartLifecycle` or register destroy method.
- **What signal does Kubernetes send first?** SIGTERM, then SIGKILL after grace period.

**7. Common mistakes**

- Setting grace period shorter than request timeout.
- Closing executors manually too early.
- Ignoring readiness probes.
- Assuming graceful shutdown guarantees messaging consumers finish.

**8. Interview difficulty**

**Intermediate**

---

### Question 15: What changed in Spring Boot 3?

**1. Interview-ready answer**

Spring Boot 3 requires Java 17, is built on Spring Framework 6, migrates from javax to Jakarta EE 10, and adopts Micrometer Observability for tracing and metrics. It also changes auto-configuration registration to `AutoConfiguration.imports`, supports AOT compilation and native images, and removes several deprecated APIs.

**2. Detailed explanation**

Major changes:

| Area | Boot 2 | Boot 3 |
|---|---|---|
| Java baseline | Java 8 | Java 17 |
| Spring Framework | 5.x | 6.x |
| Servlet API | `javax.servlet` | `jakarta.servlet` |
| Auto-config imports | `spring.factories` | `AutoConfiguration.imports` |
| Observability | Spring Sleuth | Micrometer Tracing |
| Native image | Experimental | Production-oriented AOT |

**3. Internal working**

The Jakarta migration is a package rename, not just a version bump. Filters, servlets, JPA annotations, and validation annotations must import from `jakarta.*`. AOT builds a native-friendly context configuration ahead of runtime.

**4. Practical Java/Spring Boot example**

```java
import jakarta.servlet.Filter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import org.springframework.stereotype.Component;

@Component
class RequestIdFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws java.io.IOException,
            jakarta.servlet.ServletException {
        chain.doFilter(request, response);
    }
}
```

Using `javax.servlet.Filter` would fail to compile in Boot 3.

**5. Real-world use case**

Upgrading a Boot 2 service to Boot 3 requires changing JPA, servlet, and validation imports and verifying observability dependencies.

**6. Interview follow-up questions**

- **Can Boot 3 run on Java 11?** No, Java 17 is minimum.
- **Is Sleuth still used?** Replaced by Micrometer Tracing.
- **Do all javax imports change?** Jakarta EE APIs do; some unrelated `javax` packages remain.
- **What is AOT?** Ahead-of-time context optimization, useful for native images.
- **Is native image mandatory?** No.

**7. Common mistakes**

- Mixing javax and jakarta dependencies.
- Assuming only version numbers changed.
- Keeping Boot 2 observability configuration.
- Upgrading without checking third-party library compatibility.

**8. Interview difficulty**

**Intermediate**

---

---

## Part D — Packaging, testing, and coding

### Question 16: How does an executable Spring Boot fat jar work?

**1. Interview-ready answer**

A Boot fat jar contains application classes in `BOOT-INF/classes`, dependencies in `BOOT-INF/lib`, and Boot's `JarLauncher` in the root. `java -jar` starts `JarLauncher`, which creates a custom classloader that can load nested jars and then invokes the application's `main` method.

**2. Detailed explanation**

Structure:

```text
orders.jar
├── META-INF/MANIFEST.MF
├── org/springframework/boot/loader/JarLauncher.class
├── BOOT-INF/classes
│   └── application.yml
└── BOOT-INF/lib
    ├── spring-boot-3.x.jar
    └── ...
```

The manifest's `Main-Class` is `JarLauncher`; `Start-Class` is the application main class.

**3. Internal working**

The standard Java launcher cannot load classes from nested jars directly. Boot's `LaunchedURLClassLoader` extends `URLClassLoader` and registers nested jar entries as classpath resources. `PropertiesLauncher` supports additional external classpath locations via `loader.path`.

**4. Practical Java/Spring Boot example**

```bash
java -jar orders-service.jar --server.port=9090

java -Dloader.path=/opt/ext -cp orders-service.jar \
  org.springframework.boot.loader.launch.PropertiesLauncher
```

Inspect contents:

```bash
jar tf orders-service.jar
```

**5. Real-world use case**

A deployment pipeline builds one immutable jar and runs it with environment-specific arguments, without unpacking dependencies.

**6. Interview follow-up questions**

- **Can you run a Boot jar with `java -cp` directly?** Not normally; use JarLauncher or PropertiesLauncher.
- **How do you add external jars?** `loader.path` with PropertiesLauncher.
- **Why nested jars?** Single deployable artifact with isolated dependencies.
- **Can Boot build a WAR?** Yes with `spring-boot-starter-tomcat` provided scope.
- **Does the fat jar include the JDK?** No.

**7. Common mistakes**

- Editing a jar in place.
- Assuming `BOOT-INF/lib` is on the normal system classpath.
- Forgetting `Start-Class` when packaging manually.
- Shipping debug artifacts to production.

**8. Interview difficulty**

**Intermediate**

---

### Question 17: How do auto-configuration ordering and backoff work?

**1. Interview-ready answer**

Auto-configurations are sorted with `@AutoConfigureOrder`, `@AutoConfigureBefore`, and `@AutoConfigureAfter`. Backoff is implemented with `@ConditionalOnMissingBean`, so if an application defines its own bean, Boot's default bean is skipped. Conditions are evaluated in definition order, so ordering matters when one condition depends on another configuration's definitions.

**2. Detailed explanation**

Annotations:

| Annotation | Purpose |
|---|---|
| `@AutoConfigureOrder` | Global ordering |
| `@AutoConfigureBefore` | Run before a named config |
| `@AutoConfigureAfter` | Run after a named config |
| `@ConditionalOnMissingBean` | Back off if bean exists |
| `@ConditionalOnBean` | Activate only if bean exists |

**3. Internal working**

`AutoConfigurationSorter` builds a dependency graph from ordering annotations and sorts candidates before condition evaluation. This ensures, for example, that web MVC configuration is processed after core servlet configuration.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.boot.autoconfigure.AutoConfiguration;
import org.springframework.boot.autoconfigure.AutoConfigureAfter;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;

@AutoConfiguration
@AutoConfigureAfter(CoreClientAutoConfiguration.class)
class ReportingAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    ReportClient reportClient(CoreClient coreClient) {
        return new DefaultReportClient(coreClient);
    }
}
```

If the application defines its own `ReportClient`, `reportClient` is not registered.

**5. Real-world use case**

A custom starter configures a metrics client only after the tracing auto-configuration has registered the tracer it depends on.

**6. Interview follow-up questions**

- **Why is ordering important for OnBean conditions?** Conditions see definitions registered so far.
- **Can user config override auto-config?** Yes.
- **What happens with two auto-configs defining the same bean type?** Ambiguity or backoff depending on conditions.
- **How do you debug order?** Condition evaluation report with `--debug`.
- **Is @Order the same as @AutoConfigureOrder?** No.

**7. Common mistakes**

- Using @Order on auto-config classes.
- Forgetting to import the config in AutoConfiguration.imports.
- Conditions depending on instantiated beans.
- Missing backoff conditions in starters.

**8. Interview difficulty**

**Advanced**

---

### Question 18: What are Spring Boot test slices?

**1. Interview-ready answer**

Test slices load only part of the application context, making tests faster and more focused. `@WebMvcTest` loads web layer beans, `@DataJpaTest` loads JPA repositories, `@JsonTest` tests JSON serialization, and `@SpringBootTest` loads the full context. Use mocks for collaborators not included in the slice.

**2. Detailed explanation**

| Annotation | Loads |
|---|---|
| `@SpringBootTest` | Full application context |
| `@WebMvcTest` | Controllers, MVC infrastructure |
| `@WebFluxTest` | Reactive web layer |
| `@DataJpaTest` | JPA entities/repositories, embedded DB |
| `@JsonTest` | JSON converters |
| `@RestClientTest` | RestClient/RestTemplate configuration |

**3. Internal working**

Slice annotations use type-exclude filters and auto-configuration imports to build a restricted context. `@MockBean`/`@MockitoBean` replaces beans in the context with Mockito mocks.

**4. Practical Java/Spring Boot example**

```java
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.web.servlet.MockMvc;

@WebMvcTest(OrderController.class)
class OrderControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private OrderService orderService;

    @Test
    void returnsOk() throws Exception {
        when(orderService.exists("ORD-1")).thenReturn(true);

        mockMvc.perform(get("/orders/ORD-1"))
                .andExpect(status().isOk());
    }
}
```

**5. Real-world use case**

Controller validation tests use `@WebMvcTest` with a mocked service; integration tests use `@SpringBootTest` with Testcontainers for PostgreSQL.

**6. Interview follow-up questions**

- **When use @SpringBootTest?** For integration/wiring confidence.
- **Why not always full context?** Slow startup and broad failures.
- **Does @WebMvcTest load services?** No, mock them.
- **What database does @DataJpaTest use?** An embedded database by default.
- **How do you test real PostgreSQL?** Testcontainers with `@ServiceConnection`.

**7. Common mistakes**

- Using full context for every unit test.
- Expecting unmocked beans to exist in a slice.
- Mixing slice and full-context assumptions.
- Overusing mocks for integration paths.

**8. Interview difficulty**

**Intermediate**

---

### Question 19: What are production configuration best practices for Spring Boot?

**1. Interview-ready answer**

Externalize secrets, validate configuration at startup, expose only secured Actuator endpoints, use structured logging with trace IDs, configure probes and graceful shutdown, avoid DEBUG logs, pin dependency versions via the BOM, and keep environment-specific values outside the artifact. Use profiles for bean groups, not for storing secrets in Git.

**2. Detailed explanation**

Checklist:

- Secrets from environment/secret manager, never in Git.
- `@ConfigurationProperties` with validation.
- Separate management port or network policy.
- Health/liveness/readiness probes.
- Bounded thread pools and connection pools.
- Graceful shutdown and termination period.
- JSON logs with correlation IDs.
- Metrics and tracing via Micrometer.
- Fail fast on missing critical config.
- Audit dependencies and image layers.

**3. Internal working**

Boot's environment abstraction allows the same artifact to run across environments. Startup validation via `@Validated @ConfigurationProperties` prevents a service from appearing healthy with invalid config.

**4. Practical Java/Spring Boot example**

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USER}
    password: ${DB_PASSWORD}

management:
  server:
    port: 9090
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  endpoint:
    health:
      probes:
        enabled: true
      show-details: never

server:
  shutdown: graceful

logging:
  level:
    root: INFO
```

**5. Real-world use case**

A production service fails immediately if `DB_URL` is missing, exposes only health and metrics on a management port, and logs structured JSON for observability.

**6. Interview follow-up questions**

- **How do you avoid secret leakage via Actuator?** Do not expose env/heapdump; secure management port.
- **How do you fail fast?** Validate properties or use startup runners.
- **Why separate management port?** Limit network exposure.
- **Should profiles contain passwords?** No.
- **How do you audit dependencies?** Dependabot/Renovate, SBOM, CVE scanning.

**7. Common mistakes**

- Committing production credentials.
- DEBUG logging in production.
- Exposing all Actuator endpoints.
- No readiness probe during rolling deploys.
- Unbounded executors and connection pools.

**8. Interview difficulty**

**Intermediate**

---

### Question 20: Coding challenge — create a custom auto-configuration

**1. Interview-ready answer**

I create a properties class bound to `audit.*`, a default `AuditClient` implementation, and an `@AutoConfiguration` class guarded by `@ConditionalOnProperty` and `@ConditionalOnMissingBean`. I register the configuration in `AutoConfiguration.imports` so Boot discovers it when the starter is on the classpath.

**2. Detailed explanation**

**Problem statement:** Build a reusable auto-configuration that creates a console audit client only when `audit.enabled=true` and no custom `AuditClient` bean exists.

**Approach:**
1. Define `AuditProperties` with `@ConfigurationProperties`.
2. Define `AuditClient` and a default implementation.
3. Create `AuditAutoConfiguration` with conditions.
4. Register it in `AutoConfiguration.imports`.
5. Use it in an application with `audit.enabled=true`.

**3. Internal working**

`@AutoConfiguration` is discovered from `AutoConfiguration.imports`. `@EnableConfigurationProperties` registers the properties bean. `@ConditionalOnMissingBean` backs off when the application supplies its own bean.

**4. Practical Java solution**

```java
package com.example.audit;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "audit")
public record AuditProperties(boolean enabled, String sink) {
}
```

```java
package com.example.audit;

public interface AuditClient {

    void record(String event);
}
```

```java
package com.example.audit;

public final class ConsoleAuditClient implements AuditClient {

    private final String sink;

    public ConsoleAuditClient(String sink) {
        this.sink = sink;
    }

    @Override
    public void record(String event) {
        System.out.println("[" + sink + "] " + event);
    }
}
```

```java
package com.example.audit;

import org.springframework.boot.autoconfigure.AutoConfiguration;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;

@AutoConfiguration
@EnableConfigurationProperties(AuditProperties.class)
@ConditionalOnProperty(prefix = "audit", name = "enabled", havingValue = "true")
public class AuditAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    AuditClient auditClient(AuditProperties properties) {
        return new ConsoleAuditClient(properties.sink());
    }
}
```

`src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`:

```text
com.example.audit.AuditAutoConfiguration
```

Application configuration:

```yaml
audit:
  enabled: true
  sink: stdout
```

Usage:

```java
import com.example.audit.AuditClient;
import org.springframework.stereotype.Service;

@Service
class OrderService {

    private final AuditClient auditClient;

    OrderService(AuditClient auditClient) {
        this.auditClient = auditClient;
    }

    void checkout(String orderId) {
        auditClient.record("checkout:" + orderId);
    }
}
```

Output when `checkout("ORD-1")` runs:

```text
[stdout] checkout:ORD-1
```

**5. Dry run**

| Step | Condition/action | Result |
|---|---|---|
| 1 | Boot reads AutoConfiguration.imports | AuditAutoConfiguration candidate loaded |
| 2 | audit.enabled=true | class-level condition passes |
| 3 | No custom AuditClient | bean method runs |
| 4 | Application injects AuditClient | ConsoleAuditClient used |

**6. Complexity**

- Time: O(1) at startup beyond normal bean creation.
- Space: O(1) for one client bean.

**7. Edge cases**

- `audit.enabled=false` → no client bean; injection fails if a service requires it.
- Custom AuditClient bean → default backs off.
- Missing `audit.sink` → null sink; use a default value in properties.
- Wrong imports file location → auto-configuration is not discovered.
- Boot 2 style `spring.factories` → ignored for auto-configuration in Boot 3.

**8. Real-world use case**

An internal starter auto-configures audit, tracing, and retry clients for multiple services with property-driven opt-in.

**9. Interview follow-up questions**

- **Why @ConditionalOnMissingBean?** It lets applications override defaults.
- **Where must the imports file live?** `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`.
- **How do you order this config?** `@AutoConfigureBefore`/`@AutoConfigureAfter`.
- **How do you debug whether it loaded?** `--debug` condition report.
- **How would you make the client conditional on a class?** `@ConditionalOnClass`.

**10. Common mistakes**

- Using `@Configuration` instead of `@AutoConfiguration`.
- Forgetting the imports file.
- Requiring a bean that conditions may skip.
- Hardcoding values instead of binding properties.

**11. Interview difficulty**

**Advanced**

---

## Chapter 10 Verification

| Syllabus topic | Question(s) |
|---|---|
| Spring Boot overview | Q1 |
| @SpringBootApplication | Q2 |
| Auto-configuration internals | Q3, Q17, Q20 |
| Starters | Q4 |
| Startup lifecycle | Q5 |
| Embedded servers | Q6 |
| Properties vs YAML | Q7 |
| Configuration precedence | Q8 |
| Profiles and profile groups | Q9 |
| ConfigurationProperties | Q10, Q20 |
| Actuator | Q11 |
| Logging | Q12 |
| Environment variables and containers | Q13 |
| Graceful shutdown | Q14 |
| Spring Boot 3 changes | Q15 |
| Executable fat jar | Q16 |
| Testing slices | Q18 |
| Production configuration | Q19 |
| Custom auto-configuration | Q20 |

**Total interview questions: 20** — all with the required 8-part format; coding questions include problem statement, approach, Java solution, dry run, complexity, and edge cases.

**Next:** [Chapter 11 — REST API Development](11-REST-API-Development.md). Say `NEXT CHAPTER` to continue.
