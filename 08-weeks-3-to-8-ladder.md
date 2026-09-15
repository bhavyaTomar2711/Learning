# 08 · Weeks 3 to 8: The Contribution Ladder

The two weeks made you able to read the system and build a small version of it yourself. The next six make you someone who can be handed a problem. This file describes how work is chosen, what the rules are, and how the same learning pattern applies when you are put on something entirely new.

---

## 1. The rules that carry over

- **Two stacks before you specialise.** You must merge at least one change in a second stack before you settle into a home stack. Nobody becomes "the Python person" without having shipped Java, or vice versa.
- **Every ticket teaches something.** When you pick or are given a ticket, name the module from [01-mental-models.md](01-mental-models.md) it exercises. If you cannot, ask what the ticket is for.
- **Explain before you paste.** Still the rule. Reviews will ask you to explain lines.
- **Deliverables gate progress.** A ticket is done when it is merged, deployed, and you have watched it behave in production through Grafana or the logs. Not when the pull request opens.
- **Keep the matrix current.** Monthly, honestly.
- **Teach back once a month.** Fifteen minutes on something you shipped, for the interns behind you.

---

## 2. Your first real ticket

In the first two weeks you ran only your own teaching project. From week 3 you are assigned to one company service, and that is when you run it locally for the first time.

- **Setup is prepared once per service, not per intern.** Before a service's first ticket is handed out, its local setup is written down or packaged by whoever owns that service. You follow that guide. You do not attempt to set up other company services on your own initiative, and you do not pull engineers into ad hoc setup help.
- **If the setup for your service is not ready yet,** you are not idle. Keep going with reading work on that service (trace a second flow, write the missing parts of its `CLAUDE.md` as a pull request), the stretch goals of your teaching project, and any labs or public postmortems you skipped.
- **Your first ticket is level 1,** is reviewed by your buddy first and Yash second, and is done only when you have watched it deploy and behave.

---

## 3. Ticket levels

Tickets are tagged by level. You move up when the level below is comfortable, not when a calendar says so.

**Level 1 · Bounded change, existing pattern.** Add a field to a DTO and surface it in the UI. Add a filter to a list endpoint. Fix a bug with a known reproduction. Add a missing index found by a slow query. Add a missing test to a service that has few. Add a health check to a service that lacks one. Wire a service into the observability preset.

What it teaches: conventions, the review process, the deploy pipeline, watching a change land.

**Level 2 · A feature across two layers, or a bug without a reproduction.** A new endpoint plus UI. A new Kafka event and its consumer. A scheduled job with proper overlap protection. A slow endpoint diagnosed and fixed with plans and numbers. An idempotency fix on a consumer that was not idempotent. A Dockerfile made multi-stage and fast. A dashboard for a service that has none.

What it teaches: the module underneath, end to end, in one stack.

**Level 3 · A change with production risk, or across services.** A schema change on a live table (with the `ddl-auto` caveats understood). A change to the contract between two services with a compatibility plan. Migrating a service's configuration to the shared-variables pattern. A new AI pipeline stage with an eval. Performance work on the bidding path where timing is money. A security improvement agreed with Yash.

What it teaches: risk, rollback, compatibility, and communication.

**Level 4 · Own a small service or a new project.** Design it, write its `CLAUDE.md`, wire observability from day one, deploy it, run it. This is where month four to six goes for the interns who got here.

---

## 3. Landing zones by background

Where each kind of background tends to be productive fastest, and what the second stack should be. This is a starting point, not an assignment; the Day 10 conversation decides.

| Background | Fast first wins | Second stack to force |
|---|---|---|
| Python and LangGraph strong | Lead-gen research, domain processing, negotiation and prediction agents: evals, cost control, new sources | Java: the orchestrator that calls these services |
| Node and React strong, DevOps instinct | Scraping service, dashboard UI, observability and deployment tickets | Python: a lead-gen pipeline stage |
| FastAPI plus React plus real-time experience | Domain processing, dashboard SSE and WebSocket layer | Java: the Kafka listeners that feed those channels |
| Next.js plus Supabase plus payments | The marketplace app and the name.ai frontend, both on Postgres | Java: any dashboard backend endpoint |
| Anyone with Java from college | Dashboard backend and auction service level 1 tickets immediately | Python: anything in the AI services |

The auction service is where the money is and where timing bugs cost the most. Nobody touches its bidding path at level 1 or 2. Reading it, tracing it, and adding observability to it are all encouraged.

---

## 4. When you are put on something completely new

It will happen. A new product, a new stack, a client project. The curriculum does not change; only the column does.

Day 1 on a new project:
1. Run [L1](03-labs.md), [L2](03-labs.md) and [L6](03-labs.md) against it. If it is greenfield and there is nothing to trace, trace the closest thing we have and note what will differ.
2. Fill in the appendix of [01-mental-models.md](01-mental-models.md): one page, one row per module, "how does this stack do it, where does it hurt". Blank rows are your reading list for the week.
3. Write the project's `CLAUDE.md` before you write its code. If you cannot describe it, you cannot build it.
4. Wire observability before the first feature. The conventions in [09-platform-overview.md](09-platform-overview.md) make this one environment variable; there is no excuse.
5. Decide the configuration and secrets story on day one. It is never fixed later without pain.
6. Pick the boring option for everything that is not the project's actual point. The project's point deserves your novelty budget; the database does not.

Day 2 to 5: the same mental-models rows in the same order as the two-week plan, compressed. Execution model, data, configuration, HTTP, messaging if any, observability, deployment. For each, learn how this stack does it and write it into the column.

By the end of the first week on a new stack you should be able to give a ten-minute teach-back titled "How X does the things we already know." That teach-back becomes the next column of the matrix for whoever follows you.

---

## 5. What Yash reviews

- **Pull requests** for behaviour and risk first, conventions second, style last. Expect to be asked "what happens if this fails" on every outbound call and "what happens if this runs twice" on every job.
- **The deliverables log** in your competency matrix, monthly.
- **Teach-backs**, as audience.
- **Your questions.** Bring written ones to office days. A day of questions saved is a day of progress; a day of silence is a warning sign.

---

## 6. Month-by-month shape

| Month | Focus | Exit marker |
|---|---|---|
| 1 (includes the two weeks) | Foundations, first ships in two stacks | Level 1 tickets merged in two stacks; matrix mostly at 1 or better |
| 2 | Level 2 tickets in the home stack; one level 2 in the second stack | Can be handed a two-layer feature and deliver it with tests and a dashboard |
| 3 | First level 3 ticket, paired | Has made a change with production risk, with rollback plan, and watched it land |
| 4 to 6 | Own something: a service, a project, a workstream | A `CLAUDE.md` you wrote, a service you deployed, a teach-back the next cohort uses |

The six-month evaluation looks at the ships table, the deliverables log, the teach-backs, and how you handled the things that went wrong. It does not look at how fast you finished the two weeks.
