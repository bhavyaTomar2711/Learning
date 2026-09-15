# 01 · Mental Models: The Rosetta Matrix

This is the spine of the curriculum. Every module below is one concept that exists in every stack we run. For each one you get: the invariant (what is true regardless of language), how each of our stacks does it, the nuances that bite people in production, where it lives in our code, and a self-check.

Our stacks, and the column names used throughout:

| Column | What it means here |
|--------|--------------------|
| **Java** | Spring Boot 3 on Java 21: the auction service, the dashboard backend, the lead-gen orchestrator, the EPP service |
| **Python** | FastAPI and LangGraph services: lead-gen research, domain processing, negotiation and prediction agents |
| **Node** | Node and TypeScript on the server: the scraping service, Next.js route handlers in the marketplace app |
| **React** | Browser code: Create React App plus Material UI dashboards, and Next.js frontends |
| **Data** | MySQL (primary), PostgreSQL (newer services), MongoDB (audit), Elasticsearch (search), Redis (cache) |
| **Infra** | Docker, Compose, Coolify, Cloudflare, Tailscale, Prometheus, Loki, Grafana, Tempo |

How to use this file: read a module, then go learn the concept from any source you like until you can answer the self-check without notes. Then find it in our code. Then, if a lab exists for it, do the lab. Do not read the whole file in one sitting. It is a reference, and it is meant to be re-read as you grow.

If you are put on a new stack later, your first task is to add a column for it. That exercise *is* the method.

---

## M1 · Execution model: threads, event loops, and the two kinds of "at the same time"

**The invariant.** A program has some number of units that can make progress. Concurrency is *structuring* work so many things are in flight at once. Parallelism is *actually executing* on more than one core at the same instant. You can have either without the other. Every runtime picks a default unit of concurrency and a default way to wait, and that choice decides where blocking hurts.

**Rosetta.**

| | Unit of concurrency | What happens on blocking I/O | Parallelism | Where it bites |
|---|---|---|---|---|
| Java | OS thread per request (Tomcat pool), or virtual threads on 21 | The thread waits; another thread serves the next request | Real, across cores | Pool exhaustion: 200 threads, 10 DB connections, everyone queues on the DB |
| Python | One OS thread, one event loop, `async def` coroutines; plus worker processes | `await` yields; a blocking call inside `async def` freezes *every* request | Only across processes (the GIL) | A `def` endpoint runs in a threadpool; an `async def` that calls a sync library stalls the loop |
| Node | One event loop; a small libuv threadpool for file and DNS work | Callbacks and promises yield; CPU work blocks everything | Worker threads or cluster processes | A JSON.parse of 50 MB freezes the server; so does a synchronous crypto call |
| React | Browser main thread; render is synchronous per commit | Nothing "blocks" in the server sense, but long renders freeze the UI | Web Workers | A 10k-row table rendered without virtualisation |

**Look up carefully.**
- Thread-per-request vs event loop: draw both for ten requests where each waits 100 ms on a database.
- The GIL: what it actually protects, why threads still help for I/O in Python, why they do not help for CPU.
- `async def` vs `def` in FastAPI and what the framework does with each. This is the single most common Python performance bug we see.
- Node's event loop phases; what `setImmediate` vs `process.nextTick` vs a resolved promise mean for ordering.
- Java 21 virtual threads: what problem they solve, what "pinning" is, why they do not make the database faster.
- Backpressure: what happens when producers are faster than consumers, in each runtime.
- Little's Law: concurrency = throughput × latency. Use it to size a pool once and you will never forget it.

**Where it lives in our code.** The auction service's bidding and scraping schedulers (Java thread pools and `@Async`); the lead-gen research service (three parallel sources in LangGraph, `asyncio.gather`); the scraping service (Puppeteer, one event loop driving many browser pages); the dashboard UI's large tables.

**Self-check.** Why does adding threads to a Java service not help if the database pool is the bottleneck? What does a single synchronous `requests.get` inside an `async def` endpoint do to the other users of a FastAPI app? Why can a Node server serve thousands of connections on one thread but die on one big regex?

---

## M2 · Memory, lifecycle, and resource limits

**The invariant.** Every process has a heap, a limit, and a story about how unused memory comes back. Containers add a second limit the process may not know about. When the two disagree, the container is killed and the process never sees it coming.

**Rosetta.**

| | Memory model | Default limit behaviour | How you see it | Nuance |
|---|---|---|---|---|
| Java | Generational GC; heap sized by `-Xmx` or as a percentage of container memory | JVM reads cgroup limits since Java 10, but only if you let it | Heap dumps, GC logs, `jcmd` | A heap dump contains every secret in memory. Treat it like a password file |
| Python | Reference counting plus a cycle collector | No heap cap; grows until the container kills it | `tracemalloc`, RSS from `/proc` | Big NumPy or pandas objects live outside the GC's easy reach; leaked references in long-running agents |
| Node | V8 heap with a default cap around 2 to 4 GB depending on version | Build tools blow the cap before servers do | `--max-old-space-size`, heap snapshots | Our largest React app needs about 4 GB to *build*. Know where that flag is set and why |
| React | Browser memory per tab | Detached DOM nodes and listener leaks | DevTools memory tab | Every `addEventListener` in `useEffect` without a cleanup is a leak |
| Infra | cgroup memory limit per container | OOM-kill, exit code 137, silent restart | `docker stats`, container exit codes, Prometheus | An app that is "randomly restarting" with 137 is not random |

**Look up carefully.**
- What exit code 137 means and how to find it. Then what 143 means. Then why the difference matters for graceful shutdown.
- JVM container awareness: `MaxRAMPercentage`, and why setting `-Xmx` to the container limit is wrong (the JVM needs off-heap memory too).
- Graceful shutdown: SIGTERM, the grace period, in-flight requests, and what your framework does by default. Compose and Coolify both send SIGTERM and then SIGKILL.
- Connection and file-handle leaks as "memory" problems that show up as "too many open files".
- How to choose memory and CPU limits for a container, and what happens when a process's own idea of available memory differs from the container's.

**Where it lives in our code.** Every Dockerfile's `JAVA_OPTS` or `ENTRYPOINT`; the React build scripts; your teaching project's Compose file once you set limits in L7.

**Self-check.** A Java container with a 1 GB limit and `-Xmx1g` keeps dying. Why? A React build fails with "heap out of memory" on CI but not on your laptop. What is different? Why must a heap dump never be left on a server?

---

## M3 · Build, dependencies, and packaging

**The invariant.** Source plus declared dependencies plus a lockfile plus a toolchain version produces an artifact. If any of the four is not pinned, two people building the same commit get different results. Docker multi-stage builds exist to pin the toolchain and ship only the artifact.

**Rosetta.**

| | Manifest | Lockfile | Artifact | Build tool | Nuance |
|---|---|---|---|---|---|
| Java | `pom.xml` | None by default; Maven resolves ranges deterministically only if you avoid ranges | Fat JAR | Maven (use the wrapper) | Dependency scope (`compile`, `provided`, `test`) and transitive conflicts. Spring Boot's parent POM pins versions for you |
| Python | `pyproject.toml` | `uv.lock` | A virtualenv, or an image | `uv` | Virtualenvs are not optional. Native wheels differ per platform, which is why your Mac build may not match the Linux image |
| Node | `package.json` | `package-lock.json` | `dist/` or `.next/` | npm, Vite, webpack, Next | `npm ci` vs `npm install`; `devDependencies` vs `dependencies` in a production image |
| React | same as Node | same | static bundle | CRA (webpack) or Next | Environment variables are baked in at build time in CRA. Changing one means rebuilding, not restarting |
| Infra | `Dockerfile` | Base image digest | Image | Docker BuildKit | Layer caching order: copy the manifest and install deps *before* copying source, or every build re-downloads everything |

**Look up carefully.**
- Why lockfiles exist, and what "reproducible build" actually promises.
- Semantic versioning and why `^` and `~` in `package.json` are a decision, not a default.
- Multi-stage Docker builds: builder stage with the toolchain, runtime stage with only the artifact. Then read all of our Dockerfiles and find the one that gets it wrong.
- Docker layer caching and how the order of `COPY` lines changes build time from eight minutes to forty seconds.
- Build-time vs run-time configuration, and which of our frontends has which.
- Maven's dependency tree and how to find where a transitive version comes from.

**Where it lives in our code.** Every service root: its manifest, its Dockerfile, its Compose file if any.

**Self-check.** Why does a CRA app need a rebuild to change its API URL while a Spring app only needs a restart? What does the builder stage of a multi-stage Dockerfile contain that the runtime stage must not? Why is `npm install` in CI a bug?

---

## M4 · Configuration, environments, and secrets

**The invariant.** The same artifact must run in dev, staging and prod with different configuration. Configuration comes from outside the artifact, layered, with a defined precedence. Secrets are configuration with one extra rule: they never touch disk in the repo, never appear in logs, and never appear in chat.

**Rosetta.**

| | Mechanism | Precedence | Secrets | Nuance |
|---|---|---|---|---|
| Java | `application.properties` per profile, env vars, system properties | Command line beats env beats profile file beats default file | Read from mounted files under `/run/secrets` in prod | Property names map to env vars by a relaxed binding rule; learn it or you will spend an afternoon on a typo |
| Python | `.env` files via pydantic settings or `python-dotenv`, env vars | Env beats `.env` | Env vars injected by the platform | A committed `.env.example` with placeholders is the contract; a committed `.env` is an incident |
| Node | `process.env`, `.env.*` files in Next | Next has build-time and run-time variables with different prefixes | Env vars | `NEXT_PUBLIC_` variables are shipped to the browser. Anything else in them is public |
| React | Baked at build time (CRA) | None at runtime | Never. The browser is the attacker's machine | If a "secret" is in a React bundle, it is not a secret |
| Infra | Coolify environment variables per service; team-level shared variables referenced by services | Platform injects at container start | Coolify secrets, Docker secrets | Our cross-server IPs are centralised in shared variables so one change plus redeploy re-points every service. Understand why that design exists |

**Look up carefully.**
- The twelve-factor app, config section. Old, still right.
- Spring profiles and the exact order in which Spring Boot loads property sources.
- Why secrets should be files or env vars injected by the platform rather than values in a config file, and the trade-offs between the two.
- Secret rotation: what it means, why a token that has never been rotated is a liability, and what our security guidelines require.
- How to find out what configuration a running container actually has without printing secrets to a log.

**Where it lives in our code.** Each Java service's `@ConfigurationProperties` classes; each Python service's settings module; the configuration section of [09-platform-overview.md](09-platform-overview.md); your teaching project in L6.

**Self-check.** A value is set in both `application-prod.properties` and as an env var. Which wins? Why is a `NEXT_PUBLIC_` variable not a place for an API key? What is the one-line reason our cross-server addresses live in shared variables rather than in each service's config?

---

## M5 · Data modelling and persistence: ORMs, entities, and schema evolution

**The invariant.** Your code has objects; your database has tables; something maps between them and something evolves the tables over time. The mapping layer hides the SQL until it doesn't, and the evolution mechanism is the single most dangerous automated thing in the system.

**Rosetta.**

| | Mapping layer | Schema evolution | Nuance |
|---|---|---|---|
| Java | JPA with Hibernate; entities, repositories, lazy loading | Hibernate `ddl-auto`, or a migration tool such as Flyway | We have migration folders in some services that **never run** because the dependency is not wired. The real mechanism in production is `ddl-auto=update`. Verify, never assume |
| Python | SQLAlchemy or raw asyncpg; pydantic for the wire shape | Alembic | Separate your API schema (pydantic) from your DB schema (ORM); they drift apart on purpose |
| Node | Prisma (marketplace app), Mongoose (older code) | Prisma migrate | Prisma generates a client from the schema; forget to regenerate and types lie to you |
| Data | MySQL 8 primary; PostgreSQL via Supabase for newer services; MongoDB for audit trails | Per engine | Same SQL, different defaults: case sensitivity, boolean types, `RETURNING`, upsert syntax, isolation defaults |

**Look up carefully.**
- What an ORM saves you and what it hides: the N+1 problem, lazy vs eager loading, the "open session in view" anti-pattern.
- `ddl-auto` values and precisely what `update` can and cannot do (it adds; it never drops or narrows). Then why running it in production is a trade-off we currently accept.
- Migration tools: versioned files, checksums, why they are append-only, and what happens when two developers both create version 42.
- Entity identity and equality in JPA; why `equals` and `hashCode` on entities are a trap.
- MySQL vs PostgreSQL differences that break portable code: identifiers, `LIMIT` and pagination, upsert, JSON columns, boolean handling.
- Document stores vs relational: when MongoDB is right (audit logs, recordings) and when it is a mistake (anything you will join).

**Where it lives in our code.** The dashboard backend's entity and repository packages, the marketplace app's Prisma schema, and your teaching project's entities.

**Self-check.** Why does a migration folder in a repository prove nothing about whether migrations run? What does Hibernate do when you rename a field on an entity under `ddl-auto=update`? A list endpoint issues 1 query plus 200 more. Name the problem and two fixes.

---

## M6 · SQL depth: indexes, transactions, isolation, and connection pools

**The invariant.** The database is the shared, stateful, slowest part of every system we run. Everything else scales horizontally; this does not, at least not easily. Most "backend performance" problems are one of four things: a missing index, a too-wide transaction, a wrong isolation assumption, or a pool that is too small or too large.

**Rosetta.**

| | Pool | Transaction boundary | Nuance |
|---|---|---|---|
| Java | HikariCP; default 10 connections | `@Transactional` on service methods; proxy-based, so self-invocation does not start one | Pool size is a *ceiling on parallel DB work*, not a performance knob. Ten is often right |
| Python | asyncpg or SQLAlchemy pool | Explicit `async with session.begin()` | With `async`, a held connection across an `await` to an external API pins the pool |
| Node | Prisma's internal pool, or `pg.Pool` | `prisma.$transaction` | Interactive transactions time out by default; long ones are a design smell |
| Data | Server-side `max_connections` | Engine default isolation: MySQL InnoDB is REPEATABLE READ, PostgreSQL is READ COMMITTED | Same code, different anomalies on different engines |

**Look up carefully.**
- `EXPLAIN` and `EXPLAIN ANALYZE`: read a plan, spot a full table scan, understand why a covering index changes it.
- Index fundamentals: B-tree, composite index column order, the leftmost-prefix rule, why an index on a low-cardinality column is often useless, and the write cost of every index.
- ACID, then the four isolation levels and the anomalies each permits (dirty read, non-repeatable read, phantom, write skew). Then which one your engine defaults to.
- Deadlocks: how they arise from lock ordering, how the engine detects them, why the fix is usually consistent ordering rather than retries.
- Connection pool sizing: why a pool of 200 is slower than a pool of 10 on a 4-core database. Read about it until it is obvious.
- Pagination: `OFFSET` versus keyset pagination, and why `OFFSET 100000` is a problem.
- Slow query logging and how to turn it on in a sandbox.

**Where it lives in our code.** HikariCP settings, first in your teaching project (L4); the dashboard's search paths (Elasticsearch exists because some queries outgrew MySQL); the dashboard backend's repositories and `@Query` methods.

**Self-check.** You add an index on `(status, created_at)`. Which of these use it: `WHERE status = ?`, `WHERE created_at > ?`, `WHERE status = ? AND created_at > ?`? A Java method annotated `@Transactional` calls another `@Transactional` method on the same class. How many transactions? Why might raising the pool from 10 to 100 make the service slower?

---

## M7 · HTTP APIs and service-to-service calls

**The invariant.** A remote call can fail in more ways than a local one: it can time out, succeed without you knowing, or partially apply. Every outbound call needs a timeout, a retry policy that respects idempotency, and a decision about what to do when it fails. Every inbound endpoint needs a contract, validation, and consistent error shape.

**Rosetta.**

| | Inbound | Outbound | Nuance |
|---|---|---|---|
| Java | `@RestController`, validation annotations, `@ControllerAdvice` for errors | Feign clients (declarative), or `RestClient`/`WebClient` | Feign's default has *no* timeout in some configurations. Find ours and check |
| Python | FastAPI routers, pydantic models as the contract, dependency injection | `httpx` (async) | Use the async client inside `async def`; a sync client there is M1's bug |
| Node | Express routers, Next.js route handlers | `fetch`, axios | `fetch` has no timeout by default; use `AbortController` |
| React | n/a | axios or `fetch` with auth interceptors | Retries in the browser multiply load during an outage. Back off |

**Look up carefully.**
- REST conventions we actually follow vs the textbook: resource naming, status codes, error body shape. Then find the inconsistencies in our APIs; there are many, and knowing them is part of the job.
- Timeouts: connect vs read vs total, and why "no timeout" is the worst possible default.
- Retries: exponential backoff with jitter, the retry budget, and why you must only retry idempotent operations. Then what idempotency keys are and where we would need them (bidding).
- Circuit breakers and bulkheads: what problem they solve, when they are overkill.
- Webhooks: signature verification, replay protection, and why the receiver must be idempotent because senders retry.
- OpenAPI: generating a spec from FastAPI for free, and why our Java services mostly do not have one.
- CORS: what it protects, what it does not, why the browser is the one enforcing it.

**Where it lives in our code.** The dashboard backend's controller package and its Feign clients to the auction service; the auction service's webhook endpoint for shortlisted domains; the lead-gen orchestrator's calls to the research service and the mailing wrapper.

**Self-check.** A call times out. Did the operation happen? What must be true of an operation before you retry it? Why must a webhook receiver tolerate the same event twice?

---

## M8 · Asynchronous messaging and real-time delivery

**The invariant.** Sometimes you want to tell another service something without waiting for it, and without losing the message if it is down. That is a message broker. The broker gives you durability and decoupling, and in exchange it takes away exactly-once delivery and ordering guarantees you did not know you were relying on. Real-time delivery to browsers is a different problem with different tools.

**Rosetta.**

| | Producer | Consumer | Nuance |
|---|---|---|---|
| Java | `KafkaTemplate` | `@KafkaListener` with a consumer group | Auto-commit vs manual acknowledgement decides whether a crash re-delivers or loses |
| Python | `aiokafka` or `confluent-kafka` | Consumer loop in a background task | A slow consumer with a large `max.poll.interval` looks alive but is not |
| Node | `kafkajs` | Consumer group per service | Same semantics, different names |
| React | n/a | SSE (`EventSource`) for one-way; WebSocket for two-way | Reconnect logic is yours to write. SSE reconnects for free; WebSocket does not |

Our event topics (auction won, lost, outbid, ended, updated, watchlist, bid success and failure, scraper request and response) are the vocabulary between the auction service and the dashboard. Learn what each means in business terms before reading the code.

**Look up carefully.**
- Kafka fundamentals: topics, partitions, offsets, consumer groups, and why ordering is guaranteed only within a partition. Then how the partition key is chosen, and what happens to ordering if you choose badly.
- Delivery semantics: at-most-once, at-least-once, exactly-once, and why "at-least-once plus an idempotent consumer" is the practical answer almost everywhere.
- Consumer lag: what it is, how to see it in a Kafka UI, why it is the first metric you look at.
- Dead-letter topics and poison messages.
- Schema for events: why a JSON blob with no version field becomes a problem in month six.
- SSE vs WebSocket vs polling: when each is right. Our dashboard uses two of the three.
- Why message brokers and databases both exist: the outbox pattern and dual-write problem.

**Where it lives in our code.** The auction service's Kafka producers; the dashboard backend's listeners; the dashboard UI's WebSocket and SSE clients; the local Kafka UI you set up on Day 0.

**Self-check.** A consumer crashes after processing a message but before committing the offset. What happens on restart? Two events for the same auction land on different partitions. What guarantee did you lose? Why does an SSE client survive a server restart with no code while a WebSocket client does not?

---

## M9 · Authentication, authorisation, and identity

**The invariant.** Authentication answers "who is this"; authorisation answers "what may they do". Tokens carry identity across service boundaries; sessions keep it inside one. Every stack has a place where the check happens on every request, and the bugs live in the requests that skip it.

**Rosetta.**

| | Where the check lives | Token handling | Nuance |
|---|---|---|---|
| Java | Spring Security filter chain | JWT validation filter, OAuth2 resource server | The filter chain order matters. A permissive matcher before a strict one wins |
| Python | FastAPI dependencies on routes or routers | `Authorization` header parsed in a dependency | Forgetting the dependency on one router is the classic hole |
| Node | Express middleware, Next.js middleware | JWT verify | Middleware order, again |
| React | MSAL (Azure AD) or Google OAuth in the browser; token attached by an axios interceptor | Access token in memory; refresh handled by the library | Tokens in `localStorage` are readable by any XSS. Know the trade-off you are making |

We run three identity providers across services: Azure AD, Google, and a custom company JWT. Understanding why three exist (history, different audiences) is part of understanding the system.

**Look up carefully.**
- OAuth2 and OpenID Connect: the authorisation-code flow with PKCE, step by step, and which party sees which secret. Draw it.
- JWT anatomy: header, claims, signature; what validation must check (signature, issuer, audience, expiry) and the "alg: none" class of bugs.
- Sessions vs tokens: revocation, statelessness, and why "logout" is hard with JWTs.
- RBAC vs ABAC; where roles live in our systems.
- CSRF: what it is, why token-in-header APIs are mostly immune, why cookie sessions are not.
- Password storage if you ever must: bcrypt or argon2, and why you should almost never must.
- Service-to-service auth: how the auction service knows a webhook came from the dashboard and not from the internet.

**Where it lives in our code.** Each Java service's security configuration class; the MSAL setup in the React apps; the custom JWT utilities in the legacy backend.

**Self-check.** In the authorisation-code flow, which component ever sees the client secret? What must a JWT validator check beyond the signature? Why is a permissive `permitAll` matcher placed before an `authenticated` one a vulnerability?

---

## M10 · Errors, logging, metrics, and traces

**The invariant.** A system you cannot observe is a system you debug by guessing. Logs tell you what happened, metrics tell you how much and how often, traces tell you where the time went across services. The three must share identifiers (a service name, a request or trace ID) or they cannot be joined.

**Rosetta.**

| | Logging | Metrics | Tracing | Nuance |
|---|---|---|---|---|
| Java | SLF4J with Logback; JSON in prod | Micrometer to Prometheus via Actuator | OpenTelemetry agent or starter | MDC carries the trace ID into every log line for free once configured |
| Python | `logging` or `structlog`; JSON | `prometheus_client` | OpenTelemetry SDK | Uvicorn's access log and your app log are two different loggers |
| Node | `pino` | `prom-client` | OpenTelemetry SDK | Console logging in production is slow and unstructured |
| React | Console plus an error boundary; optionally an error reporter | Web vitals | n/a | Errors in the browser are invisible to you unless you ship them somewhere |
| Infra | Promtail ships every container's stdout to Loki automatically | Prometheus scrapes `/metrics`; a service opts in with one environment variable | Tempo receives OTLP | Our conventions are summarised in [09-platform-overview.md](09-platform-overview.md). Read them before you add a service |

**Look up carefully.**
- Structured logging: why JSON lines beat free text, what fields every line needs, what must never be logged (M4 secrets, personal data).
- Log levels and what each one means operationally. `ERROR` means a human should look. If everything is `ERROR`, nothing is.
- The four golden signals: latency, traffic, errors, saturation. Then the RED and USE methods.
- Prometheus data model: counters, gauges, histograms; label cardinality and why a user ID as a label kills the server.
- Our metric naming contract and mandatory labels (summarised in the platform overview). Every job maps to a named failure archetype; learn them.
- Distributed tracing: spans, context propagation across HTTP headers and Kafka message headers, sampling.
- Alerting: symptom-based over cause-based, and why you verify every deploy yourself instead of assuming an alert would fire.
- Exceptions: checked vs unchecked in Java, exception groups in Python, error boundaries in React, and the universal rule that catching and swallowing is worse than crashing.

**Where it lives in our code.** The observability section of the platform overview; each service's logging config; your teaching project in L8; Grafana (after Day 8).

**Self-check.** Why is a per-user label on a Prometheus metric a disaster? How does a trace ID get from an HTTP request into a Kafka message and out the other side? If a deploy breaks a service tonight, who finds out and how?

---

## M11 · Testing

**The invariant.** A test is a claim about behaviour that can fail. Unit tests check one unit with everything else faked; integration tests check units together with real infrastructure; end-to-end tests check the whole system through its real interface. Cost and confidence both rise as you go up. Most of our services are under-tested; changing that is a contribution you can make in week three.

**Rosetta.**

| | Unit | Integration | Nuance |
|---|---|---|---|
| Java | JUnit 5, Mockito, AssertJ | `@SpringBootTest`, Testcontainers for a real MySQL or Kafka | Slice tests (`@WebMvcTest`, `@DataJpaTest`) load only part of the context and are much faster |
| Python | pytest, fixtures | `httpx.AsyncClient` against the app, a test database | `pytest-asyncio` modes trip everyone once |
| Node | Vitest or Jest | Supertest | Mocking modules vs mocking network |
| React | React Testing Library | Playwright or Cypress | Test behaviour a user sees, not component internals |

**Look up carefully.**
- The test pyramid and its critics; the "testing trophy" argument for more integration tests.
- Test doubles: stub, mock, fake, spy; why over-mocking produces tests that pass while production fails.
- Testcontainers: real databases in tests, why it beats H2 for MySQL-specific behaviour.
- Contract tests between services: what breaks when the dashboard changes a DTO the auction service consumes.
- Flaky tests: time, randomness, ordering, shared state; how to make a test deterministic.
- Testing LLM-backed code: golden datasets, evals, and why exact-match assertions on model output are wrong.

**Where it lives in our code.** Each service's `test` folder. Note how thin many are. That is honest and it is an opportunity.

**Self-check.** Why can a service with 90 percent unit coverage still fail on deploy? When is an in-memory database an acceptable substitute for the real one, and when is it a lie? How would you test a function whose output comes from an LLM?

---

## M12 · Frontend rendering, state, and data fetching

**The invariant.** A UI is a function of state. The framework decides when to recompute that function; you decide where state lives and how it gets there. Most frontend bugs are state in the wrong place or effects that run at the wrong time. Most frontend performance problems are rendering too much, too often.

**Rosetta.**

| | Rendering model | State | Data fetching | Nuance |
|---|---|---|---|---|
| CRA + MUI (our dashboards) | Client-side only; one big bundle | React state, context, occasionally a store | axios in effects or a query library | Bundle size and initial load are the cost of client-only rendering |
| Next.js (marketplace and name.ai frontends) | Server components by default, client components opt in; SSR and static generation | Same React model, plus server-side data | `fetch` in server components; route handlers | The server and client boundary is a new mental model; "use client" is not a performance hint, it is a semantic one |

**Look up carefully.**
- The React render cycle: render, reconcile, commit; why a component re-renders; what `key` actually does.
- Hooks rules and why they exist. `useEffect` dependencies, the cleanup function, and why effects are for synchronising with the outside world, not for computing derived state.
- State placement: local, lifted, context, external store; the cost of each. Then server state vs client state and why a query library exists.
- Memoisation: `useMemo`, `useCallback`, `React.memo`; when they help and when they are noise.
- Virtualisation for large lists, since our dashboards show thousands of domains.
- SSR vs CSR vs static generation vs streaming; hydration and its errors.
- Material UI theming and the `sx` prop; how to not fight the component library.
- Accessibility basics: semantics, focus, keyboard. Not optional in internal tools either.

**Where it lives in our code.** The dashboard UI (largest, client-only, WebSockets, rich editors, maps); the Next.js marketplace app.

**Self-check.** A child re-renders every time its parent does even though its props "did not change". Why, and what is the fix? Why must an effect that subscribes return a cleanup? What is wrong with computing a filtered list inside `useEffect` and storing it in state?

---

## M13 · AI systems: LLM applications and agents

**The invariant.** A large language model is a function from text to text that is expensive, slow, non-deterministic, and occasionally wrong with confidence. Everything in "AI engineering" is scaffolding to make that function useful: structured outputs, tools, retrieval, memory, orchestration, evaluation, and cost control. Treat the model like a flaky external API with a very good interface.

**Rosetta.**

| | Orchestration | Structured output | Tools | Nuance |
|---|---|---|---|---|
| Python | LangGraph state machines (lead-gen, negotiation), LangChain | pydantic models as the output schema | Tool calling with typed schemas | A graph with cycles needs a termination condition you can defend |
| Java | Direct HTTP to model APIs from the orchestrator | Jackson to a DTO | Rare | The orchestrator decides *when* to call AI; the Python services decide *how* |
| Node | Direct SDK calls in the marketplace app | zod schemas | SDK tool definitions | Streaming to the browser and cost per request |

**Look up carefully.**
- Tokens, context windows, and pricing. Estimate the cost of a pipeline before you run it on ten thousand domains.
- Prompt structure: system vs user roles, few-shot examples, why "be concise" is not engineering and a schema is.
- Structured outputs and tool calling: how they work mechanically, how to validate, what to do on a malformed response.
- Retrieval-augmented generation: chunking, embeddings, vector search, and the many ways it quietly retrieves the wrong thing.
- Agent loops: the plan-act-observe cycle, state machines vs free-running loops, guardrails, max steps, and human-in-the-loop points.
- Evaluation: golden sets, LLM-as-judge and its biases, regression testing prompts, and why you cannot ship a prompt change without an eval.
- Caching, rate limits, and retries against model APIs; idempotency of expensive calls.
- Prompt injection and data exfiltration when a model reads untrusted text (scraped pages, inbound emails). Our lead-gen pipeline reads both.

**Where it lives in our code.** The lead-gen research service (three parallel sources), the negotiation agent, the price prediction agent, the portfolio valuation advisor, and the orchestrator that calls them.

**Self-check.** A LangGraph node can loop back to itself. What stops it? Your pipeline costs a fraction of a cent per domain; the portfolio has half a million domains. Do the arithmetic. A scraped web page contains "ignore previous instructions". What happens in your pipeline, and where is the boundary that should stop it?

---

## M14 · Containers, networking, and deployment

**The invariant.** A container is a process with its own view of the filesystem, network and resources, sharing the host kernel. Container networking is a set of virtual networks with their own DNS. Publishing a port punches a hole from the host into a container, and it does so *below* the host's firewall tooling in a way that surprises almost everyone the first time. Deployment is the act of replacing a running container with a new one without anyone noticing.

**Rosetta (all stacks share this column).**

| Concern | Mechanism | Nuance |
|---|---|---|
| Isolation | Namespaces and cgroups | Not a VM. A kernel exploit escapes it |
| Networking | Bridge networks; containers on the same network resolve each other by service name | Compose creates a network per project; two projects cannot see each other by default |
| Port publishing | `-p host:container` inserts iptables rules in Docker's own chains | These rules are evaluated **before** UFW's. UFW saying "denied" does not mean denied. The fix is the `DOCKER-USER` chain |
| Persistence | Volumes and bind mounts | Data in the container layer dies with the container |
| Health | `HEALTHCHECK` and readiness endpoints | Coolify and Compose use them to decide when traffic can flow |
| Deployment | Coolify: a GitHub push triggers a webhook, Coolify builds the image and swaps the container | Not GitHub Actions `workflow_dispatch`. Know the actual trigger |
| Edge | Cloudflare for DNS, TLS termination, CDN, proxying | Origin must still be protected; Cloudflare in front does not mean your origin IP is hidden if it was ever public |
| Private network | A private mesh VPN between servers and engineers; no public SSH | Access is tied to identity and logged; nothing administrative is exposed to the internet |

**Look up carefully.**
- Linux namespaces and cgroups, briefly; enough to explain why a container is not a VM.
- Docker bridge networking, embedded DNS, and how `localhost` inside a container is not your laptop.
- iptables chain traversal: `PREROUTING`, `FORWARD`, the `DOCKER` and `DOCKER-USER` chains, and exactly why UFW rules do not apply to published ports. Then binding to `127.0.0.1:port` as the simpler fix for services that should never be public.
- Zero-downtime deploys: health checks, rolling replacement, draining, and what the grace period from M2 has to do with it.
- Reverse proxies and TLS termination; what Coolify's proxy does for you and what it does not.
- DNS basics and Cloudflare specifics: proxied vs DNS-only records, and what each exposes.
- Volumes vs bind mounts; backups of volumes; what happens to a database when its container is recreated without a volume.

**Where it lives in our code.** Every Dockerfile and Compose file; the deployment section of [09-platform-overview.md](09-platform-overview.md); L7 and L11.

**Self-check.** A service binds to `0.0.0.0:9999` inside a container and the Compose file publishes 9999. UFW denies 9999. Is the port reachable from the internet? Two Compose projects each run a `db` service. Can project A's app reach project B's `db` by name? What triggers a production deploy here?

---

## M15 · Security fundamentals

**The invariant.** Security is a property of the whole system, decided by its weakest point, and the weakest point is usually a human shortcut: a shared password, a token never rotated, a port left open, a file left on disk. Defence in depth means assuming each layer will fail and making the next one hold anyway.

This module is thin on purpose. The authority is [11-security-rules.md](11-security-rules.md), which you read on Day 0 and again on Day 8. [05-public-postmortems.md](05-public-postmortems.md) shows why rules like these exist.

**Look up carefully.**
- The OWASP Top 10, as a vocabulary. Injection, broken access control, security misconfiguration, and secrets exposure are the four that matter most for services like ours.
- Least privilege for humans, services, and tokens.
- SSH hardening: keys only, no root login, no password authentication, and why closing public SSH entirely is better still.
- Secret hygiene: where secrets may live, how they are rotated, how you find out if one leaked (secret scanning in the repository).
- Attack surface: every published port, every public endpoint, every dependency. How to enumerate yours.
- Supply chain: lockfiles, pinned base images, dependency scanning.
- Incident response basics: contain, preserve evidence, eradicate, recover, learn. The order matters.

**Self-check.** Name three things that must be true before a secret is considered "handled". Why is closing public SSH safer than hardening it? What is the first thing you do when you suspect a server is compromised, and what is the first thing you must not do?

---

## M16 · Git, code review, and delivery

**The invariant.** The repository is the shared memory of the team. Commits are the unit of explanation; pull requests are the unit of review; the main branch is always deployable. Everything about our workflow follows from those three.

**Look up carefully.**
- Git as a graph: commits, refs, branches as pointers. Once this clicks, `rebase`, `reset`, and `reflog` stop being scary.
- Branching: short-lived feature branches off main; why long-lived branches rot.
- Commit messages: a subject that says *what* and a body that says *why*. The diff already says how.
- Rebase vs merge, and which we use where. Interactive rebase to tidy before review.
- Pull request hygiene: small, one concern, a description that lets a reviewer verify without asking, screenshots for UI.
- How to review: read for behaviour and risk first, style last; ask questions rather than issue orders; approve when you would be comfortable being paged for it.
- Undoing things safely: `revert` over `reset` on shared branches; `reflog` when you think you lost work.
- Our deploy trigger: a push to the deploy branch means production changes. Know which branches are watched before you push.

**Self-check.** Explain `git rebase` to someone who knows only `commit` and `merge`. What is wrong with a pull request titled "fixes"? Why is `git push --force` to a shared branch a team incident rather than a personal mistake?

---

## M17 · Working inside a large existing codebase

**The invariant.** You will spend most of your career reading code you did not write. Navigating it is a skill with techniques, and none of your projects so far taught it because they were yours and they were small. The goal is to go from "a button in the UI" to "the table it writes" in under fifteen minutes in a codebase you have never seen.

**Techniques.**
- Start from the edge you can see: a URL, a button label, a log line, an error string. Grep for it literally. Strings are the most reliable entry point into any codebase.
- Follow the request inward: route, handler, service, repository, table. Write each hop down as you go.
- Read the tests first if there are any. They are the only documentation that is verified.
- Read the manifests and configuration before the code. They tell you the dependencies, the entry point, the profiles, and the ports.
- Use the project's own maps: each service has a `CLAUDE.md`. Use it, then verify it against the code. Documentation drifts, and a doc that says something is "unused" or "stale" is a claim to check, never a fact to act on.
- Build a call graph for one feature on paper. Not the whole system. One feature, end to end.
- Change something trivial and prove you can see the change. A log line, a response field. It proves the build, the run, and your understanding of the entry point in one move.
- Note every convention you spot (naming, package layout, error handling, how DTOs are shaped). Then follow them even when you dislike them. Consistency beats local taste in shared code.
- Note every inconsistency too. Those are future cleanup tickets and they are yours to propose.

**Look up carefully.**
- Reading code with an IDE: go to definition, find usages, call hierarchy. Learn the keyboard shortcuts in Cursor for these; they matter more than the AI features for this skill.
- `rg` (ripgrep) flags: type filters, context lines, multiline.
- How to read a Spring application's startup log to learn what it wired.
- Chesterton's fence: do not remove a thing until you know why it was put there.

**Self-check.** Given only a button label from a screenshot, list the steps to find the database table it affects. What do you do when the `CLAUDE.md` disagrees with the code? Why do you change a log line before you change any logic?

---

## M18 · Time, scheduling, and background work

**The invariant.** Anything that happens "later" or "every N minutes" is a scheduler, and schedulers have three enemies: time zones, overlapping runs, and the machine restarting mid-job. Our core business runs on time: auctions end at instants, drops happen at instants, bids must land before instants. Getting time wrong here is losing money.

**Rosetta.**

| | Scheduler | Overlap protection | Nuance |
|---|---|---|---|
| Java | `@Scheduled` with cron expressions; `TaskScheduler` pools | None by default; two instances of the service run the job twice | The default scheduler pool size is one. A slow job delays every other job |
| Python | APScheduler, Celery beat, or a loop with `asyncio.sleep` | None by default | A `while True` loop with `sleep` drifts; it does not run "every minute" |
| Node | `node-cron`, or platform cron | None by default | Same drift, same overlap |
| Infra | Container restart policies; Coolify restarts | n/a | A job that was mid-flight when the container was replaced simply did not finish. Design for it |

**Look up carefully.**
- Time zones and offsets: store UTC, display local, never do arithmetic on local time. Then why "midnight" is ambiguous twice a year.
- Cron syntax, and the difference between "every 5 minutes" and "at :00, :05, :10" under load.
- Idempotent jobs: a job that can be run twice safely is a job you never have to worry about.
- Distributed locks and leader election when more than one instance runs the same scheduler; why a database row lock is often enough.
- Clock skew between servers and why NTP matters when you are bidding against a deadline.
- Retries for jobs vs retries for requests, and dead-letter handling for jobs that keep failing.
- Precision: the difference between "the auction ends at 14:00:00" and "our poll noticed at 14:00:03".

**Where it lives in our code.** The auction service's monitoring and bidding schedulers, the EPP dropzone capture timing, the lead-gen campaign cadence.

**Self-check.** A job runs every minute and sometimes takes ninety seconds. What happens? Two replicas of a service both have `@Scheduled` on the same method. What happens? Why is storing an auction end time in local time a bug even if the server never moves?

---

## M19 · Caching and rate limiting

**The invariant.** A cache trades freshness for speed and has exactly two hard problems: invalidation and stampedes. A rate limiter protects a resource from too many callers and has one hard problem: deciding what to do with the excess. Both are usually Redis, and both are places where a small bug looks like a big outage.

**Look up carefully.**
- Cache-aside vs read-through vs write-through; TTLs; what "eventually consistent" means for your users.
- Invalidation strategies and why "just set a TTL" is the pragmatic answer far more often than event-driven invalidation.
- Cache stampede (thundering herd) and the fixes: locking, early refresh, request coalescing.
- Redis data structures beyond strings: hashes, sorted sets, and what they are good for; Redis persistence modes and what you lose on restart.
- Rate limiting algorithms: fixed window, sliding window, token bucket; where to enforce (edge, gateway, service) and what to return (429 with `Retry-After`).
- Registrar and third-party API quotas that we live under, and how the scraping service and lead-gen pipeline respect them.
- HTTP caching headers as the cache you get for free if you set them.

**Self-check.** A cached value is wrong for five minutes after an update. Is that a bug? It depends on what. On what? Fifty requests miss the cache at the same instant for the same key. What happens to the database, and what is the fix?

---

## Appendix · Adding a column

When you are put on a stack this file does not cover, before writing any code, produce a one-page document with a row for each module above answering "how does this stack do it, and where does it hurt". Missing answers are fine; they are your first week's reading list. Hand the page to Yash and to the interns after you. That page becomes the next column of this matrix.
