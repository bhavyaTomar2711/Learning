# 02 · The Two-Week Schedule

Ten working days: three in the office each week, two at home. The plan is written as **Office** and **Home** days rather than weekdays. Arrange your week so the Office days fall on office days; if your week does not line up, swap adjacent days and keep the order of deliverables.

The plan assumes about ten hours of focused work a day. That is deliberate. Everything you cannot finish stays in this curriculum for later, but aim to finish.

References: modules M1 to M19 are in [01-mental-models.md](01-mental-models.md). Labs L1 to L12 are in [03-labs.md](03-labs.md). Spring sections S1 to S9 are in [04-spring-for-node-and-python-devs.md](04-spring-for-node-and-python-devs.md). Milestones P0 to P9 are in [10-teaching-project.md](10-teaching-project.md).

---

## Threads that run every day

- **Java hour, Days 2 to 10.** Ninety minutes first thing: one Spring section, its Build task (the day's milestone), and its Read task.
- **Daily log.** Five lines at the end of each day, format in [07-learning-with-ai.md](07-learning-with-ai.md) section 5.
- **Look-up queue.** Every "look up carefully" item you meet goes into a list. Home days are when you drain it.
- **Pull requests.** Every milestone lands as a pull request on your teaching project, reviewed by your buddy, or by another intern in the first cohort.

---

## Week 1

### Day 0 · Before you start

[00-day-zero-setup.md](00-day-zero-setup.md) sections 1 to 3: accounts, tools, Linux baseline. If you start on a Monday, do this the weekend before.

### Day 1 · Office · Orient and set up

- Finish Day 0 setup sections 4 and 5: teaching project **P0**, first reading.
- Draw the system from memory, correct it, photograph both.
- Create your progress file through a pull request; fill in the Day 1 column.
- Meet your buddy. Ask them the three things they wish they had known on their Day 1.
- **Deliverables:** P0 merged, diagram before and after, progress file.

### Day 2 · Office · Learn to read code you did not write

- **S1** with **P1**.
- **L1 · Legacy navigation** (Read, no AI first), in pairs: you choose labels for each other.
- Start **L2 · The golden path** (Read): the dashboard UI and the dashboard backend.
- Read M1 and M17.
- **Deliverables:** P1, L1 writeup, first half of the L2 trace.

### Day 3 · Home · Finish the golden path

- **S2** with **P2**.
- Finish **L2**: the auction service, the Kafka events, the consumers, Telegram. Sequence diagram with every hop labelled.
- Read M7 and M8 while you trace; you will meet both.
- Drain the look-up queue for at least two hours.
- **Deliverables:** P2, L2 diagram and hop list sent to your buddy for review.

### Day 4 · Office · The web layer and three runtimes

- **S3** with **P3**.
- **L3 · Three runtimes, one worker** (Run).
- Teach-back slot: someone who joined before you presents; you are the audience. In the first cohort, two volunteers present M1 and M17 from their reading.
- **Deliverables:** P3, L3 writeup with numbers.

### Day 5 · Home · Persistence and configuration

- **S4** with **P4** and **L5 · Schema and plans** (Run, plus a Read part).
- **L6 · Configuration design** (Run, plus a Read part).
- Read M3, M4, M5, M6.
- Re-read [07-learning-with-ai.md](07-learning-with-ai.md). You have used the tools for a week; check your habits against it.
- **Deliverables:** P4, L5 plans before and after, L6 writeup.

---

## Week 2

### Day 6 · Office · Relationships, transactions, and pools

- **S5** with **P5** and **L4 · Exhaust the pool** (Run).
- Teach-back: you present. Ten minutes, whiteboard, your choice of M1, M6, or M8. Five minutes of questions.
- **Deliverables:** P5, L4 writeup, teach-back notes.

### Day 7 · Home · The outside world and messaging

- **S6** with **P6** and **L7 · Build and run it the production way** (Run).
- **L9 · Kafka by hand** (Run, plus a Read part). You will need it for P8 on Day 9.
- Read M2, M14, M18, M19.
- **Deliverables:** P6, L7 writeup and Dockerfile, L9 writeup and topic inventory.

### Day 8 · Office · Concurrency, security, and access

- **S7** with **P7** and **L10 · Race the bids** (Run). This is the most important Java day; ask for help early.
- Re-read [11-security-rules.md](11-security-rules.md). Read M9 and M15.
- Afternoon: read-only infrastructure access is granted. Get connected, open Grafana, find the dashboards for the services you traced in L2, and compare what you see with what you expected from the code.
- **Deliverables:** P7, L10 writeup, access verified.

### Day 9 · Home · Events, observability, and failure

- **S8** with **P8** and **L8 · Make it observable** (Run).
- **L11 · Docker networking and the firewall that lied** (Run, in WSL 2 or a VM).
- [05-public-postmortems.md](05-public-postmortems.md), cases 1 to 4. Answer the questions in writing before reading what each company did.
- Read M10, M11, M12, M13, M16.
- Fill in the Day 10 column of your matrix a day early, so tomorrow's conversation is about the gaps.
- **Deliverables:** P8, L8 dashboard and alert rule, L11 writeup, postmortem answers.

### Day 10 · Office · Demo day

- Morning: **S9** with **P9**. Tests green from a clean clone.
- Midday: **L12 · Incident drill** (Run). Another intern breaks your system; you diagnose it and write the postmortem.
- Afternoon: **the demo**, fifteen minutes each, following the script in [10-teaching-project.md](10-teaching-project.md) section 5.
- Then a final teach-back, ten minutes: redraw the system from memory on the whiteboard and compare it with your Day 1 photo in front of the group. Name one thing you now understand that you did not two weeks ago, and one thing you still do not.
- Thirty minutes with Yash: the matrix, Day 1 versus Day 10, and your week 3 assignment.
- Postmortem cases 5 to 8 by the end of the day, or in the first two days of week 3.
- **Deliverables:** P9, L12 postmortem, demo, Day 10 diagram, agreed week 3 plan.

---

## If you are behind

Priority when something must give:

1. **Non-negotiable:** L1, L2, the security rules, P0 to P5, and a Day 10 demo of whatever you have.
2. **Next:** L4, L5, P7 with L10. These are where real bugs live.
3. **Then:** P6, P8, L7, L8, L9.
4. **Can slide into week 3:** L3, L6, L11, L12, P9, the postmortems, and the reading of M11, M12, M13, M16, M19.

Tell your buddy and Yash what slid. Do not quietly skip. Everything you skipped is still here.

## If you are ahead

- Take a stretch goal from the teaching project. The transactional outbox and leader election are the most valuable.
- Pick the Spring checklist rows marked **Heavy** that you have not ticked and build something small for each.
- Write the "look up carefully" answers for one module as a document and add it to this repository in a pull request, for the next intern.
