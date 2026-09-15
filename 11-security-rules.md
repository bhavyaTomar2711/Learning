# 11 · Security Rules (Intern Edition)

These rules are non-negotiable from day one. They apply equally to you and to any AI tool you use, whether it is writing code, reading files, or running commands on your behalf. Follow them as standard practice, not as a reaction to any specific event.

## 1. Core Principles

- **Assume anything sent over chat or email can leak.** Never rely on a private channel to keep something safe.
- **Least privilege.** You should only ever have the access your current task needs, nothing more. The same applies to any AI agent you run.
- **Layered defence.** No single habit or tool keeps you safe on its own. Good practices stack: careful secrets handling, careful git habits, careful access requests, careful AI use.
- **AI agents are not automatically trustworthy.** A coding agent can read every file in your project, run shell commands, and make network calls. Treat it like a capable but unsupervised contractor: useful, but you stay responsible for what it does.

## 2. Using AI Tools Safely

You will use Cursor, ChatGPT, Claude, and similar tools constantly. Used well, they are a huge advantage. Used carelessly, they are a fast way to leak something you shouldn't.

**Never put these into a prompt, chat window, or AI tool context:**
- Passwords, API keys, tokens, or any credential, even "just to debug"
- The contents of a `.env` file
- Customer data, personal data, or anything that identifies a real person
- Production logs that have not been redacted (they often contain user data, tokens, or internal identifiers)
- Database dumps or exports of any kind

If you need help debugging something that touches one of these, sanitize it first: replace real values with placeholders, strip real names and emails, remove tokens entirely.

**Review AI-generated code before you trust it.** Read it the way you would review a teammate's pull request. Check that it does what you asked, not something adjacent to it. Do not merge code you don't understand just because it runs.

**Never let an AI agent run destructive or irreversible commands on your behalf without you reading them first**, things like deleting files, force-pushing, dropping data, or restarting a service. Auto-approve modes that skip your confirmation are fine for low-risk, local, read-only work (writing a test, drafting docs) and should never be used for anything that touches shared systems, secrets, or production-adjacent work.

**Don't grant AI tools broad access.** If a tool or plugin asks for a token or API key, give it the narrowest scope available, not an admin-level or all-repos token. If you're not sure what scope something needs, ask Yash before granting it.

## 3. Secrets and Credentials

A "secret" is any password, API key, token, private key, or connection string.

- **Never share a secret in chat, email, a screenshot, a commit, a ticket, or an AI prompt.** There is no exception for "just this once" or "I'll delete it after."
- **Secrets belong in two places only:** environment variables injected by the platform at deploy time (not committed anywhere), or a local `.env` file on your own machine that is listed in `.gitignore` and never tracked by git.
- **Check `.gitignore` before you start a project.** Confirm `.env`, `.env.local`, and any file with `key`, `secret`, `credentials`, or `password` in its name is excluded.
- **If you accidentally commit or paste a secret anywhere:** tell Yash immediately. Do not try to quietly delete the message, rewrite git history, or fix it yourself. Once a secret has been exposed, even briefly, it is treated as leaked and gets rotated. Acting fast is what limits the damage, silence does not.
- **Some older repositories may still contain configuration values that were committed in the past.** If you ever come across something in a repo that looks like a real key, password, or token, do not copy it, reuse it, test it, or share it anywhere, including in a prompt to an AI tool. Report it to Yash and move on.

## 4. Code and Git

- Respect branch protection. If `main` requires a pull request and a review, that is not a formality to route around.
- Never force-push to a shared branch. If you think you need to, stop and ask first.
- Don't add a new dependency because it looked convenient. Every package you add is something the whole team now has to trust and maintain. If you need one, say why in the pull request.
- Review your own diff before opening a pull request, and read through any suggested changes before merging, whether they came from a teammate or an AI tool.
- Keep commits focused and messages clear enough that someone else can understand what changed and why.

## 5. Access

- You get exactly the access your role needs, nothing broader. If a task seems to need more access than you have, ask, don't work around it.
- Never share your account, your login session, or your credentials with anyone else, including another intern. Your access is tied to your identity and everything you do with it is logged.
- For your first week, you will not have infrastructure access. Starting Day 8, you get **read-only** access to some production systems. Read-only means:
  - You can look at data to understand how things work.
  - You must never run a write, update, insert, or delete query against production data, even "just to test something."
  - You must never restart, stop, or redeploy a production service.
  - If you need to change something or need write access for a specific task, ask Yash first. Don't attempt it because you technically found a way.

## 6. Laptop Hygiene

- Keep full-disk encryption enabled (BitLocker on Windows, FileVault on Mac).
- Lock your screen every time you step away, and set your OS to auto-lock after a short idle period.
- Keep your operating system and browser up to date. Don't defer updates indefinitely.
- Use a password manager for all work accounts. Don't reuse passwords across services.
- Enable two-factor authentication on your GitHub account and your work email. This is mandatory, not optional.
- Don't store company code, credentials, or documents on a personal cloud drive (personal Google Drive, Dropbox, iCloud, etc). Use only company-approved storage.

## 7. Reporting

If something looks off, say something right away. That includes:
- A password, key, or token you spot sitting in a repository, ticket, or chat
- A login or account-activity email you weren't expecting
- An API endpoint, dashboard, or file that seems reachable by more people than it should be
- Anything an AI tool did that you didn't expect or can't explain

Report it to Yash immediately. Reporting is always the right call. You will never get in trouble for flagging something, even if it turns out to be nothing. Staying quiet because you're not sure, or because you're worried about looking inexperienced, is the only wrong move here.

## 8. Quick Checklist

**Day 0**
- [ ] Password manager installed and set up
- [ ] Two-factor authentication enabled on GitHub
- [ ] Two-factor authentication enabled on work email
- [ ] Disk encryption confirmed on (BitLocker / FileVault)
- [ ] Screen auto-lock configured
- [ ] `.gitignore` checked in any repo you touch (`.env` and credential-like filenames excluded)
- [ ] No company code saved to a personal cloud drive

**Day 8**
- [ ] Confirmed your new access is read-only
- [ ] Understand: look, don't change, on production systems
- [ ] Know never to run write queries against production data
- [ ] Know never to restart or redeploy a production service
- [ ] Know to sanitize anything touching real data before putting it in an AI prompt
- [ ] Know to contact Yash immediately for any secret exposure or anything that looks wrong
