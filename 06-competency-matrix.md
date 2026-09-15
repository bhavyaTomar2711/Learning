# 06 · Competency Matrix

A self-assessment. Fill it in on Day 1 (before you start), Day 10, and then at the end of each month. Copy this file to `progress/<your-name>.md` in the same folder and fill in that copy; keep this one blank as the template.

It is not a performance review. Its only job is to show you and Yash where the gaps are so the next month's tickets target them. Be honest on Day 1; inflated numbers only hide what you need.

---

## Levels

| Level | Meaning | Evidence |
|---|---|---|
| **0** | Have not met it | Nothing |
| **1** | Can explain the concept and recognise it in code | Can answer the module's self-check without notes |
| **2** | Can use it correctly in one of our stacks with guidance | Have shipped or completed a lab that exercises it, reviewed by someone |
| **3** | Can use it in two or more stacks, debug it in production, and teach it | Have fixed a real bug in it or taught it back; can add a column for a new stack |

Level 3 in everything is not the goal of an internship. Level 1 everywhere plus level 2 in the rows that match your work plus level 3 in two or three rows is a strong six months.

---

## The grid

| Area (module) | Day 1 | Day 10 | Month 1 | Month 2 | Month 3 | Evidence / notes |
|---|---|---|---|---|---|---|
| M1 Execution model and concurrency | | | | | | |
| M2 Memory, lifecycle, resource limits | | | | | | |
| M3 Build, dependencies, packaging | | | | | | |
| M4 Configuration and secrets | | | | | | |
| M5 Data modelling, ORMs, schema evolution | | | | | | |
| M6 SQL depth: indexes, transactions, pools | | | | | | |
| M7 HTTP APIs and service calls | | | | | | |
| M8 Messaging and real-time | | | | | | |
| M9 Auth and identity | | | | | | |
| M10 Logging, metrics, traces | | | | | | |
| M11 Testing | | | | | | |
| M12 Frontend rendering and state | | | | | | |
| M13 AI systems and agents | | | | | | |
| M14 Containers, networking, deployment | | | | | | |
| M15 Security fundamentals | | | | | | |
| M16 Git, review, delivery | | | | | | |
| M17 Working in a large codebase | | | | | | |
| M18 Time, scheduling, background work | | | | | | |
| M19 Caching and rate limiting | | | | | | |

## Stack familiarity

Separate from the concepts. Same 0 to 3 scale, where 3 means "I could be given a ticket in this stack with no ramp-up".

| Stack | Day 1 | Day 10 | Month 1 | Month 2 | Month 3 | Notes |
|---|---|---|---|---|---|---|
| Java and Spring Boot | | | | | | |
| Python and FastAPI | | | | | | |
| LangGraph and LLM tooling | | | | | | |
| Node and TypeScript on the server | | | | | | |
| React (CRA and Material UI) | | | | | | |
| Next.js | | | | | | |
| MySQL | | | | | | |
| PostgreSQL and Supabase | | | | | | |
| Kafka | | | | | | |
| Docker and Compose | | | | | | |
| Linux and shell | | | | | | |
| Coolify, Cloudflare, Tailscale (after Day 8) | | | | | | |
| Prometheus, Loki, Grafana (after Day 8) | | | | | | |

## Deliverables log

One line per deliverable, with the date. This is the part Yash actually reads.

| Date | Deliverable | Reviewed by | Link or location |
|---|---|---|---|
| | Day 1 system diagram, before and after | | |
| | Teaching project P0: skeleton and Compose running | | |
| | L1 navigation writeup | | |
| | L2 golden path trace | | |
| | Teaching project P1 to P9, one row each | | |
| | Labs L3 to L12, one row each | | |
| | Public postmortem answers | | |
| | Day 10 demo and teach-back | | |

For Spring specifically, also tick off [04b-spring-concept-checklist.md](04b-spring-concept-checklist.md) in your copy. The matrix says how well; the checklist says what is still untouched.

## Ships

Every merged pull request, by stack. The two-stack rule from the README is checked here.

| Date | Repository | Stack | What | Size (S/M/L) |
|---|---|---|---|---|
| | | | | |

---

## For Yash: reading the matrix

- A row at 0 after Day 10 that the two-week plan covered means the intern skipped it or the material failed. Ask which.
- A row stuck at 1 for two months means they never got a ticket that exercised it. Fix the ticket, not the intern.
- A stack at 3 with the concept rows at 1 is memorisation, not understanding. Probe it in a teach-back.
- Compare Day 1 columns across interns to decide the pairing and the first tickets; compare Month 3 columns to decide who gets the new project.
