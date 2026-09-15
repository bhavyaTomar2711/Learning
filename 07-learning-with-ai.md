# 07 · Learning With AI Without Outsourcing Your Brain

You will have Cursor, ChatGPT, Claude and whatever comes next. Used well, they compress months of learning into weeks. Used badly, they produce an intern who can ship code they cannot debug, and who leaks a secret into a prompt. This file is about the difference.

Read it on Day 0. Read it again on Day 5, when you have started to trust the tools too much.

---

## 1. The one rule

**You may not commit, run in a shared environment, present, or defend a line of code you cannot explain.**

"Explain" means: what it does, why it is there, what happens if you delete it, and what it assumes about its inputs. If you cannot do that, you have not finished. Ask the tool to explain it, then close the tool and explain it back in your own words. If you still cannot, you do not understand it yet.

This rule is not about purity. It is about the moment at 11pm when production is broken and the AI is confidently wrong. The only thing that saves you then is your own model of the system.

---

## 2. Two modes, and knowing which one you are in

**Tutor mode.** You are trying to *learn* something. The AI is a patient expert who never tires of your questions. Use it hard. Tactics that work:

- Ask for the concept first, then the stack instance. "Explain what a connection pool is and why it exists. Now show me how HikariCP configures one. Now how Prisma does. What is different and why?"
- Ask for failure modes, not features. "What goes wrong when this is misconfigured? What does the error look like? How would I notice in logs?"
- Ask for counterexamples and edge cases. "When would I *not* want this?"
- Ask it to quiz you. "Ask me five questions about Kafka consumer groups, one at a time, and tell me where my answers are weak."
- Ask it to draw parallels. "I know Express middleware. Explain Spring filters and interceptors in those terms, and tell me where the analogy breaks."
- Use the Feynman check. Explain the topic to the AI as if it were a junior, and ask it to find the holes in your explanation.
- Never accept a fact about *our* system from an AI unless you have verified it in our code or docs. General knowledge is usually right. Specifics about our repositories are guesses unless the tool has read the file in front of you.

**Pair mode.** You are trying to *produce* something. The AI writes code with you. Tactics that work:

- Give it the concept and the constraints, not just the outcome. "Add an endpoint following the same controller, service, repository pattern as the existing ones in this package, with validation done the way the other controllers do it."
- Make it read before it writes. Point it at the neighbouring files. Our codebases have conventions, and the tool will invent its own if you let it.
- Ask it to explain the diff before you accept it. Every time. It takes thirty seconds and it is how you learn the codebase.
- Never let it "fix" a failing test by changing the test. Ask why the test fails.
- When it proposes a dependency, ask whether the project already has one that does this. It usually does.
- Run the code. AI-written code that has not executed is a hypothesis.

---

## 3. Things you must never put in a prompt

- Credentials of any kind: API keys, tokens, passwords, connection strings with passwords, private keys, `.env` file contents, the contents of any file under a `secrets` folder.
- Customer data or personal data from our databases.
- Production log excerpts without checking them for the above first. Logs leak secrets constantly.
- Full database dumps or schema dumps with data.

AI tools are a chat, and one you do not control: prompts can be logged, retained, and reviewed outside the company. The authority on this is [11-security-rules.md](11-security-rules.md), which you read on Day 0.

If you are unsure whether something is sensitive, it is. Redact it, or ask.

---

## 4. How the tools lie to you, specifically

Know these so you recognise them.

- **Confident configuration.** It will produce a plausible `application.properties` or Compose file with keys that do not exist or that were renamed two versions ago. Check property names against the actual documentation of the version we run.
- **Version drift.** Its training data mixes Spring Boot 2 and 3, React class components and hooks, Next.js pages and app routers, Python 3.8 and 3.12. Always tell it the version, and check.
- **Invented APIs.** Methods that should exist but do not. Compile, run, test.
- **Silent behaviour changes.** A refactor that "cleans up" code and quietly changes the transaction boundary, or the order of middleware, or whether an exception is swallowed. Read diffs for behaviour, not just style.
- **Agreement bias.** If you propose a wrong idea confidently, it will often help you do it. Ask "what is wrong with this approach?" before "how do I do this?".
- **Local optimum.** It solves the problem in front of it and does not know the project already solves it elsewhere. Search the codebase first.

---

## 5. A daily habit that makes this work

End each day with five lines in your own notes, in your own words, with no AI involved:

1. One thing I learned today that I can now explain without help.
2. One thing I used today that I could not yet explain.
3. One thing the AI got wrong today and how I caught it.
4. One question I want to ask a human.
5. One thing I want to look up carefully tomorrow.

Line 2 is your queue for tutor mode tomorrow. Line 3 is how you calibrate trust. Line 4 goes to the office.

---

## 6. Using AI for the labs

Every lab in [03-labs.md](03-labs.md) asks you to *observe* something and *explain* it. The AI can help you set up, and it can help you interpret, but the observation has to be yours and the explanation has to be in your words. A lab writeup that reads like model output will be sent back.

Some labs say "no AI for the first attempt." Respect that. The point of those labs is to build the muscle of navigating unfamiliar code with grep and reading. You will use that muscle when the tool is wrong, which is exactly when it matters.
