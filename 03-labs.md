# 03 · Labs

Every lab has the same shape: **Goal**, **Setup**, **Steps**, **Observe** (what you must see with your own eyes), **Answer** (questions in writing, in your own words), and **Deliverable**. A lab is done when the deliverable exists and the answers are yours.

Each lab is marked:

- **Read:** needs only repository access to our services. You trace and reason; nothing of ours runs.
- **Run:** happens entirely on your laptop, on your teaching project from [10-teaching-project.md](10-teaching-project.md) or a small standalone setup.

Some labs have both parts. No lab asks you to run a company service.

Labs marked **No AI first** must be attempted without AI tools before you use them. They build a muscle the tools cannot build for you.

A habit for every Read lab: configuration values live outside the code in production. When a trace leads you to a configuration key, record the key's name and move on. You never need its value.

---

## L1 · Legacy navigation · Read · No AI first

**Goal.** Go from a visible UI label to the database table behind it, in a codebase you have never read. (M17)

**Setup.** The dashboard UI and dashboard backend repositories, cloned. `rg` installed. Cursor's AI features off for the first attempt.

**Steps.**
1. Work in pairs. Your partner browses the dashboard UI code and picks the visible text of a button or menu item, such as a label inside a component. They give you only the text, not the file.
2. Grep the UI for the text. Find the component, the event handler, and the request it sends: method, path, body shape.
3. Grep the backend for the path. Follow it: controller method, service, repository, entity, table name.
4. Write each hop as one line: file, symbol, what it does, what it calls next.
5. Time yourself. Then do a second label with Cursor's AI allowed, and time that too.

**Observe.** Which hops were hard to find, and why: indirection, naming, generated code, configuration? Where did the AI help, and where did it guess wrong?

**Answer.** How many hops from click to table? Which naming conventions did you discover? Where did the code disagree with the service's `CLAUDE.md`?

**Deliverable.** Two hop lists with timings, and a five-line comparison of the two methods.

---

## L2 · The golden path · Read

**Goal.** Follow one real business action through every layer of our system by reading: the dashboard UI, the dashboard backend, the webhook to the auction service, Kafka, the consumer, and Telegram. (M7, M8, M17)

**Setup.** The three repositories. [09-platform-overview.md](09-platform-overview.md) and [09b-domain-auction-lifecycle.md](09b-domain-auction-lifecycle.md) open.

**Steps.**
1. Start in the dashboard UI at the acquisition shortlist action. Record the request it sends.
2. In the dashboard backend, follow the request to the point where it calls the auction service. Identify the HTTP client used, what happens if the call throws, and whether a timeout is set in code.
3. In the auction service, find the receiving endpoint. Identify how it decides the caller is allowed. Follow what it saves.
4. Find the Kafka events the auction service publishes on this path or later for the same domain. Record topic, key, and payload shape.
5. Find the consumers of those topics in the dashboard backend. Record what they do.
6. Find where a Telegram notification is produced and what triggers it.
7. Draw a sequence diagram. Label every arrow with its mechanism. Mark every point that can fail, and what happens when it does.

**Observe.** Where the path crosses a service boundary. Where it becomes asynchronous. Where a failure would be silent.

**Answer.** If the auction service is down when the user clicks, what does the user see? If Kafka is down, what is lost and what is only delayed? Where is the first place you would add a log line to debug "I shortlisted a domain and nothing happened"? Which of the stage-specific ceilings from the auction lifecycle document travel with a shortlisted domain, and where do you see each one in the code?

**Deliverable.** The sequence diagram and annotated hop list, reviewed by your buddy. This is the most important deliverable of the two weeks.

---

## L3 · Three runtimes, one worker · Run

**Goal.** Feel the difference between thread-per-request, asyncio, and the Node event loop, with numbers. (M1)

**Setup.** A tiny local HTTP server that waits 200 ms and then responds; the fake registrar from your teaching project works if you have it. Java 21, Python 3.11 or newer, Node 20.

**Steps.**
1. Write a worker in each runtime that makes 100 calls to the slow server and reports total wall time: Java with a fixed pool of 10 threads, then with virtual threads; Python with `asyncio.gather` over `httpx.AsyncClient`; Node with `Promise.all` over `fetch`.
2. Record the wall times.
3. Sabotage each one. Java: pool size 1. Python: replace the async client with a synchronous `requests.get` inside the coroutine. Node: add a synchronous 200 ms CPU loop before each call.
4. Record the wall times again.

**Observe.** Which sabotage hurt most, and why. In the Python case, run a second task that prints a heartbeat every 50 ms and watch what happens to it during the synchronous call.

**Answer.** Explain concurrency versus parallelism using your numbers. Why did virtual threads change the Java result? What rule about `async def` in FastAPI falls out of the Python result?

**Deliverable.** A table of six timings and one paragraph per runtime.

---

## L4 · Exhaust the pool · Run

**Goal.** See what connection pool exhaustion looks like from outside and inside, then size a pool with arithmetic. (M1, M6)

**Setup.** Your teaching project's `auction-api` with data from P4. A load tool: `hey`, `oha`, `k6`, or a short script.

**Steps.**
1. Set HikariCP's maximum pool size to 2 and its connection timeout to 5 seconds.
2. Pick an endpoint that reads from the database. Fire 20 concurrent requests, 10 rounds.
3. Read the application log and the Hikari metrics from Actuator.
4. Set the pool to 10 and repeat. Then 50, and repeat.
5. Add a 300 ms sleep inside the transactional service method. Repeat at pool size 10. Then move the sleep outside the transaction and repeat.

**Observe.** The exact error a caller gets on exhaustion. Latency at each pool size. Whether 50 was faster than 10 on your laptop, and why or why not. The difference the sleep's position made.

**Answer.** Using Little's Law and your measured latency, what pool size does this endpoint need for 100 requests per second? Why is slow work inside a transaction worse than the same work outside it? How would you tell pool exhaustion apart from "the database is slow"?

**Deliverable.** A latency table by pool size, the error text, your sizing calculation.

---

## L5 · Schema and plans · Run, plus Read

**Goal.** Move from "the ORM handles it" to reading and shaping the SQL that actually runs. (M5, M6)

**Setup.** Teaching project P4: SQL logging on, tens of thousands of seeded auctions and bids.

**Steps.**
1. Hit three list or search endpoints. Capture the SQL each emits.
2. Run `EXPLAIN` on each. Record access type, estimated rows, and index use.
3. Find a full table scan. Design an index. Add it. `EXPLAIN` again.
4. Provoke an N+1: a list endpoint that emits one query plus one per row. Count the queries. Fix it with a fetch join or a projection. Count again.
5. Open a transaction in one SQL client session, update a row, and do not commit. In a second session, read the same row. Commit, then read again.
6. **Read part:** in the dashboard backend, find two entity relationships that would cause N+1 if a list endpoint iterated them, and one repository method that you think needs an index. Justify each from the code alone.

**Observe.** Plans before and after. Query counts before and after. What the second session saw, and when.

**Answer.** Why did the ORM produce the N+1 without warning you? What does your new index cost on every write? Which isolation level is MySQL using, and which anomaly did you just demonstrate or fail to demonstrate?

**Deliverable.** Plans before and after, the N+1 counts, the isolation experiment log, and the Read-part findings.

---

## L6 · Configuration design · Run, plus Read

**Goal.** Know every place a value can come from, in order, and design configuration that is safe to deploy. (M4)

**Setup.** Your teaching project.

**Steps.**
1. Give `auction-api` three profiles: `dev`, `test`, and `prod`. Put registrar URL, timeouts, and pool size in typed `@ConfigurationProperties` records with validation, so the app refuses to start on a missing or invalid value.
2. Make `prod` take its database password from a mounted file and everything else from environment variables. Run it that way in Compose.
3. Set one property in the profile file, as an environment variable, and as a command-line argument. Prove which wins, and in what order.
4. Write `.env.example` listing every variable with a placeholder and a one-line description. Make sure `.env` is ignored by Git.
5. Do the same exercise for `notifier` in its own stack, when you build it.
6. **Read part:** in one of our Java services, find its `@ConfigurationProperties` classes and list the configuration key **names** each one expects, grouped by what they configure. Values are not needed and not part of this lab.

**Observe.** How many places one setting can come from. What the fail-fast startup error looks like.

**Answer.** Why should a platform inject secrets rather than a repository containing them? Give an example of an environment variable that binds to a nested property. What one change would make our service's configuration easier to reason about?

**Deliverable.** Profile files, `.env.example`, the precedence proof, and the Read-part key map.

---

## L7 · Build and run it the production way · Run

**Goal.** Own the path from source to running container, including its failure modes. (M2, M3, M14)

**Setup.** Teaching project `auction-api`. Docker.

**Steps.**
1. Write a Dockerfile that copies the whole project and builds. Time a build. Change one line of Java and time it again.
2. Rewrite it as a multi-stage Dockerfile that resolves dependencies before copying source. Time two rebuilds after a one-line change.
3. Run the image with a 256 MB memory limit and no JVM memory setting. Load it. Watch it die. Find the exit code.
4. Configure the JVM to use a percentage of container memory. Run again. Explain why this is right and why a fixed `-Xmx` equal to the limit is wrong.
5. Send SIGTERM while a slow request is in flight. Does it complete? Enable graceful shutdown in Spring and compare.
6. Add a health check to the Compose service so dependent services wait for it.

**Observe.** Build times before and after. The exit code and the last log lines on out-of-memory. Whether in-flight requests survived shutdown.

**Answer.** Why is layer order the biggest build-time lever? What do exit codes 137 and 143 each mean? Why is `-Xmx` equal to the container limit a bug?

**Deliverable.** Build timings, out-of-memory evidence, shutdown results, and the final Dockerfile.

---

## L8 · Make it observable · Run

**Goal.** Add metrics, structured logs, and a dashboard following our conventions. (M10)

**Setup.** Teaching project, with Prometheus and Grafana added to Compose. [09-platform-overview.md](09-platform-overview.md) section 9 open.

**Steps.**
1. Expose Actuator's Prometheus endpoint and get Prometheus scraping it.
2. Decide which failure archetypes apply to `auction-api`. Likely: liveness, freshness of the registrar sync, error rate of registrar calls, and dependency on the registrar.
3. Export one metric per archetype, named by the `<service>_<subject>_<unit>` contract. For the sync, export both the last success timestamp and the expected interval.
4. Switch logging to JSON, and make one request's log lines share a trace or request id.
5. Build a Grafana dashboard with request rate, error rate, sync freshness, and registrar dependency panels.
6. Write the alert rules you would add, in Prometheus rule syntax, with `service` and `severity` labels. Write the one dimensionless freshness rule from the overview.
7. Add a label with a high-cardinality value, such as a user id, generate traffic, look at the series count, then remove it.

**Observe.** Your sync freshness ratio when the fake registrar rate-limits you. The series count explosion in step 7.

**Answer.** Why does the naming contract require `service` and `severity` labels? Is each of your alerts a symptom or a cause? How would you know tonight that your deploy broke something?

**Deliverable.** Compose file, dashboard screenshot, alert rule file, a sample of log lines.

---

## L9 · Kafka by hand · Run, plus Read

**Goal.** Feel consumer groups, offsets, redelivery, and idempotency, then read our real topics with that understanding. (M8)

**Setup.** The Kafka and Kafka UI from your teaching project. A producer and consumer in any language.

**Steps.**
1. Create a topic with 3 partitions. Produce 30 messages with keys `a`, `b`, and `c`. See which partition each key lands in.
2. Start one consumer in a group; it owns all partitions. Start a second in the same group; watch the rebalance. Start a fourth; watch one sit idle.
3. Make a consumer process a message and crash before committing. Restart it. Watch the redelivery.
4. Make the consumer idempotent by recording processed message ids and skipping duplicates. Crash it again. Watch the difference.
5. Produce a message the consumer cannot parse. Watch what happens to the partition behind it. Add a dead-letter topic.
6. **Read part:** in the auction service and the dashboard backend, for each of our Kafka topics, record the producing class, the key, the payload type, the consuming class, and whether the consumer looks idempotent. Use the Java code only.

**Observe.** Ordering within a partition versus across partitions. The moment of redelivery. What a poison message does to everything queued behind it.

**Answer.** Why is the partition key a business decision? What does at-least-once delivery require of a consumer? Which of our real consumers would misbehave if a message arrived twice, and what would the symptom be?

**Deliverable.** An experiment log with Kafka UI screenshots, and the topic inventory table.

---

## L10 · Race the bids · Run

**Goal.** Create a real race condition in a Spring app, fix it several ways, and understand which fix holds across two instances. (M1, S7)

**Setup.** Teaching project P7: the closing sprint with a shared total budget, and the fake registrar with latency on.

**Steps.**
1. Store the remaining budget as a plain `long` field on a singleton service. Run the sprint with 50 concurrent bids against a budget that allows 20. Repeat 1,000 times in a test. Count how often the budget is overspent.
2. Fix it with `synchronized`. Measure throughput.
3. Fix it instead with `AtomicLong` and compare-and-set. Measure throughput.
4. Write a check-then-act bug on a `ConcurrentHashMap` of per-auction bid counts, prove it breaks, and fix it with `compute` or `merge`.
5. Run the sprint on: a cached thread pool, a fixed pool of 10, and virtual threads. Record wall time and peak thread count for each.
6. Run it with `CompletableFuture.supplyAsync` **without** an executor. Find out which threads ran the work. Then make one bid throw and find out where the exception went.
7. Add a timeout with `orTimeout` so one slow registrar call cannot stall the sprint.
8. Start **two** instances of `auction-api` running the sprint against the same database. Watch your in-memory fix fail. Fix it at the database with optimistic locking or a conditional update.

**Observe.** Overspend counts before and after each fix. Thread counts per executor. Where the swallowed exception went. The two-instance failure.

**Answer.** Why did the singleton field race when a Node developer would expect it not to? When would you choose `synchronized` over an atomic? Why does no in-JVM fix work across two instances?

**Deliverable.** A table of overspend counts and throughput per fix, executor comparison numbers, and the two-instance explanation.

---

## L11 · Docker networking and the firewall that lied · Run

**Goal.** Prove with your own hands that a host firewall does not protect Docker-published ports, then fix it correctly. (M14)

**Setup.** A Linux environment with root: WSL 2 with systemd, or a small VM. Docker, `ufw`, and `iptables` inside it.

**Steps.**
1. Run a small HTTP container with `-p 8081:80`. Curl it from the host. It works.
2. Enable UFW with default deny incoming, allowing only SSH. Confirm UFW reports 8081 as blocked.
3. From another machine or network namespace that can reach this host, curl port 8081. It still works.
4. List the `FORWARD` and `DOCKER-USER` chains. Trace the packet's path and find why UFW's rules were never consulted.
5. Add a rule to `DOCKER-USER` that drops outside traffic to that port. Curl again. It fails.
6. Remove that rule. Instead run the container with `-p 127.0.0.1:8081:80`. Curl from outside. It fails. Explain why binding to loopback is the simpler fix for anything that should never be public.

**Observe.** UFW confidently reporting a block that did nothing. The exact chain where the packet was accepted.

**Answer.** Why does Docker insert its rules where it does? In your teaching project, which services should bind to loopback only? What one-line rule would you add to a deployment checklist?

**Deliverable.** Chain listings before and after, and the checklist rule.

---

## L12 · Incident drill · Run

**Goal.** Diagnose a broken system from observation alone, then write a blameless postmortem. (M10, [05-public-postmortems.md](05-public-postmortems.md))

**Setup.** Your teaching project with the observability from L8. Another intern secretly applies one scenario. You are told only the user-visible symptom.

**Scenarios.** Add your own over time.

| Change made | Symptom you are told |
|---|---|
| HikariCP pool set to 1 | "The site is slow sometimes." |
| `notifier` consumer group id changed | "I bid and never got a notification." |
| `auction-api` container memory limit far too low | "The API disappears for a few seconds every few minutes." |
| Database host environment variable wrong | "Everything returns 500." |
| A malformed message on a topic | "Notifications stopped an hour ago." |
| Fake registrar clock skewed by five minutes | "The closing sprint bids at the wrong time." |
| Fake registrar rate limit lowered sharply | "Auctions stopped updating, but nothing is erroring." |
| Scheduler pool left at one thread with a slow job added | "The sync runs late and irregularly." |

**Steps.**
1. Reproduce the symptom. Write down the time.
2. Form a hypothesis before touching anything. Write it down.
3. Check logs, then metrics, then configuration, in that order. Record each thing you checked, what it showed, and when.
4. Find the cause. Fix it. Verify the fix with the same observation that showed the symptom.
5. Write the postmortem with the template in [05-public-postmortems.md](05-public-postmortems.md).

**Observe.** Time from symptom to correct hypothesis. Your wrong turns. Which signal would have found it fastest had it been alerted.

**Answer.** In the postmortem: what was the detection gap, and what one alert rule would have closed it?

**Deliverable.** The postmortem, with your time to diagnose recorded honestly.

---

## After the two weeks

Every lab can be repeated on another service, stack, or scenario. When you are assigned to a real service in week 3, do L1, L2, and the Read part of L6 on it in your first two days. That is the fastest way to earn the right to change it.
