# Month 2 Project · Career Copilot

**When:** Weeks 5–8 of the Agentic AI Developer Program.
**Time:** 4 weeks, one milestone per week. It becomes your **Capstone 2**.
**Language:** TypeScript. This is where you move from n8n to code.
**You hand in:** one GitHub repo that grows every week, plus a live URL at the end.

In Month 1 you built **pipelines**: a fixed list of steps that runs the same way every time. This month you build an **agent**: a program where the model decides which tools to use, and your code decides what it's allowed to do.

You build **one product over four weeks**, not four throwaway exercises. Each week adds a layer on top of the last, and reuses what you made in Month 1.

---

## What you're building

**Career Copilot** is a web app you chat with about your job hunt and your build-in-public habit:

> "Anything from recruiters today?"
> "Write today's LinkedIn post from what I pushed to GitHub."
> "Did I already post about Zod this week?"

It reads, sorts and drafts. It **never** sends an email or publishes a post on its own. Those stay a button **you** press.

```
            Week 5              Week 6                  Weeks 6–7               Weeks 7–8
        ┌─────────────┐   ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────────┐
        │ Your poster,│──▶│ Email classifier │──▶│ The same tools   │──▶│ Web app: streaming   │
        │ every post  │   │ + agent loop with│   │ as an MCP server │   │ chat, Approve button,│
        │ checked with│   │ 5 tools, written │   │                  │   │ deployed (Capstone 2)│
        │ Zod         │   │ by hand          │   │                  │   │                      │
        └─────────────┘   └──────────────────┘   └──────────────────┘   └──────────────────────┘
```

## The milestones

| # | Week | Milestone | New idea | Guide |
|---|---|---|---|---|
| 1 | 5 | The post drafter | Never trust a model's answer until your code has checked it. Measure how often it's good. You upgrade your Month 1 LinkedIn poster | [milestone-1-post-drafter.md](<milestone-1-post-drafter.md>) |
| 2 | 6 | The agent loop | Reuse the same check-and-retry for an email classifier, then write the tool-use loop by hand | [milestone-2-agent-loop.md](<milestone-2-agent-loop.md>) |
| 3 | end of 6, start of 7 | The MCP server | Write tools once, use them from any app | [milestone-3-mcp-server.md](<milestone-3-mcp-server.md>) |
| 4 | 7–8 | The web app | A real product: safe keys, streaming, a public URL | [milestone-4-web-app.md](<milestone-4-web-app.md>) |

Start each milestone only when the previous one's **Done when** list passes.

---

## The rules for the whole month

These apply to every milestone. Breaking one means the milestone isn't done, however good the rest is.

1. **Secrets.** Create `.gitignore` (with `node_modules`, `dist` and `.env`) **before your first commit**. No key, token or real email address is ever in the repo or its history. If a key was ever pushed, even for a minute, cancel it and make a new one. Deleting the file doesn't help: git keeps every old version.
2. **Check before you trust.** Every model answer your code depends on is checked with Zod first. A failed check never crashes the program and is never treated as a valid answer.
3. **The model doesn't get the dangerous buttons.** No tool sends email, publishes a post or deletes anything. Those actions exist only as something a person triggers by pressing a button.
4. **You can explain every line.** At the end of each milestone you'll have a 15-minute call with your instructor. You walk through your code and make one small change without AI help. Using AI coding tools is fine; handing in code you can't explain isn't.
5. **Test data is fake.** Your demo, screenshots and deployed app use the sample emails in this repo, never your real inbox.

---

## Getting started

1. Create a new **empty** GitHub repo called `career-copilot`, and clone it.
2. Copy everything from [`starter/`](<starter/>) into it, including the hidden `.gitignore`.
3. Check that `data/` has `notes.json` and `emails.json` (the 10 fake sample emails from the n8n project P3). You'll need the emails in Milestone 2.
4. Copy `.env.example` to `.env` and add your model key (OpenAI, or Gemini if your poster uses it).
5. Bring in your Month 1 LinkedIn poster code under `src/`, or keep it in its own repo and copy in only the drafting part. Milestone 1 upgrades it.
6. Run `git status`. If `.env` is listed, **stop** and fix `.gitignore` before committing anything.
7. Run `pnpm install` (or `npm install`), then make your first commit.
8. Open [Milestone 1](<milestone-1-post-drafter.md>).

You need Node 22 or newer. The model names and prices are in the n8n projects' [CONVENTIONS.md](https://github.com/SoftServedWeb-com/n8n-projects-norlabs/blob/main/CONVENTIONS.md).

## Folder layout you'll end up with

```
career-copilot/
  .gitignore  .env.example  package.json  tsconfig.json  README.md
  data/
    notes.json           10 fixed "what I learned" notes, for scoring the drafter
    emails.json          the 10 sample emails (fake)
    posts.json           your last few LinkedIn posts (public text only)
    drafts.json          created by the draft_post tool
  src/
    post/                Milestone 1: schema.ts, build.ts, build.test.ts, draft.ts, score.ts
    triage/              Milestone 2: classify.ts, route.ts, route.test.ts, score.ts
    agent/               Milestone 2: tools.ts, execute.ts, loop.ts, cli.ts
    mcp/                 Milestone 3: server.ts
    server.ts            Milestone 4: the Hono backend
  public/                Milestone 4: index.html, the chat page
  CAPSTONE-2-SPEC.md     Milestone 4, written on day 1 of Week 8
```

---

## What to hand in at the end

1. **Repo URL** with a README a stranger can follow: the problem, a demo GIF, the stack, setup steps, a diagram of your agent loop, and what you learned.
2. **Live URL.**
3. **A 2-minute demo video** using sample data only.
4. **Your Capstone 2 #BuildWithNor post** on LinkedIn.

## How it's checked

| # | Criterion | Milestone |
|---|---|---|
| 1 | No secrets in the repo or its history, at any point in the month | All |
| 2 | Drafter: every post checked with Zod and built by code; all 9 non-empty notes pass; a failed draft is never posted | 1 |
| 3 | Classifier gets 9/10 or better on the samples, with the scam and prompt-injection emails sent to review | 2 |
| 4 | Hand-written loop with Zod-checked tool arguments, a step limit and readable tool errors | 2 |
| 5 | MCP server used from two different clients, sharing one copy of the tool code | 3 |
| 6 | No tool can send, publish or delete; those actions need a button press | 2, 4 |
| 7 | Streaming chat UI, keys only on the backend, and a request limit | 4 |
| 8 | Deployed, and a stranger can use it from the README alone | 4 |
| 9 | Passes all four code walkthrough calls | All |

## What comes after

In Month 3 (Weeks 9–12) you add memory and retrieval. Career Copilot's `get_previous_posts` becomes a search over all your posts and notes, and your Capstone 3 can grow out of this same repo.
