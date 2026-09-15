# 00 · Day Zero: Accounts, Tools, Baseline, First Reading

**Goal:** by the end of Day 1 your laptop has every toolchain we use, your own teaching project skeleton runs, you have read the documents that orient you, and you have drawn the system from memory once.

You will **not** run any company service in the first two weeks. You read our code and you run your own. See the README, section 3.

---

## 1. Accounts and access

Yash sets these up. You check them off and report anything missing on Day 1.

- [ ] GitHub: member of the Namekart organisation. Use this account for everything, including future infrastructure access, because access is tied to identity.
- [ ] Two-factor authentication on GitHub and on your company email. Before anything else.
- [ ] Company email and chat.
- [ ] Cursor licence.
- [ ] Read access to this curriculum repository.
- [ ] Read access to the service repositories used in reading exercises: the dashboard backend, the dashboard UI, and the auction service. Others come later as needed.
- [ ] Added to the interns group. This is where you ask after twenty minutes stuck.
- [ ] Assigned a buddy.

Not granted yet, on purpose: server access, the deployment platform, Grafana, any database. Day 8.

---

## 2. Laptop tooling (any OS)

Install these and verify each with its version command. Write the versions in your notes.

| Tool | Why | Nuance to know |
|---|---|---|
| Git | Everything | Configure your name and company email |
| Docker Desktop, or Docker Engine plus Compose on Linux | Your teaching project runs in Compose | On Windows use the WSL 2 backend. Give Docker at least 6 GB of RAM; MySQL plus Kafka plus a JVM need it |
| Java 21 (Temurin or similar) | All our core backends | Understand `JAVA_HOME`, and the difference between a JDK and a JRE |
| Maven | Java builds | Projects ship a `mvnw` wrapper; know why wrappers exist |
| Node 20 LTS through a version manager (nvm, fnm, volta) | Frontends and Node services | Projects pin different Node versions. Learn to switch |
| Python 3.11 or newer, plus `uv` | AI and data services | Virtual environments before anything else. Never install project packages globally |
| A SQL client (DBeaver, TablePlus, or the MySQL CLI) | You will read schemas and query plans constantly | Learn to run `EXPLAIN` from it |
| An HTTP client: `curl`, plus Bruno, Postman, or HTTPie | Poking APIs | Learn `curl` properly; it is in every runbook |
| `rg` (ripgrep) | Searching codebases fast | The main tool for the navigation lab |
| Cursor | Editor and AI pair | Read [07-learning-with-ai.md](07-learning-with-ai.md) before you rely on it |

**On Windows,** do your terminal work in WSL 2 (Ubuntu). Almost every Dockerfile, script, and runbook assumes a POSIX shell, and lab L11 needs Linux networking.

**Sanity check your toolchains:** create a throwaway FastAPI app with `uv` and a throwaway Node HTTP server, and get a response from each with `curl`. Ten minutes. It proves Python and Node are ready before you need them on Day 9.

---

## 3. Linux and shell baseline

You should be able to do all of these without looking anything up by the end of week 1. Test yourself on Day 0 and note the gaps.

- Navigate, find files, search inside files (`find`, `grep`, `rg`), follow a log (`tail -f`), page through output (`less`).
- Processes and ports: what is listening on a port, which process owns it, how to stop it.
- Environment variables: set one for a single command, export one for a shell, see what a process has.
- Permissions: read `ls -l`, use `chmod` and `chown`, and know why running as root is the wrong default.
- Pipes and redirection, exit codes, `&&` versus `;`.
- SSH concepts: key pairs, `~/.ssh/config`, why password login is disabled on any serious server.
- Disk: `df`, `du`, and why a directory with millions of small files behaves very differently from one big file.

Look up carefully: login versus non-login shells and why your `PATH` differs between them; file descriptors 0, 1, 2 and `/dev/null`; signals, and what SIGTERM versus SIGKILL means for a container.

---

## 4. Your own environment

Follow milestone **P0** in [10-teaching-project.md](10-teaching-project.md):

- Create your private `mini-amp-<your-name>` repository and add Yash and your buddy.
- Write a Compose file that runs MySQL 8, Kafka in KRaft mode, and a Kafka UI. Writing it yourself is the exercise; do not copy one wholesale.
- Generate a Spring Boot 3 project from Spring Initializr and get it healthy against your MySQL.
- Write the README so your buddy can start everything with one command.

If you have never run a Spring app: you are about to spend ninety minutes a day on it. Do not try to understand it today. Get it running and write down every question.

---

## 5. First reading

Read in this order. Do not skim the first two.

1. **[09-platform-overview.md](09-platform-overview.md).** What we run, how it connects, and the vocabulary of the business.
2. **[09b-domain-auction-lifecycle.md](09b-domain-auction-lifecycle.md).** How the business actually acquires domains. Answer its self-check questions in writing.
3. **[11-security-rules.md](11-security-rules.md).** Fully. These rules apply from your first hour.
4. **[07-learning-with-ai.md](07-learning-with-ai.md).** You will be using AI tools from tomorrow morning.
5. **The `CLAUDE.md` of the dashboard backend, the dashboard UI, and the auction service,** in their repositories. Read for orientation, not mastery. Write down every term you do not know; that list is your first look-up queue.

---

## 6. Day 1 deliverable: draw the system from memory

Close every document. On paper or a whiteboard, draw:

- Every service you remember, grouped by language.
- Every datastore.
- Every connection you remember, labelled with its mechanism: REST, Kafka topic, WebSocket, webhook.
- Where the user's browser touches it.
- Where the outside world touches it: registrars, Telegram, email providers, LLM APIs.

Then open the platform overview and mark your drawing in a second colour: what you missed, what you got wrong. Photograph both versions. That photo is your Day 1 deliverable and your Day 10 comparison point.

Also fill in the Day 1 column of the competency matrix in your progress file, honestly.

---

## 7. Done for Day 0 and Day 1

- [ ] Accounts and two-factor authentication in place.
- [ ] All tools installed, versions noted, Python and Node sanity checks passed.
- [ ] Linux baseline self-test done, gaps noted.
- [ ] Teaching project P0 merged: Compose healthy, Spring Boot app healthy, README runnable by your buddy.
- [ ] Platform overview, auction lifecycle with self-check answers, security rules, AI guide, and three service `CLAUDE.md` files read.
- [ ] System diagram drawn from memory, corrected, photographed.
- [ ] Progress file created through your first pull request, Day 1 column filled.
- [ ] A written list of every term you did not know. Bring it to the office.
