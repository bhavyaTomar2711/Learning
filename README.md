# Namekart Engineering Curriculum

**Status:** v0.2 draft. Owner: Yash. Read this file first, fully. It takes ten minutes.

This curriculum takes a strong final-year student and makes them a productive engineer on a multi-stack production system inside two weeks, and a self-sufficient one inside two months. It is built for a startup: the pace is deliberately hard, the load is deliberately high, and nothing here is optional unless it says so.

It is also built to outlive the first two weeks. Anything you cannot finish stays here for you to come back to. Treat it as a map, not a checklist you throw away.

---

## Before you start: what this is really for

This curriculum exists to bring out the best in you, and to help you understand how real systems work as clearly as we can possibly explain it. It is not a race, and it is not a test you pass by reaching the last page first.

The schedule is ambitious on purpose, because ambitious plans stretch people further than comfortable ones. But the days are a guide, not a verdict. If you are genuinely learning, doing the labs properly, and building real understanding, then taking longer than the schedule says is completely fine. Understanding one thing deeply is worth far more, to you and to us, than skimming three.

What we do ask is that you use your time well. Stay focused, write things down, ask after twenty minutes instead of staying stuck for a day, and tell your buddy or Yash when something slides. Honest effort and steady progress are exactly what we are looking for, and if you bring those, you have nothing to worry about.

Parts of this will feel hard. That is not a sign you are behind. It is a sign it is working. Every good engineer you will meet here found this material hard once too.

---

## 1. Who this is for

Any engineering intern or new hire joining Namekart, whenever they join. It assumes:

- You can already program well in at least one language and have built at least one full-stack project.
- You are comfortable with data structures and algorithms. We spend zero time on them.
- You have used Git, basic Docker, and a REST framework in some language.
- You have **not** necessarily used Java or Spring, Kafka, MySQL at scale, or any observability tooling. Most joiners have not. That is expected and planned for.

If you join after others, you follow the same path from Day 0. The interns ahead of you are your first line of help, and their teach-backs are part of your material.

---

## 2. What we are trying to build in you

Not "knows Spring" or "knows React". Those are instances. We want three things:

1. **Portable mental models.** You understand the concept underneath (a connection pool, an event loop, at-least-once delivery) so well that when you meet a new stack you ask "how does this one do X?" instead of starting from zero.
2. **Production instincts.** You read logs before you guess, you check the pool before you blame the database, you set a timeout on every outbound call, and you never paste a secret anywhere.
3. **The ability to work inside code you did not write.** Every project you have built so far was greenfield and yours. Ours is large, old in places, and full of decisions made under pressure. Navigating that is a skill, and we teach it explicitly.

You may be put on something brand new in month two. The pattern is identical: find the concept, find how this stack does it, find where it can hurt you in production.

---

## 3. The one idea behind the first two weeks

**Read our real code. Run your own.**

- **Reading** our services needs nothing but repository access. The navigation and golden-path labs, and a reading exercise in every Spring section, have you trace real flows through real code.
- **Running** happens only on a project you build yourself: **Mini AMP**, a small domain auction platform with the same vocabulary and the same problems as our real systems. Every hands-on lab runs on it.

You do not run any company service locally in the first two weeks. That is deliberate. Our services depend on a lot of infrastructure, and setting them up is not a good use of your first fortnight or of anyone else's time. From week 3, you run the one service you are assigned to, using a setup prepared once for that service.

---

## 4. How the material is organised

| File | What it is | When you use it |
|---|---|---|
| [00-day-zero-setup.md](00-day-zero-setup.md) | Accounts, tools, Linux baseline, first reading | Day 0 to 1 |
| [01-mental-models.md](01-mental-models.md) | The Rosetta matrix: every core concept, how each of our stacks does it, nuances to look up | Daily reference for months |
| [02-two-week-schedule.md](02-two-week-schedule.md) | Day-by-day plan, aware of office and home days | Days 1 to 10 |
| [03-labs.md](03-labs.md) | Twelve hands-on labs, each marked Read or Run | As scheduled, and revisited later |
| [04-spring-for-node-and-python-devs.md](04-spring-for-node-and-python-devs.md) | Java and Spring taught as translation from what you already know, nine sections | Daily Java hour, Days 2 to 10 |
| [04b-spring-concept-checklist.md](04b-spring-concept-checklist.md) | Every Spring and Java concept to touch, with how heavily we use it | Tick off over months |
| [05-public-postmortems.md](05-public-postmortems.md) | Famous public incidents as teaching material | Days 9 to 10, and whenever you touch production |
| [06-competency-matrix.md](06-competency-matrix.md) | Self-assessment grid and deliverables log | Day 1, Day 10, then monthly |
| [07-learning-with-ai.md](07-learning-with-ai.md) | Using Cursor, ChatGPT and Claude as a tutor without outsourcing your understanding | Day 0, again on Day 5 |
| [08-weeks-3-to-8-ladder.md](08-weeks-3-to-8-ladder.md) | Your first real ticket and the contribution ladder after that | Week 3 onward |
| [09-platform-overview.md](09-platform-overview.md) | What we run, how it fits together, our conventions | Day 0, again on Day 10 |
| [09b-domain-auction-lifecycle.md](09b-domain-auction-lifecycle.md) | How domain acquisition works in the real world, and why our auction and acquisition code looks the way it does | Day 0, and before any acquisition ticket |
| [10-teaching-project.md](10-teaching-project.md) | Mini AMP: the project you build in the first two weeks | Day 1 to Day 10, then stretch goals |
| [11-security-rules.md](11-security-rules.md) | The security rules that apply to you | Day 0, again on Day 8 |

Resources are deliberately **not** linked. Each topic lists what to learn and which nuances matter. Learn from whatever works for you: documentation, videos, blogs, books, an AI tutor. Judge a resource by whether it explains the *why* and the failure modes, not just the API.

---

## 5. How to use this repository

- Read it on GitHub or in Cursor's Markdown preview. GitHub renders the diagrams directly; in Cursor, install a Mermaid preview extension to see them.
- **If you received this as a zip file** because your GitHub access is not ready yet: read it in Cursor, keep your progress file in the `progress` folder on your laptop, and keep your teaching project as a local Git repository. Move both to GitHub through pull requests once your access arrives. The zip is internal material: do not upload it anywhere or share it outside Namekart.
- On Day 1, create `progress/<your-name>.md` by copying [06-competency-matrix.md](06-competency-matrix.md) and [04b-spring-concept-checklist.md](04b-spring-concept-checklist.md) into it. Open a pull request to add it. That is your first pull request here.
- Update your progress file at least on Day 10 and monthly, each time through a pull request.
- If something in the curriculum is wrong, unclear, or outdated, fix it in a pull request. Improving this curriculum for the next person is part of the job.
- Your teaching project lives in its own repository, described in [10-teaching-project.md](10-teaching-project.md).

---

## 6. Vocabulary

**Module.** One concept in the mental-models file. Has an invariant, a stack-by-stack comparison, a "look up carefully" list, a pointer to our code, and a self-check.

**Lab.** A hands-on exercise with a goal, setup, steps, something you must observe with your own eyes, questions you answer in writing, and a deliverable. **Read** labs need only repository access. **Run** labs happen on your own laptop with your own project. Reading about a lab is worth nothing. Doing it is worth a lot.

**Teaching project.** Mini AMP. Built one milestone per day, reviewed through pull requests, demoed on Day 10.

**Deliverable.** The artifact that proves a module, lab, or milestone is done: a diagram, a writeup, a pull request, a postmortem. Progress is measured only by deliverables. Never by pages read or videos watched.

**Teach-back.** You explain a topic to the other interns for ten minutes, whiteboard only, no slides, then take five minutes of questions. It is the cheapest proof that you understand something. Teach-backs happen on office days.

**Golden path.** One real business action traced end to end through every layer of our real code: a domain shortlisted in the dashboard, handed to the auction service, turned into Kafka events, and announced on Telegram. You trace it by reading, not running.

**Buddy.** The intern who joined most recently before you. Your first reviewer and your first person to ask.

**Rosetta.** The idea that the same concept has a name in every stack, and your job is to build the translation table in your head.

---

## 7. Rules

1. **Explain before you paste.** Use any AI tool you like. Never commit, present, or defend a line you cannot explain. See [07-learning-with-ai.md](07-learning-with-ai.md).
2. **Read real code, run your own.** No company service runs on your laptop in the first two weeks, and you do not ask engineers for help setting one up.
3. **No infrastructure access before Day 8.** Access granted on Day 8 is read-only: look, never change.
4. **Secrets never go in chat, prompts, screenshots, tickets, or commits.** If you see something in a repository that looks like a credential, do not copy, use, or test it. Report it. See [11-security-rules.md](11-security-rules.md).
5. **Write it down.** Every lab produces a written answer. Every day ends with a five-line log in your own notes. Bring your questions to the office.
6. **Ask after twenty minutes.** Stuck for twenty minutes: ask your buddy or the interns group. Not after two minutes, not after a day.
7. **Ship in two stacks before you specialise.** Nobody becomes "the Python person" until they have merged something in a second stack.
8. **Deliverables gate progress.** You move on because the deliverable exists, not because a day passed.

---

## 8. How the two weeks work

Ten working days: three in the office each week, two at home.

- **Office days** are for labs that benefit from a second pair of eyes, teach-backs, pairing, reviews, and questions you saved.
- **Home days** are for deep self-study, solo labs, building milestones, and writing deliverables.
- **The daily Java hour** runs every day from Day 2 to Day 10. Java and Spring are the shared unknown for almost everyone who joins, and daily contact is the only way through.

The schedule is a default. If you are already strong in an area, prove it with the deliverable and move on. If an area is new, take the time. Two weeks is the target, not a deadline that ends learning.

---

## 9. How progress is tracked

Your progress file holds the competency matrix and the Spring checklist. You fill the matrix three times in the first month: Day 1, honestly and before you start; Day 10; and at the end of the month. It is a self-assessment, not a performance review. Its job is to show you and Yash where the gaps are.

Your internship is evaluated at the end of six months on what you shipped and how you worked, not on this grid and not on how fast you finished the two weeks.

---

## 10. For the mentor

- **No full-time engineer time goes into local setup of company services during the first two weeks.** The curriculum is designed so it is never needed.
- **Before a new joiner starts,** grant: GitHub organisation membership, read access to the dashboard backend, dashboard UI, and auction service repositories, this curriculum repository, Cursor, and the interns group. Assign a buddy.
- **Office days:** review deliverables in batch, fifteen minutes per intern, driven by their written questions. Attend at least one teach-back a week.
- **Day 8:** grant read-only infrastructure access.
- **Day 10:** attend the demos, then thirty minutes per intern on the matrix and the week 3 assignment.
- **Before week 3:** for each service an intern will be assigned to, make sure a local setup exists once, written by whoever owns the service. Never per intern.
- **After each cohort:** harvest teach-back notes, confusions, and pull requests into this curriculum. It should get sharper with every intake.
- **When someone is put on a new stack:** ask them to add its column to the mental-models file before they write code. That is the whole method.
