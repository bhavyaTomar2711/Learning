# 04 · Spring Boot for People Who Know Express or FastAPI

You already know how to build a backend. You know routes, middleware, a data layer, configuration, and async. This document does not teach you backend development. It teaches you the *names* Spring gives to things you already understand, the places where the analogy breaks, and the handful of ideas that are genuinely new.

Nine sections, one per day, ninety minutes each, Days 2 to 10. Each section ends with two tasks:

- **Build:** a milestone of your teaching project, [10-teaching-project.md](10-teaching-project.md).
- **Read:** an exercise on one of our real Java repositories that needs only the code, never a running service.

Do both. Reading Spring is useless without running it, and running your own Spring is useless without seeing how a large real codebase does it.

Our services are Spring Boot 3 on Java 21, built with Maven. When you look things up, always include "Spring Boot 3" in the query. Half of what you find online is for Boot 2 and will not work.

A flat list of every Spring and Java concept you should eventually touch, with how heavily our code uses each, is in [04b-spring-concept-checklist.md](04b-spring-concept-checklist.md). Tick it off as you go.

---

## Section 1 · Java the language, and a project from scratch

You know C++ or TypeScript or Python. Java sits between them. What is actually different:

**Look up carefully.**
- Everything is a class, but since Java 16 `record` gives you an immutable data class in one line, and that is what DTOs should be.
- Static typing with generics; type erasure and why `List<String>` and `List<Integer>` are the same at runtime.
- Checked exceptions: the compiler forces you to declare or handle them. Runtime exceptions do not. Spring converts most checked exceptions into runtime ones. Learn the difference once.
- `Optional` as the answer to null, and the two ways people misuse it: as a field and as a parameter.
- Streams and lambdas: `map`, `filter`, `collect`, `groupingBy`. This is your `array.map` and list comprehension.
- Interfaces and why Spring codes against them everywhere.
- Java 21 features you will see: records, `var`, text blocks, switch expressions, pattern matching for `instanceof` and `switch`, sealed interfaces.
- The JVM: bytecode, JIT, why startup is slow and steady state is fast.
- Packages, `import`, and the one-public-class-per-file rule.
- `equals`, `hashCode`, and why records get them for free while classes do not.
- **Lombok.** Our services use it everywhere: `@Data`, `@Getter`, `@Builder`, `@RequiredArgsConstructor`, `@Slf4j`. Learn what each generates, why `@Data` on a JPA entity is a trap, and why records replace much of what Lombok was for.
- **Spring Initializr and project structure.** What the generated `pom.xml`, main class, `src/main/resources`, and `src/test` are for. What a "starter" is.

**Skip for now.** Inner classes, annotation processing internals, modules, reflection. You will meet them when you need them.

**Build.** Teaching project **P0 and P1**: generate the project from Initializr, get it healthy against MySQL in Compose, then write the domain statistics command. Build and run from the command line with Maven.

**Read.** In the dashboard backend, count how Lombok is used on entities versus DTOs. Find one entity with `@Data` and write two sentences on what could go wrong.

---

## Section 2 · Dependency injection, or why nothing has a constructor call

**What you know.** In Express you `require` a module and call it. In FastAPI you write `Depends(get_db)` and the framework passes it in.

**What Spring does.** Spring runs the whole application as a container of objects called *beans*. You never construct a service; you declare a class as a bean (`@Service`, `@Component`, `@Repository`, `@Controller`) and ask for it by type in a constructor. Spring builds the graph at startup. FastAPI's `Depends` is the same idea applied per request instead of at startup, and without a container owning object lifetimes.

**Where the analogy breaks.**
- **Beans are singletons by default.** One instance for the whole application. State in a field is shared by every request on every thread. This is the first bug every Node developer writes in Spring, and it connects directly to Section 7.
- Startup fails if the graph cannot be satisfied. That is a feature. Read the error; it names the missing bean.
- Constructor injection is the only injection you should write. Field injection with `@Autowired` exists in older code; do not add more.
- `@Configuration` classes with `@Bean` methods register things you did not write, such as an HTTP client or a Kafka template.
- **Proxies.** Spring wraps some beans in a generated subclass to add behaviour: transactions, security, caching, async. Calling a method on `this` bypasses the proxy. This explains a whole family of "my annotation did nothing" bugs.

**Look up carefully.**
- Bean scopes: singleton, prototype, request. When each is right.
- Component scanning and why the main class's package matters.
- `@ConfigurationProperties` for typed configuration instead of `@Value` sprinkled everywhere.
- Circular dependencies: how to recognise the error and why the fix is a design change, not a flag.
- `@Profile` and conditional beans.
- Application events: `ApplicationEventPublisher` and `@EventListener`, as an in-process alternative to calling another service directly.

**Build.** Teaching project **P2**: package structure, constructor injection, typed configuration for the registrar settings.

**Read.** In the dashboard backend, pick one service class and draw its dependency graph three levels deep by reading constructors. Find a bean that is a proxy because it has `@Transactional`, `@Async`, or `@Cacheable`, and look for a self-invocation trap.

---

## Section 3 · The web layer: routes, middleware, validation, errors

**What you know.** Express `app.get(path, middleware, handler)`. FastAPI `@router.get(path)` with pydantic models for body and response.

**What Spring does.**

| You know | Spring calls it | Notes |
|---|---|---|
| Router or blueprint | `@RestController` with `@RequestMapping` on the class | One class per resource is the convention |
| Route handler | Method with `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping` | Return an object; Jackson serialises it |
| Path param, query param, body | `@PathVariable`, `@RequestParam`, `@RequestBody` | Explicit, unlike FastAPI's inference |
| pydantic model | A `record` DTO with Bean Validation annotations (`@NotNull`, `@Size`) plus `@Valid` on the parameter | Validation is opt-in per parameter. Forget `@Valid` and nothing validates |
| Middleware | Filters, which run before Spring, and interceptors, which know the handler | Order matters and is configured, not positional |
| Error handler middleware | `@RestControllerAdvice` with `@ExceptionHandler` methods | One place that maps exceptions to HTTP responses |
| `res.status(201).json(x)` | `ResponseEntity.status(201).body(x)` | Or return the object for 200 |

**Where the analogy breaks.**
- Jackson decides the JSON shape. Field naming, null handling, and date formats are configuration, and the defaults are not what you expect.
- There is no request object flowing through everything. Request-scoped data across layers is what MDC or a request-scoped bean is for.
- **Never return entities from controllers.** Map to DTOs. Returning entities leaks your schema, triggers lazy loading during serialisation, and couples your API to your database.

**Look up carefully.**
- The servlet model and `DispatcherServlet`, just enough to know where filters end and controllers begin.
- HTTP method semantics: safe, idempotent, and which of GET, POST, PUT, PATCH, DELETE are which.
- `ProblemDetail` (RFC 7807) as the modern error body.
- Bean Validation groups and custom validators.
- Pagination and sorting parameters with `Pageable`.
- Content negotiation and CORS configuration.

**Build.** Teaching project **P3**: CRUD for domains and auctions, validation, pagination, a global exception handler.

**Read.** Find the global exception handler in the dashboard backend, if there is one, and describe the error shape it produces. Then find two controllers that do not follow it. Inconsistency is normal in real codebases; noticing it is the skill.

---

## Section 4 · Data: JPA, Hibernate, repositories, and the schema that manages itself

**What you know.** Mongoose schemas or Prisma models; `Model.find({})`; SQLAlchemy sessions.

**What Spring does.** JPA is the specification, Hibernate is the implementation. An `@Entity` class maps to a table. A `JpaRepository<Entity, Id>` interface gives you `findAll`, `findById`, `save`, `delete` with no implementation. Method names like `findByStatusAndEndsAtAfter` are parsed into queries. `@Query` lets you write JPQL or native SQL when the names get silly.

**Where the analogy breaks, and it breaks hard.**
- **Entities are managed objects.** Inside a transaction, Hibernate tracks changes and writes them at commit without you calling `save`. Outside a transaction, the same code does nothing. This is the persistence context, and it is the source of most confusion in JPA.
- **Lazy loading.** A relation is not loaded until you touch it. Touch it outside the transaction and you get `LazyInitializationException`. Touch it inside a loop and you get N+1 queries. Both are the same design decision seen from two sides.
- **`ddl-auto`.** Hibernate can create and alter your tables from the entities at startup. In our dashboard backend this is `update`, in every environment. It adds columns and tables; it never drops or narrows. A Flyway folder exists there but does not run. Verify what a service actually does before you touch an entity.
- **Entity equality.** Two entities with the same id may not be `equals`, and mutable ids break hash sets. Read the standard guidance before overriding anything.

**Look up carefully.**
- Persistence context and entity states: transient, managed, detached, removed.
- Derived query methods, `@Query` with JPQL, native queries, and when each is right.
- Projections: interface-based and DTO (record) projections as the escape hatch from loading whole entities.
- Pagination with `Pageable` and `Page`, and the extra count query it costs; `Slice` when you do not need the count.
- Hibernate's SQL logging properties for seeing what actually runs.
- HikariCP configuration keys and defaults.
- `ddl-auto` values, and how Flyway or Liquibase replace them in a disciplined codebase.
- MySQL specifics with Hibernate: dialect, identifier quoting, ID generation strategies and why `IDENTITY` disables batch inserts.

**Build.** Teaching project **P4** with lab **L5**: entities, repositories, seeding, SQL logging, an N+1 found and fixed, an index with plans before and after.

**Read.** In the dashboard backend, find three derived query methods and three `@Query` methods. For each `@Query`, predict the SQL. Find one collection relation that would cause N+1 if iterated in a list endpoint.

---

## Section 5 · JPA relationships and transactions, properly

The old way to learn Spring spent a full week here, and it was right to. Our dashboard backend has many one-to-one and many-to-many relationships, and getting them wrong corrupts data quietly.

**Relationships.**

| Mapping | Example in the teaching project | What to get right |
|---|---|---|
| `@ManyToOne` | `Bid` to `Auction` | This side owns the foreign key. Default fetch is EAGER; change it to LAZY almost always |
| `@OneToMany(mappedBy = ...)` | `Auction` to its `Bid`s | The inverse side. `mappedBy` names the field on the owning side. Default fetch is LAZY |
| `@OneToOne` | `Domain` to `DomainProfile` | Decide which table holds the key. Lazy one-to-one on the inverse side often is not lazy |
| `@ManyToMany` | `AppUser` to `Domain` via a watchlist | A join table. The moment the join needs a column such as "added at", replace it with an explicit entity and two many-to-ones |

**Look up carefully.**
- The **owning side** versus the **inverse side**, and why only changes on the owning side are written.
- Keeping both sides of a bidirectional relationship in sync with helper methods.
- `cascade` types, and why `CascadeType.ALL` on a many-to-one is almost always a bug.
- `orphanRemoval` and how it differs from cascade remove.
- `FetchType.LAZY` versus `EAGER`, fetch joins, `@EntityGraph`, and `@BatchSize`.
- `Set` versus `List` for collections, and the "MultipleBagFetchException".

**Transactions.**

- `@Transactional` belongs on service methods, and the unit of work is the service method.
- Read-only methods get `@Transactional(readOnly = true)`.
- **Rollback rules:** by default a transaction rolls back on unchecked exceptions and **commits on checked exceptions**. This surprises everyone. Learn `rollbackFor`.
- **Propagation:** `REQUIRED`, `REQUIRES_NEW`, `NESTED`, and what each does when one transactional method calls another. Then remember that on the same class, the proxy is bypassed and none of it applies.
- **Isolation:** the four levels, what the database defaults to (MySQL InnoDB uses REPEATABLE READ), and when to override.
- **Locking:** optimistic locking with `@Version` and what `OptimisticLockException` means; pessimistic locking with `@Lock` and `SELECT ... FOR UPDATE`; when each is right.
- Never make a slow external call inside a transaction. It holds a database connection for the duration.

**Build.** Teaching project **P5** with lab **L4**: all relationships, place-bid as one transaction with optimistic locking, a rollback test.

**Read.** In the dashboard backend, find one `@ManyToMany` and one `@OneToOne`. For each, identify the owning side, the cascade settings, and the fetch type, and say whether you would change anything and why. Find one `@Transactional` method that calls an external service.

---

## Section 6 · Configuration, outbound HTTP, scheduling, and caching

**Configuration.** `application.properties` or YAML holds defaults. `application-{profile}.properties` overrides it when that profile is active. Environment variables override files. Command-line arguments override everything. A property `spring.datasource.url` is satisfied by an environment variable `SPRING_DATASOURCE_URL` through relaxed binding. Typed access is `@ConfigurationProperties`; ad hoc access is `@Value("${some.key}")`. Prefer the former.

**Outbound HTTP.** Our code uses all of these, so you need to recognise all of them:

| Client | Style | Where you will see it |
|---|---|---|
| **`RestTemplate`** | Synchronous, imperative, older | Heavily in the auction service and in parts of the dashboard |
| **OpenFeign** | Declarative: an interface with `@GetMapping` annotations, generated at runtime | Throughout the dashboard backend |
| **`WebClient`** | Reactive, non-blocking, from WebFlux | Some dashboard code |
| **`RestClient`** | Synchronous, fluent, modern (Boot 3.2 and later) | The recommended choice for new synchronous code |

Whatever the client: **set timeouts**, handle non-2xx responses deliberately, and retry only idempotent calls.

**Scheduling.** `@Scheduled(cron = "...")` or `fixedDelay` on a bean method runs it on a scheduler thread. The default scheduler pool has **one thread**; a slow job delays every other job. `ThreadPoolTaskScheduler` configures a bigger pool. Nothing prevents two replicas of the service from running the same job.

**Caching.** `@Cacheable`, `@CachePut`, and `@CacheEvict` on bean methods, enabled with `@EnableCaching`, backed by a simple in-memory map by default or Caffeine or Redis in production. It goes through the proxy, so self-invocation is not cached.

**Look up carefully.**
- The full property source precedence list for Spring Boot 3, and relaxed binding rules for environment variables.
- Profile activation with `SPRING_PROFILES_ACTIVE`, and profile groups.
- Actuator: which endpoints to expose and which never to expose publicly.
- `RestTemplate` error handlers; Feign timeouts, retryers, error decoders, and logging levels; `WebClient` timeouts and why calling `.block()` inside a servlet app defeats its purpose.
- Spring Retry's `@Retryable`, and Resilience4j for retries, circuit breakers, and rate limiters.
- `@EnableScheduling`, `fixedRate` versus `fixedDelay` versus cron, and overlap protection with a lock such as ShedLock.
- Cache key generation, expiry with Caffeine, cache stampedes, and why cached entities are dangerous.

**Build.** Teaching project **P6** with lab **L7**: the registrar client twice, timeouts and retries, a non-overlapping scheduled sync, a cache, a Dockerfile.

**Read.** In the dashboard backend, find every Feign client interface and every `RestTemplate` or `WebClient` bean definition. For each, find its timeout configuration in code, or note that you could not find one. Stay in Java code; you do not need to open properties files for this.

---

## Section 7 · Java concurrency for backend work

This is the biggest gap in most people's Spring knowledge and the most important section for our auction service, which is built on concurrent code. It watches many auctions at once, runs many registrar calls in parallel, and must never double-bid.

**What you know.** Node has one thread and an event loop, so shared mutable state is safe between `await`s. Python asyncio is the same. Python threads have the GIL.

**What Java does.** Real threads, running truly in parallel on many cores, sharing the same heap. Any mutable state reachable from two threads is a potential race. Remember from Section 2 that singleton beans are shared by every request thread. That is where races hide.

**The toolbox, in the order you should reach for it.**

| Tool | Use it for |
|---|---|
| **No shared mutable state** | Immutable records, local variables, method parameters. The best fix for a race is having nothing to race on |
| **`ExecutorService` and thread pools** | Running work on a bounded number of threads. Never create threads by hand in a server |
| **`CompletableFuture`** | Composing asynchronous work: `supplyAsync`, `thenApply`, `thenCompose`, `allOf`, `exceptionally`. Our auction service uses this extensively |
| **Concurrent collections** | `ConcurrentHashMap` and its atomic `compute` and `merge` methods; `CopyOnWriteArrayList`; blocking queues |
| **Atomics** | `AtomicInteger`, `AtomicLong`, `AtomicReference`, `LongAdder` for counters and compare-and-set |
| **`synchronized`** | Simple mutual exclusion on a monitor |
| **`ReentrantLock` and `ReadWriteLock`** | Locks with timeouts, try-lock, and fairness |
| **Virtual threads** (Java 21) | Many cheap threads for blocking I/O. Not faster CPU, not a fix for races |
| **`@Async`** | Spring's way to run a bean method on an executor. Goes through the proxy. Configure its executor, or it may use a default you did not intend |

**Look up carefully.**
- Race conditions, check-then-act, and read-modify-write: why `if (!map.containsKey(k)) map.put(k, v)` is broken even on a `ConcurrentHashMap`.
- Visibility: what `volatile` guarantees and what it does not.
- Deadlocks: lock ordering, and how to read a thread dump to find one with `jstack` or `jcmd`.
- Thread pool sizing for I/O-bound versus CPU-bound work, bounded queues, and rejection policies. What happens when an unbounded queue fills memory.
- `CompletableFuture` pitfalls: forgetting to pass an executor (it then uses the common pool), swallowed exceptions, and blocking with `join` or `get` inside another async stage.
- Timeouts on futures: `orTimeout` and `completeOnTimeout`.
- `ThreadLocal` and why it leaks in thread pools; MDC as a ThreadLocal you must propagate to async work.
- Virtual thread pinning inside `synchronized` blocks, and what changed in recent Java versions.
- Structured concurrency, as a direction of travel.
- Distributed concurrency: a lock inside one JVM does nothing across two replicas. Database locks or optimistic versioning are the cross-instance tools.

**Build.** Teaching project **P7** with lab **L10**: the closing sprint, the race on the shared budget, the fix, and the platform-thread versus virtual-thread comparison.

**Read.** In the auction service, find three different concurrency tools in use: a `ConcurrentHashMap`, an atomic, and a `CompletableFuture` chain. For each, say what shared state it protects or what work it parallelises, and whether you can see a check-then-act race around it.

---

## Section 8 · Security, Kafka, and WebSockets

**Security.** Spring Security is a chain of filters that runs before your controllers. A `SecurityFilterChain` bean declares which paths need which authority and how identity is established: a JWT resource server, OAuth2 login, or a custom filter. Matcher order matters; the first match wins. Method-level `@PreAuthorize` adds checks inside the application, again through the proxy.

What is new compared with Express middleware: the chain is declared as configuration rather than sequenced by code position, and CSRF protection is on by default for session-based apps. For a token-based API it is usually disabled, deliberately.

**Look up carefully.**
- `SecurityFilterChain` configuration in Boot 3 with the lambda DSL, and why `WebSecurityConfigurerAdapter` examples online are obsolete.
- **Roles versus authorities:** a role is an authority with a `ROLE_` prefix; `hasRole("ADMIN")` versus `hasAuthority("ROLE_ADMIN")`.
- `@PreAuthorize` with SpEL expressions, `@EnableMethodSecurity`, and why `@Secured` is the older, less expressive option.
- `oauth2ResourceServer` with JWT: which claims are validated, and how authorities are extracted from claims.
- Refresh tokens: why access tokens are short-lived, how refresh works, and rotation.
- Service-to-service authentication: shared secrets versus signed tokens versus network isolation.

**Kafka.** `KafkaTemplate` produces. `@KafkaListener(topics, groupId)` consumes, on a container thread, with the container handling polling and offset commits. The acknowledgement mode decides whether commits happen automatically or when you call `ack()`. Error handlers and dead-letter publishing are configured on the container factory. Concurrency on a listener means multiple threads, each owning partitions. Deserialisation errors happen before your code and need their own handler.

**Look up carefully.**
- Spring Kafka acknowledgement modes, `ConcurrentKafkaListenerContainerFactory`, `DefaultErrorHandler`, `DeadLetterPublishingRecoverer`, `ErrorHandlingDeserializer`.
- Headers for event type, schema version, and trace propagation.
- Transactions in Kafka and why the outbox pattern is usually simpler.

**WebSockets and SSE.** The dashboard pushes live updates to the browser. Spring supports raw WebSockets, STOMP messaging over WebSockets with `@MessageMapping` and `SimpMessagingTemplate`, and SSE with `SseEmitter` or WebFlux. Know which our dashboard uses where, and why SSE is simpler when the browser only listens.

**Build.** Teaching project **P8** with lab **L8**: JWT roles and method security, event publishing, the notifier in your second stack, live updates in the page, metrics and a dashboard.

**Read.** Read the security configuration of the dashboard backend and the auction service. Write one paragraph each on how a request is authenticated. Then find every `@KafkaListener` and record its topic, group, and acknowledgement handling as far as the Java code shows.

---

## Section 9 · Testing, the build, clean code, and reading a Spring app cold

**Testing.**

| You know | Spring calls it |
|---|---|
| Jest unit test with mocks | JUnit 5 plus Mockito plus AssertJ, no Spring at all |
| Supertest against the app | `@WebMvcTest` with `MockMvc` (controller slice, services mocked) |
| A test database | `@DataJpaTest` with Testcontainers for real MySQL |
| Full app test | `@SpringBootTest`, sparingly, because it boots everything |
| Test config and fixtures | Test profiles, `@TestConfiguration`, `@BeforeEach` |

Slice tests are the sweet spot: fast, and they exercise Spring's wiring for one layer.

**The build.** `mvnw clean package -DskipTests` produces the fat JAR, and that is how our Dockerfiles build. Know what `-DskipTests` means about where tests run. The parent POM from Spring Boot pins dependency versions; you almost never write a version number for a starter.

**Clean code in a Spring codebase.**
- Thin controllers, services that hold business logic, repositories that only access data.
- DTOs at the edges, entities inside. Map explicitly, by hand or with MapStruct.
- One reason to change per class. A 3,000-line service is a smell you will find; do not add to it.
- Name things after the business: `placeBid`, not `processData`.
- Fail loudly: validate at the edge, throw specific exceptions, never catch and ignore.
- Configuration in `@ConfigurationProperties`, not magic strings.
- Log with context and at the right level, never secrets.
- Prefer constructor injection and immutability; they make tests trivial.
- Follow the conventions of the code around you, even when you would do it differently. Propose the change separately.

**Reading a Spring app cold.** In this order:
1. `pom.xml`: which starters are present tells you what the app does.
2. The main class: its package sets the component scan root.
3. Configuration classes: security chain, beans for clients, scheduling, async executors.
4. Controllers, then services, then repositories, following one request.
5. `@Scheduled` and `@KafkaListener` methods: the work that happens without a request.
6. Tests, if there are any: the only documentation that is verified.

**Recognise, but know why we do not use.** Older Spring courses teach Eureka for service discovery, Ribbon for client load balancing, Zuul as a gateway, and Sleuth with Zipkin for tracing. Ribbon and Zuul were removed from Spring Cloud years ago; Sleuth was replaced by Micrometer Tracing in Spring Boot 3. We do not use service discovery at all: the deployment platform and DNS do that job, and tracing goes through OpenTelemetry. Know what these tools are so you can read old material and old code.

**Build.** Teaching project **P9**: slice tests, Testcontainers, an event contract test, README and DEVLOG complete. Then the Day 10 demo.

**Read.** From a clean clone of a Java service you have not opened yet, such as the EPP service or the lead-gen orchestrator, apply the cold-reading order and produce a one-page description of what it does and how it is wired, in under an hour.

---

## After the nine sections

You now know enough Spring to read any of our Java services and to add code that fits. You do not know Spring deeply, and that is fine; depth comes from tickets. The things that will keep biting you for months, in order, are: the proxy and self-invocation, lazy loading outside a transaction, singleton beans holding mutable state that threads share, checked exceptions not rolling back, and matcher order in security. When something in Spring "does nothing", it is almost always one of those five.
