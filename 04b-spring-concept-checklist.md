# 04b · Spring and Java Concept Checklist

Every Spring Boot and backend Java concept you should touch at least once during your internship, in one flat list. Copy it into your progress file and tick items as you go. An item is ticked when you have **used it in code and can explain it**, not when you have read about it.

**Our usage** tells you how much it matters here, based on how often it appears across our Java services:

- **Heavy:** used throughout at least one core service. Learn it well in the first month.
- **Some:** used in several places. Learn it when you meet it, but meet it on purpose.
- **Rare:** a few uses, or the right tool we should use more.
- **None:** we do not use it. Recognise it, know why not.

**Where** is the section of [04-spring-for-node-and-python-devs.md](04-spring-for-node-and-python-devs.md) that covers it, or the teaching project milestone that exercises it.

---

## Java language

| Done | Concept | Our usage | Where |
|---|---|---|---|
| [ ] | Classes, interfaces, packages, access modifiers | Heavy | S1 |
| [ ] | Records for DTOs | Some | S1, P1 |
| [ ] | Generics and type erasure | Heavy | S1 |
| [ ] | Checked versus unchecked exceptions | Heavy | S1, S5 |
| [ ] | `Optional`, and its misuses | Heavy | S1 |
| [ ] | Streams, lambdas, collectors | Heavy | S1, P1 |
| [ ] | Java 21: `var`, text blocks, switch expressions, pattern matching, sealed types | Some | S1 |
| [ ] | `equals` and `hashCode` contracts | Heavy | S1, S4 |
| [ ] | Lombok: `@Data`, `@Getter`, `@Builder`, `@RequiredArgsConstructor`, `@Slf4j` | Heavy | S1 |
| [ ] | Maven: `pom.xml`, lifecycle phases, dependency scopes, the wrapper, dependency tree | Heavy | S1, S9 |

## Spring core

| Done | Concept | Our usage | Where |
|---|---|---|---|
| [ ] | Spring Initializr, starters, project layout | Heavy | S1, P0 |
| [ ] | `@SpringBootApplication` and component scanning | Heavy | S2 |
| [ ] | Beans and stereotypes: `@Component`, `@Service`, `@Repository`, `@RestController` | Heavy | S2 |
| [ ] | Constructor injection | Heavy | S2, P2 |
| [ ] | `@Configuration` and `@Bean` | Heavy | S2 |
| [ ] | Bean scopes, and singletons sharing state | Heavy | S2, S7 |
| [ ] | Proxies and the self-invocation trap | Heavy | S2 |
| [ ] | `@Profile` and conditional beans | Some | S2, S6 |
| [ ] | `@ConfigurationProperties` versus `@Value` | Heavy | S2, S6 |
| [ ] | `ApplicationEventPublisher` and `@EventListener` | Rare | S2 |
| [ ] | Auto-configuration and the condition evaluation report | Some | S9 |

## Web layer

| Done | Concept | Our usage | Where |
|---|---|---|---|
| [ ] | `@RestController`, mappings, path and query parameters, request bodies | Heavy | S3, P3 |
| [ ] | CRUD with correct HTTP methods and status codes | Heavy | S3, P3 |
| [ ] | Bean Validation with `@Valid` | Some | S3, P3 |
| [ ] | `@RestControllerAdvice` and `@ExceptionHandler` | Some | S3, P3 |
| [ ] | `ProblemDetail` error bodies | Rare | S3, P3 |
| [ ] | `ResponseEntity` | Heavy | S3 |
| [ ] | Filters versus interceptors | Some | S3 |
| [ ] | Jackson configuration: naming, nulls, dates | Heavy | S3 |
| [ ] | DTOs at the edges, never entities in responses | Some | S3, S9 |
| [ ] | CORS configuration | Some | S3 |
| [ ] | SOAP web services starter | Some | Recognise only |

## Data and JPA

| Done | Concept | Our usage | Where |
|---|---|---|---|
| [ ] | `@Entity`, `@Id`, ID generation strategies | Heavy | S4, P4 |
| [ ] | `JpaRepository` and derived query methods | Heavy | S4, P4 |
| [ ] | `@Query` with JPQL and native SQL | Heavy | S4 |
| [ ] | Projections: interface and record | Some | S4 |
| [ ] | `Pageable`, `Page`, `Slice`, sorting | Heavy | S4, P3 |
| [ ] | Persistence context and entity states | Heavy | S4 |
| [ ] | Lazy versus eager loading, N+1, fetch joins, `@EntityGraph` | Heavy | S4, S5, L5 |
| [ ] | `@ManyToOne` and `@OneToMany` with `mappedBy` | Heavy | S5, P5 |
| [ ] | `@OneToOne` | Heavy | S5, P5 |
| [ ] | `@ManyToMany`, and when to replace it with an entity | Heavy | S5, P5 |
| [ ] | Owning side, cascade types, orphan removal | Heavy | S5, P5 |
| [ ] | `@Transactional`: placement, `readOnly`, rollback rules | Heavy | S5, P5 |
| [ ] | Propagation and isolation levels | Some | S5 |
| [ ] | Optimistic locking with `@Version` | Rare, and should be more common | S5, P5 |
| [ ] | Pessimistic locking with `@Lock` | Rare | S5 |
| [ ] | HikariCP pool configuration | Heavy | S4, L4 |
| [ ] | `ddl-auto`, and migration tools such as Flyway | Heavy, with known debt | S4 |
| [ ] | Spring Data Elasticsearch | Some | Recognise, then stretch goal |
| [ ] | Testcontainers for real MySQL in tests | Rare, and should be more common | S9, P9 |

## Concurrency and async

| Done | Concept | Our usage | Where |
|---|---|---|---|
| [ ] | Thread safety, race conditions, check-then-act | Heavy | S7, L10 |
| [ ] | `ExecutorService` and thread pool sizing | Heavy | S7, P7 |
| [ ] | `CompletableFuture` composition and error handling | Heavy | S7, P7 |
| [ ] | `ConcurrentHashMap` and atomic compute and merge | Heavy | S7, P7 |
| [ ] | Atomics and `LongAdder` | Heavy | S7, P7 |
| [ ] | `synchronized` | Heavy | S7 |
| [ ] | `ReentrantLock`, `ReadWriteLock` | Some | S7 |
| [ ] | `volatile` and visibility | Some | S7 |
| [ ] | Deadlocks and reading thread dumps | Some | S7 |
| [ ] | `@Async` and configuring its executor | Heavy | S7 |
| [ ] | Virtual threads and pinning | Rare | S7, P7 |
| [ ] | `ThreadLocal` and MDC propagation | Some | S7 |
| [ ] | `@Scheduled`, `ThreadPoolTaskScheduler`, overlap protection | Heavy | S6, P6 |

## Integration

| Done | Concept | Our usage | Where |
|---|---|---|---|
| [ ] | `RestTemplate` and its error handling | Heavy | S6, P6 |
| [ ] | OpenFeign clients, timeouts, error decoders | Heavy | S6, P6 |
| [ ] | `WebClient` | Some | S6 |
| [ ] | `RestClient` | Rare, preferred for new code | S6, P6 |
| [ ] | Retries: `@Retryable`, Resilience4j, backoff | Rare, and should be more common | S6, P6 |
| [ ] | `@Cacheable`, `@CacheEvict`, cache providers | Some | S6, P6 |
| [ ] | `KafkaTemplate` and `@KafkaListener` | Heavy | S8, P8 |
| [ ] | Consumer groups, acknowledgement modes, error handlers, dead letters | Heavy | S8, L9 |
| [ ] | WebSockets, STOMP messaging, SSE | Some | S8, P8 |
| [ ] | Actuator endpoints and Micrometer metrics | Heavy | S6, L8 |
| [ ] | Micrometer Tracing and OpenTelemetry | Some | S8, L8 |

## Security

| Done | Concept | Our usage | Where |
|---|---|---|---|
| [ ] | `SecurityFilterChain` with the Boot 3 lambda DSL | Heavy | S8, P8 |
| [ ] | Matcher ordering, `permitAll`, `authenticated` | Heavy | S8 |
| [ ] | Roles versus authorities | Heavy | S8 |
| [ ] | `@PreAuthorize` and `@EnableMethodSecurity` | Heavy | S8, P8 |
| [ ] | `@Secured` | None | Recognise only |
| [ ] | JWT resource server and claim-to-authority mapping | Heavy | S8, P8 |
| [ ] | OAuth2 login flows, refresh tokens and rotation | Some | S8 |
| [ ] | CSRF, and when to disable it | Some | S8 |
| [ ] | Service-to-service authentication | Some | S8 |

## Testing and quality

| Done | Concept | Our usage | Where |
|---|---|---|---|
| [ ] | JUnit 5, AssertJ | Some | S9, P9 |
| [ ] | Mockito: mocks, stubs, verification, argument captors | Some | S9, P9 |
| [ ] | `@WebMvcTest` | Rare | S9, P9 |
| [ ] | `@DataJpaTest` | Rare | S9, P9 |
| [ ] | `@SpringBootTest` and test profiles | Some | S9 |
| [ ] | Contract tests for shared payloads | None yet | S9, P9 |
| [ ] | Layering: thin controllers, business logic in services | Some | S9 |
| [ ] | DTO mapping by hand or with MapStruct | Some | S9 |
| [ ] | Naming, single responsibility, failing loudly | Some | S9 |

## Recognise, but we do not use

| Done | Concept | Why we do not use it |
|---|---|---|
| [ ] | Eureka service discovery | The deployment platform and DNS locate services |
| [ ] | Ribbon client load balancing | Removed from Spring Cloud; replaced by Spring Cloud LoadBalancer, which we also do not need |
| [ ] | Zuul gateway | Removed from Spring Cloud; Spring Cloud Gateway replaced it; our edge is a reverse proxy and Cloudflare |
| [ ] | Sleuth and Zipkin | Replaced by Micrometer Tracing in Boot 3; we trace with OpenTelemetry |
| [ ] | Spring Cloud Config Server | Configuration is injected by the deployment platform |
| [ ] | RabbitMQ | We standardised on Kafka for messaging |
