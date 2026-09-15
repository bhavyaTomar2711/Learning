# 05 · Public Postmortems: Learning From Other People's Worst Days

You will not personally cause a $440 million loss in your first year, and that is exactly why you should study someone who did. Public postmortems are the closest thing engineering has to a black box recorder: companies with far more money and far better engineers than any of us have already made the mistakes you are about to make, and several of them wrote it up in detail. That is a gift. Use it.

The mindset that makes this useful is blameless. Every incident below has a name attached to it in the press coverage, and none of those names matter. What matters is the system: the checks that did not exist, the assumption that seemed reasonable on a normal Tuesday, the safeguard that was one config flag away from working. A blameless read asks "what made this the reasonable thing to do at the time" instead of "who screwed up."

For each case, stop at "What happened" and answer the questions in "Before reading on" yourself, on paper, before you read the rest. That rehearsal is the actual lesson. Reading the ending without doing that is trivia, not training.

---

### PM-1 · The flag nobody deleted (Knight Capital, August 2012)

**What happened:** Knight Capital was a major market maker that relied on automated trading software running across eight production servers. Ahead of a new NYSE program, engineers pushed an updated version of the trading software, but the deployment only reached seven of the eight servers. The eighth server kept running old code that included a dormant, years-unused function, activated by a flag that had since been repurposed to mean something else entirely on the updated servers. When markets opened, that eighth server read the repurposed flag as an instruction to run the old, dead trading logic, which began buying and selling huge volumes of shares with no sane limits. Nobody realized which server was misbehaving or why for a long time, because the error output was not an obvious crash, it was a torrent of live trades. In about 45 minutes, the firm built enormous unintended positions across roughly 150 securities and lost approximately $440 million, more than the company was worth. Knight survived only by taking emergency financing within days and was acquired by a competitor months later.

**The concept underneath:** A deployment that only reaches some of your fleet does not fail loudly, it leaves you running two different programs under one name, and the safest-looking flag reuse can silently resurrect code you thought was gone.

**Connects to modules:** M14 Containers, networking, and deployment; M16 Git, code review, and delivery; M10 Errors, logging, metrics, and traces; M11 Testing.

**Look up carefully:**
- Partial deployments and why "deploy to N of M servers" needs to be an impossible state, not a manual step
- Dead code and why "we'll just leave it, it's not called" is a decision with an expiry date
- Feature flags and config flags as code: repurposing a flag's meaning is a breaking change
- Kill switches: what it means to have one, and why Knight effectively didn't
- Atomic and reversible deployments vs rolling ones without a fast, verified rollback
- Pre-trade risk limits and circuit breakers as a second, independent line of defense

**Before reading on, answer:**
- What would make a partial deployment across 8 servers physically impossible, not just against policy?
- How would you detect, within seconds, that one server is running different code than its seven siblings?
- What single safeguard, if it had existed, would have capped the loss regardless of what the code did?
- Why is "we'll clean up the dead code later" more dangerous than it sounds?

**What they did afterwards:** The episode became one of the most cited case studies in deployment safety and is widely credited with accelerating industry adoption of automated, all-or-nothing deployment tooling, mandatory kill switches for trading systems, and stricter pre-trade risk controls under later SEC rules. Knight Capital itself did not survive as an independent company.

**The lesson in one line:** If your fleet can run two different versions of your code at the same time without anyone knowing, you do not have a deployment process, you have a coin flip.

**Teach-back prompt:** Your deploy tool reports "7/8 servers updated successfully" and moves on. Is that a passing deploy? Design the check that would make it fail instead.

---

### PM-2 · The command that knew how to count, not how much (AWS S3, February 28 2017)

**What happened:** Engineers at Amazon Web Services were debugging why the S3 billing system in the us-east-1 region was running slower than expected. Following an established playbook, one engineer ran a command meant to take a small number of servers offline for one billing-related subsystem. One of the command's inputs was entered incorrectly, and it removed a much larger set of servers than intended, taking down capacity for two other subsystems, the index system that tracks where every object lives and the placement system that decides where new objects go. Losing those meant S3 in that region could not serve or store objects at all, which cascaded into every service that depended on S3, including, notably, AWS's own public status dashboard, which itself relied on S3 and could not be updated to tell customers what was happening. Restarting the affected subsystems was not a quick flip: they had not been fully restarted at that scale in years, and safety checks on the much larger dataset they now held took hours to complete. The region was substantially degraded for several hours.

**The concept underneath:** Blast radius is decided before the incident, by how broad and how reversible your operational commands are allowed to be, and a system you never fully restart is a system whose restart time you do not actually know.

**Connects to modules:** M14 Containers, networking, and deployment; M10 Errors, logging, metrics, and traces; M2 Memory, lifecycle, and resource limits; M6 SQL depth.

**Look up carefully:**
- Blast radius as a design property: capacity-removal tooling with a maximum, enforced regardless of operator input
- Input validation on operational scripts, not just application code
- Status pages and monitoring that share infrastructure with the thing they monitor
- Cold-start behavior: systems that have never been fully restarted are systems whose recovery time is a guess
- Staged, scoped operational changes vs one command hitting an entire subsystem
- Dependency mapping: which of your own systems secretly depend on the system you are about to touch

**Before reading on, answer:**
- What limit on the command itself, not on the operator's care, would have capped the damage?
- Why is a status dashboard hosted on the same infrastructure it reports on a design flaw, not a coincidence?
- How would you find out, safely, whether a subsystem that has run continuously for years would actually restart cleanly?
- What would you want logged or confirmed before an operational command executes against production capacity?

**What they did afterwards:** AWS published a detailed public postmortem, added safeguards to prevent capacity removal below defined minimums regardless of command input, and re-architected the status dashboard to run across multiple regions independent of any single region's health.

**The lesson in one line:** The scariest bugs are not in your application, they are in the scripts you trust because you have run them a hundred times before.

**Teach-back prompt:** Name one script or command you or a teammate runs regularly against a shared system. What is the worst input error that script currently allows, and what would stop it?

---

### PM-3 · Six hours, five backup systems, zero working backups (GitLab.com, January 31 2017)

**What happened:** GitLab.com's primary database was under load and its secondary replica had fallen behind, and engineers investigating assumed they were under a spam-driven traffic spike. To fix replication, an engineer set out to wipe the secondary's data directory and resync it fresh from the primary. The command was run against the wrong database, the primary, deleting roughly 300 gigabytes of live production data, including projects, issues, comments, and user accounts. Engineers then turned to backups and found that GitLab had five different theoretically overlapping mechanisms for this exact scenario, and effectively none of them were working: one produced backup files too small to be real, another had been failing silently for weeks, others simply did not run on the schedule believed. The only usable copy was a manual snapshot taken roughly six hours before the deletion. GitLab restored from it, meaning six hours of writes from thousands of users were permanently gone, and streamed the entire recovery process live on YouTube as it happened.

**The concept underneath:** A backup that has never been used to actually restore something is a hypothesis, not a backup, and the only way to know your recovery plan works is to rehearse it before you need it.

**Connects to modules:** M5 Data modelling and persistence; M6 SQL depth; M14 Containers, networking, and deployment; M11 Testing.

**Look up carefully:**
- Restore drills: scheduling regular, verified restores from backups, not just backup jobs
- Replication lag and the operational pressure it creates to "fix it now"
- Destructive commands and why production and non-production targets should be visually and mechanically hard to confuse
- Point-in-time recovery vs periodic snapshots, and what each actually promises
- Runbooks under pressure: why tired engineers doing an unfamiliar recovery procedure at 2am need the procedure written down in advance, not improvised
- Blameless postmortems: GitLab's own writeup is a widely cited example of the genre

**Before reading on, answer:**
- Which of the five failing backup mechanisms would a monthly restore drill have caught, and how quickly?
- What UI or command-line safeguard would make it hard to run a destructive command against the wrong host?
- Why does "we have backups in multiple places" not answer the question "can we recover"?
- If you had to recover from a six-hour-old backup right now, do you know what "six hours of data" means for the systems you touch?

**What they did afterwards:** GitLab published a fully public, detailed incident report the same week, built and now regularly exercises automated backup verification and restore testing, and the incident is one of the most frequently taught examples of "backups you have not tested are not backups" in the industry.

**The lesson in one line:** A backup you have never restored from is a rumor about your data, not a copy of it.

**Teach-back prompt:** Pick any database you have access to. Right now, could you say with confidence how old the most recent restorable backup is, and how you would know if that backup were silently broken?

---

### PM-4 · Forty-three seconds, twenty-four hours (GitHub, October 21 2018)

**What happened:** During routine maintenance to replace failing network equipment, GitHub's US East Coast data center lost network connectivity to the rest of GitHub's infrastructure for 43 seconds. That is barely enough time to notice, but GitHub's MySQL database topology was managed by an automated failover tool that treated the brief partition as the East Coast primary database being down, and it promoted a database on the West Coast to be the new primary so writes could continue. Connectivity came back a moment later, but by then both the East Coast and West Coast MySQL clusters had accepted writes independently, meaning the two copies of GitHub's data had diverged. Untangling which write was correct, replaying the right ones, and getting a single consistent primary back required roughly a day of significantly degraded service across issues, pull requests, and other core features while engineers manually reconciled the data.

**The concept underneath:** Automated failover reacts in seconds to conditions that a human would spend minutes just confirming, and when that speed advantage collides with a split-brain scenario, the automation can create a worse, harder-to-fix problem than the outage it was built to prevent.

**Connects to modules:** M6 SQL depth; M14 Containers, networking, and deployment; M8 Asynchronous messaging and real-time delivery; M10 Errors, logging, metrics, and traces.

**Look up carefully:**
- Split-brain in database failover: how two masters can both believe they are correct
- Automated failover tools and quorum-based leader election, and what they optimize for
- Cross-region database topology and the latency and consistency cost of allowing failover across regions
- Network partitions as a distributed-systems fundamental, not an edge case
- Brief flaps vs sustained outages: why failover logic needs a debounce, not just a trigger condition
- Reconciliation strategies once two copies of data have diverged

**Before reading on, answer:**
- Should a 43-second network blip trigger an irreversible action like a cross-region database failover? What would you require before allowing it to?
- What would prevent both clusters from accepting writes once the partition healed?
- How would you even detect that a split-brain had occurred, and how quickly?
- Is fast, automatic failover always the right default? When would you prefer a human in the loop even at the cost of a longer outage?

**What they did afterwards:** GitHub published a detailed post-incident analysis, changed their failover automation so it no longer promotes a primary across regional boundaries automatically, and invested in tooling to detect and prevent this class of data divergence earlier.

**The lesson in one line:** Automation that reacts faster than a human can double-check is only safe if it also refuses to act when it is not sure.

**Teach-back prompt:** Name one piece of automation in a system you use that can take an irreversible action without a human confirming it first. Should it be able to?

---

### PM-5 · A rule that was config on paper and code in practice (Cloudflare, July 2 2019)

**What happened:** Cloudflare's Web Application Firewall runs on every edge server worldwide and evaluates incoming requests against a large set of rules to block malicious traffic. An engineer deployed an update to one of those rules, intended to better detect malicious inline JavaScript, and it went out globally to the entire edge network at once, the way WAF rule updates normally did. The new rule contained a regular expression that, on certain inputs, triggered catastrophic backtracking, a pathological case where a regex engine's matching time explodes exponentially instead of scaling linearly with input size. Within minutes this drove CPU usage on edge servers across the world to nearly 100 percent, and Cloudflare, which sits in front of a large share of the internet's traffic, went down globally for about 27 minutes. A safeguard that would normally have limited how much CPU any single WAF rule could consume had been removed during an earlier, unrelated refactor of the WAF, so nothing stopped the runaway rule.

**The concept underneath:** A regular expression, a config value, or a rule file can behave exactly like code the moment it runs against real input, and it needs the same staged rollout, review, and resource limits that code gets, or a "config change" can take down everything at once.

**Connects to modules:** M4 Configuration, environments, and secrets; M14 Containers, networking, and deployment; M1 Execution model; M11 Testing.

**Look up carefully:**
- Regex engines and catastrophic backtracking, and why some engines guarantee linear-time matching and others do not
- CPU exhaustion vs memory exhaustion as distinct failure modes with different symptoms
- Global, simultaneous rollout vs staged or canary rollout, and what a canary would have caught here in seconds
- Kill switches for rule engines and WAFs specifically, separate from application deploy rollback
- Why "it's just a config change" is a dangerous sentence, and what review process config changes should get
- Resource limits and timeouts on rule evaluation, not just on request handling

**Before reading on, answer:**
- Why does treating a WAF rule update as "not a deploy" remove exactly the safeguards that would have caught this?
- What would a canary rollout of this rule to 1 percent of edge traffic have shown, and how fast?
- What resource limit, if enforced per rule, would have prevented global CPU exhaustion regardless of the regex?
- How do you test a regex for worst-case, not just typical-case, performance before it ships?

**What they did afterwards:** Cloudflare restored the missing CPU protection immediately, audited all existing WAF rules for similar risk, moved toward regex engines with linear-time guarantees, and committed to staged rollouts and performance testing for all future rule and configuration changes, not just code deploys.

**The lesson in one line:** If it runs against untrusted input in production, it is code, whatever you call the file it lives in.

**Teach-back prompt:** Find a config file, rule set, or feature-flag definition in a system you work on. Does a change to it go through the same review and rollout process as a code change? Should it?

---

### PM-6 · The outage that locked engineers out of the building (Facebook, October 4 2021)

**What happened:** During routine maintenance on Facebook's backbone network, an automated command intended to assess available capacity between data centers instead withdrew the routing announcements, BGP routes, for the network paths connecting Facebook's data centers to the rest of the internet. A bug in an auditing tool that should have caught the mistaken command failed to stop it. Losing those routes meant Facebook's DNS servers, still running perfectly fine internally, became unreachable from the outside internet, because nothing could route to them. Facebook, Instagram, and WhatsApp all disappeared from the internet globally for hours. The recovery was made dramatically worse because the internal tools engineers needed to diagnose and fix the network were themselves only reachable over that same now-broken network, and even physical badge-entry systems at some data centers depended on internal services that had gone down with everything else, so engineers had to arrange physical, hands-on access to specialized hardware to force the fix through.

**The concept underneath:** When your recovery tools, your monitoring, and even your building access all depend on the same system you are trying to fix, you have no way in when it breaks, so recovery paths must be built to survive the failure of the thing they recover.

**Connects to modules:** M14 Containers, networking, and deployment; M15 Security fundamentals; M10 Errors, logging, metrics, and traces.

**Look up carefully:**
- BGP fundamentals: route announcements, withdrawal, and why "unreachable" is different from "down"
- DNS and why a perfectly healthy server with no route to it is indistinguishable from a dead one
- Out-of-band access: management networks, break-glass procedures, and physical access paths independent of production
- Auditing and safety tooling for infrastructure-level commands, and why this one failed to catch the error
- Single points of shared fate: identifying every system that secretly depends on the thing you are about to change
- Blast radius at the scale of "the entire company's internal tooling," not just one service

**Before reading on, answer:**
- What would an out-of-band access path have looked like here, and why is it easy to skip building one until you need it?
- Why is "our monitoring shows everything is fine" not reassuring if the monitoring itself depends on the broken network?
- What check on the maintenance command itself would have stopped the BGP withdrawal before it executed?
- If your primary tools to fix an outage were unreachable right now, what is your actual fallback, concretely?

**What they did afterwards:** Facebook published a detailed engineering postmortem, added stronger safeguards and rollback mechanisms for backbone configuration changes, and invested in out-of-band infrastructure so critical tools and access do not depend entirely on the production network they manage.

**The lesson in one line:** Your emergency exit cannot be inside the room that catches fire.

**Teach-back prompt:** If the primary network or platform you rely on to operate a system went fully dark right now, how would you even communicate with your team, let alone fix it?

---

### PM-7 · A content update that crashed the kernel, not the app (CrowdStrike Falcon, July 19 2024)

**What happened:** CrowdStrike's Falcon security software runs with deep, kernel-level access on Windows machines to detect threats in real time, and it regularly receives small data files called channel files that update its detection logic without going through a full software release. One such update, Channel File 291, contained malformed data that should have been rejected by CrowdStrike's own content validation tooling, but a bug in that validator let it through. When Falcon's kernel driver tried to process the bad file, it performed an out-of-bounds memory read and crashed, and because it runs at the kernel level, the crash took the entire operating system down with it, the Windows "blue screen of death." The update was pushed to essentially all Windows machines running Falcon at once, since it was treated as routine content rather than a risky release. An estimated 8.5 million Windows devices worldwide were affected, crashing airlines, hospitals, banks, and broadcasters simultaneously. Because the crash happened before the machine could even reach the network, there was no way to push a fix remotely: recovery required physically or manually booting each affected machine into a recovery mode and deleting or renaming the bad file, one machine at a time.

**The concept underneath:** Anything that runs with kernel-level trust and updates automatically is exactly as risky as a code release, regardless of what you call it, and if your fix requires network access on a machine that can no longer reach the network, you have no remote recovery path at all.

**Connects to modules:** M4 Configuration, environments, and secrets; M14 Containers, networking, and deployment; M11 Testing; M2 Memory, lifecycle, and resource limits.

**Look up carefully:**
- Kernel-mode vs user-mode code, and why a crash in the former takes the whole machine down
- Content or configuration updates that bypass normal release review because they are not classified as "code"
- Staged and canary rollouts for automatic content updates, not just for application deploys
- Out-of-bounds memory reads and why type or bounds validation on input data matters even for "just a data file"
- Remote recoverability: what happens when the only fix requires access the broken state itself prevents
- Vendor blast radius: what it means when a single third-party agent has kernel access across your entire fleet at once

**Before reading on, answer:**
- Why does calling this a "content update" rather than a "release" matter for how much testing and staging it got?
- What staged rollout, even a fast one, would have limited this to a small fraction of the 8.5 million machines?
- Why couldn't CrowdStrike simply push a fix to already-crashed machines?
- If a vendor's agent runs with kernel-level access on every machine you operate, what questions should you be asking about their own deployment process?

**What they did afterwards:** CrowdStrike published a detailed public root cause analysis, fixed the validator bug, and committed to staged, canary-based rollouts for content updates going forward along with more rigorous testing of the update pipeline itself, treating content updates with the same rigor as code releases.

**The lesson in one line:** "It's just a data update, not a code deploy" is the sentence right before the outage that reaches every machine at once.

**Teach-back prompt:** Does any agent, plugin, or dependency in your systems auto-update itself with elevated privileges, without going through your own release process? What happens if its next update is bad?

---

### PM-8 · The credentials that were never supposed to be in the repo (Uber, breach 2016, disclosed 2017)

**What happened:** In October 2016, attackers obtained credentials that Uber engineers had stored in a private GitHub repository. Those credentials included access keys to Uber's cloud storage, and the attackers used them to reach an Amazon S3 bucket holding rider and driver data, downloading personal information, including names, email addresses, phone numbers, and driver's license numbers, for roughly 57 million people. Rather than disclosing the breach to regulators, affected users, or the public, Uber's security team contacted the attackers directly, paid them 100,000 dollars, and had them sign a nondisclosure agreement, structuring the payment to look like a legitimate bug bounty reward. The breach stayed hidden for about a year until new company leadership investigated and disclosed it publicly in November 2017. Uber fired its chief security officer and a senior security lawyer involved in handling the incident. Years later, the former CSO was criminally prosecuted and convicted of obstruction of justice for his role in concealing the breach from regulators who were actively investigating an earlier, separate Uber security incident at the time.

**The concept underneath:** Secrets stored anywhere a broader set of people can read than the resource they unlock actually requires, including "private" repositories, are one leaked credential away from becoming an incident, and how an organization responds to a breach is itself subject to law, not just judgment.

**Connects to modules:** M4 Configuration, environments, and secrets; M15 Security fundamentals; M9 Authentication and identity; M16 Git, code review, and delivery.

**Look up carefully:**
- Secret scanning: automated detection of credentials committed to any repository, private or not
- Least privilege for cloud credentials, and scoping keys so a single leaked key cannot reach everything
- "Private" repository access in practice: who actually has access, and how that set grows over time
- Credential rotation and why a key that could plausibly have leaked must be rotated even if you are not sure
- Legal and regulatory breach disclosure obligations, and why concealment is a separate, additional offense from the breach itself
- Bug bounty programs and why paying an attacker outside that structure is not the same thing, however it is worded

**Before reading on, answer:**
- Why does "private repository" not mean "safe place for a credential"?
- What scope should the leaked key have had so that even if stolen, it could not reach 57 million records?
- What would automated secret scanning on that repository have caught, and how quickly, compared to a year?
- Separate from the breach itself, why did the response to it create additional, distinct legal consequences?

**What they did afterwards:** Uber settled with US state attorneys general and the FTC, paid substantial fines, implemented mandatory breach disclosure processes and stronger credential and access management, and the case became a landmark example establishing that concealing a breach can carry personal criminal liability for the executives involved, not just corporate consequences.

**The lesson in one line:** A secret in a private repository is a secret exactly until someone you didn't plan for reads that repository, and how you respond to a breach is a decision with its own consequences, separate from the breach itself.

**Teach-back prompt:** If a credential leaked from a private repository you have access to today, would you find out from a scanner, or from an attacker?

---

## Patterns across all eight

| Pattern | Cases |
|---|---|
| Config or content update behaves exactly like a code release | PM-1 (flag), PM-5 (WAF rule), PM-7 (channel file) |
| Blast radius was global instead of staged | PM-1, PM-5, PM-7 |
| Backups or redundancy existed on paper but were never verified | PM-3 |
| The tool meant to detect or fix the problem depended on the thing that broke | PM-2 (status dashboard), PM-6 (internal tools and badge access) |
| Automation acted faster than a human could sanity-check it | PM-1, PM-4, PM-6 |
| A safeguard existed once and was quietly removed or never wired up | PM-1 (repurposed flag), PM-5 (missing CPU guard) |
| A single mistaken input had no limit on how much damage it could do | PM-2, PM-6 |
| Secrets or access were broader than the task required | PM-8 |
| Recovery required manual, per-machine or physical action because remote access was gone | PM-6, PM-7 |
| How the organization responded became part of the incident | PM-8 |

## Writing your own postmortem

```
Summary
Impact (who, how long, how bad)
Timeline (with times)
Root cause(s)
Contributing factors
Detection (how we found out, and how long it took)
What went well
What went poorly
Action items (each with owner and due date)
Lessons for the curriculum
```

Write it blameless. Describe systems and decisions, not people: not "X deleted the database" but "the recovery runbook did not distinguish primary from secondary at the point the destructive command was run." For every decision in the timeline, ask "what made this the reasonable thing to do at the time," not "how could they have missed that." The people involved almost always had incomplete information, reasonable assumptions, and normal pressure, exactly like you will the day this happens to you.
