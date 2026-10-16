# n8n Projects · Norlabs Agentic AI Developer Program

26 real automation projects you build in **n8n**, each with a complete requirements document. Each one solves a problem you actually have while building a career in AI: finding jobs, answering recruiters, preparing for interviews, building in public, learning faster, or earning from freelance automation work.

Every project runs on **your own laptop**, in a free, self-hosted n8n. It uses OpenAI for the AI steps, Telegram to talk to you, and Google Sheets for data you can read.

---

## Start here

1. **Set up n8n once:** follow [SETUP.md](<SETUP.md>). It takes about an hour.
2. **Build the core four, in this order:**
   1. [P1 · Build-log → LinkedIn with approval](<projects/p1-build-log/REQUIREMENTS.md>) turns your real GitHub commits into LinkedIn posts that you approve before they go out.
   2. [P3 · Recruiter inbox triage](<projects/p3-inbox-triage/REQUIREMENTS.md>) gets recruiter emails answered the same day.
   3. [P2 · Job-hunt radar](<projects/p2-job-radar/REQUIREMENTS.md>) gives you a daily list of jobs that fit you, and the skills you're missing.
   4. [P4 · Interview-prep quiz bot](<projects/p4-quiz-bot/REQUIREMENTS.md>) builds your interview-prep bank one question a day.
3. **Then pick from the project bank** by goal (below).

You can start a project once you've reached its **Ready after** day in the program; each project's table row and header say which day that is. Do the projects in Hour 5, as homework, or on weekends. They're not graded.

---

## How to use a requirements document

Each project has one `REQUIREMENTS.md`. It tells you **what** to build and how you'll know it works. It doesn't give you the workflow: designing that is the exercise.

| Section | What it gives you |
|---|---|
| Header table | Difficulty, time, when you're ready, whether it runs fully on a laptop |
| 1–3 | The problem, who it's for, and what's in and out of scope |
| 4 · Functional requirements | Numbered `FR-` rules your workflow must follow. Build to these |
| 5–6 | The data (Sheets, tables, columns) and every AI step: model, JSON output shape, prompt rules |
| 7 · Non-functional requirements | Catch-up, cost, privacy, platform rules |
| 8 · Credentials & setup | Exactly which accounts and permissions it needs |
| 9 · Suggested building blocks | A flow sketch and the n8n nodes that fit. A suggestion, not the only way |
| 10 · Milestones | Build it in stages (v1, v2, v3). Each stage has a "done when" |
| 11 · Acceptance tests | Run these. When they all pass, you're done |
| 12–15 | Stretch goals, what to hand in, known gotchas, and how it could seed a capstone |

**Reference solutions:** P1–P4 will get a reference workflow later, in their `reference/` folder. Bank projects are requirements only.

---

## Repository layout

```
README.md          ← you are here
SETUP.md           ← install n8n + credentials (do this first)
CONVENTIONS.md     ← versions, models, limits: the single source for all projects
setup/
  docker-compose.yml   n8n 2.43.7, bound to your laptop only
  .env.example         copy to .env and add bot tokens here
  files/               files your workflows read or write
projects/
  p1-build-log/REQUIREMENTS.md
  ...
  b22-ask-your-notes-bot/REQUIREMENTS.md
```

---

## All projects

**Key to the Laptop column:**
- ✅ runs fully on your laptop.
- 📨 receives Telegram messages by polling, so it needs its own bot (see [SETUP.md, Step 6](<SETUP.md>)).
- 🌐 fine to build and demo on a laptop; serving real customers needs an always-on host.

### Core four

| # | Project | Category | Difficulty | Time | Ready after | Laptop |
|---|---|---|---|---|---|---|
| P1 | [Build-log → LinkedIn with approval](<projects/p1-build-log/REQUIREMENTS.md>) | Build in public | ★★☆ | 6–8 h | W2 D3 | ✅ |
| P2 | [Job-hunt radar](<projects/p2-job-radar/REQUIREMENTS.md>) | Career | ★★☆ | 6–8 h | W2 D2 / D3 | ✅ |
| P3 | [Recruiter inbox triage](<projects/p3-inbox-triage/REQUIREMENTS.md>) | Career | ★★☆ | 5–7 h | W2 D3 | ✅ |
| P4 | [Interview-prep quiz bot](<projects/p4-quiz-bot/REQUIREMENTS.md>) | Career | ★★★ | 8–10 h | W2 D3 | 📨 |

### Project bank

| # | Project | Category | Difficulty | Time | Ready after | Laptop |
|---|---|---|---|---|---|---|
| b01 | [Application tracker](<projects/b01-application-tracker/REQUIREMENTS.md>) | Career | ★★☆ | 4–6 h | W2 D3 | ✅ |
| b02 | [Resume tailor](<projects/b02-resume-tailor/REQUIREMENTS.md>) | Career | ★★☆ | 5–7 h | W2 D3 | 📨 |
| b03 | [Outreach drafter](<projects/b03-outreach-drafter/REQUIREMENTS.md>) | Career | ★☆☆ | 3–4 h | W2 D3 | ✅ |
| b04 | [Mock-interview debrief](<projects/b04-mock-interview-debrief/REQUIREMENTS.md>) | Career | ★★☆ | 4–6 h | W2 D3 | 📨 |
| b05 | [Evening check-in](<projects/b05-evening-check-in/REQUIREMENTS.md>) | Build in public | ★☆☆ | 3–4 h | W2 D2 | 📨 |
| b06 | [Post repurposer](<projects/b06-post-repurposer/REQUIREMENTS.md>) | Build in public | ★☆☆ | 2–3 h | W2 D3 | ✅ |
| b07 | [Repo hygiene reviewer](<projects/b07-repo-hygiene-reviewer/REQUIREMENTS.md>) | Build in public | ★★☆ | 4–6 h | W3 D1 | ✅ |
| b08 | [AI news brief](<projects/b08-ai-news-brief/REQUIREMENTS.md>) | Learning | ★☆☆ | 3–4 h | W2 D3 | ✅ |
| b09 | [Read-later inbox](<projects/b09-read-later-inbox/REQUIREMENTS.md>) | Learning | ★★☆ | 4–6 h | W2 D3 | 📨 |
| b10 | [Lecture-notes maker](<projects/b10-lecture-notes-maker/REQUIREMENTS.md>) | Learning | ★★☆ | 5–7 h | W2 D3 | ✅ |
| b11 | [Daily planner](<projects/b11-daily-planner/REQUIREMENTS.md>) | Learning | ★☆☆ | 3–4 h | W2 D3 | ✅ |
| b12 | [Lead qualifier](<projects/b12-lead-qualifier/REQUIREMENTS.md>) | Freelance | ★★☆ | 4–6 h | W2 D3 | ✅ |
| b13 | [FAQ support bot](<projects/b13-faq-support-bot/REQUIREMENTS.md>) | Freelance | ★★★ | 6–8 h | W3 D2 | 📨 / 🌐 |
| b14 | [Invoice & receipt extractor](<projects/b14-invoice-extractor/REQUIREMENTS.md>) | Freelance | ★★☆ | 5–7 h | W3 D1 | ✅ |
| b15 | [Appointment reminders](<projects/b15-appointment-reminders/REQUIREMENTS.md>) | Freelance | ★★☆ | 5–7 h | W2 D2 | 📨 / 🌐 |
| b16 | [Review-reply drafter](<projects/b16-review-reply-drafter/REQUIREMENTS.md>) | Freelance | ★☆☆ | 3–4 h | W2 D3 | ✅ |
| b17 | [#BuildWithNor tracker](<projects/b17-buildwithnor-tracker/REQUIREMENTS.md>) | Cohort tools | ★☆☆ | 3–4 h | W2 D2 | ✅ |
| b18 | [Homework checker](<projects/b18-homework-checker/REQUIREMENTS.md>) | Cohort tools | ★★☆ | 4–6 h | W3 D1 | ✅ |
| b19 | [Question router](<projects/b19-question-router/REQUIREMENTS.md>) | Cohort tools | ★★★ | 6–8 h | W2 D3 | 📨 / 🌐 |
| b20 | [Personal assistant agent](<projects/b20-personal-assistant-agent/REQUIREMENTS.md>) | Later in the course | ★★★ | 8–10 h | W5 D3 | 📨 |
| b21 | [Job radar as an MCP tool](<projects/b21-job-radar-mcp-tool/REQUIREMENTS.md>) (needs P2) | Later in the course | ★★★ | 4–6 h | W6 D2 | ✅ |
| b22 | [Ask-your-notes bot](<projects/b22-ask-your-notes-bot/REQUIREMENTS.md>) | Later in the course | ★★★ | 8–12 h | W11 D1 | 📨 |

### Pick by goal

| Your goal | Pick from |
|---|---|
| Getting hired | b01, b02, b03, b04 |
| Building in public | b05 (feeds P1), b06, b07 |
| Learning faster | b08, b09, b10, b11 |
| Freelance income, or the "AI Automation Specialist" role | b12 (the cleanest Capstone 1 shape), then b13–b16 |
| Helping your cohort | b17, b18, b19 |
| Later months | b20 (Week 5), b21 (Week 6), b22 (Weeks 10–11) |

---

## Rules every project follows

You'll see these in every requirements document. Learn the pattern once.

1. **Catch up, never duplicate.** Your laptop sleeps. Every scheduled workflow handles "everything since the last successful run", skips what it has already done, and acts once a day even if it's checked every hour.
2. **AI output is JSON, checked.** Every AI step returns a stated JSON shape. Invalid output gets one retry, then a fallback branch. It's never treated as valid.
3. **A human approves what goes out.** Anything AI-written that will be published or sent to another person waits for your approval. Email replies are saved as Gmail drafts, never sent automatically.
4. **No public address needed.** Projects that receive Telegram messages poll for them, each with its own bot. Telegram only keeps undelivered messages for 24 hours, so a polling bot catches up after a night off but not after a weekend.
5. **Cheap first.** Filter before calling a model, cap each run, and use the fast or default model from [CONVENTIONS.md](<CONVENTIONS.md>).
6. **Know what leaves your laptop.** Each document says which data goes to OpenAI or Google. Never export a workflow with tokens, chat IDs or email addresses in it.
7. **Platform rules.** LinkedIn only through its official API, for your own posts. No scraping LinkedIn, Naukri or Indeed. No automated likes, comments, connections or messages.

---

## Left out on purpose

| Idea | Why not |
|---|---|
| LinkedIn auto-connect, auto-comment, auto-DM, or profile/job scraping | Against LinkedIn's User Agreement; accounts get restricted |
| Scraping Naukri or Indeed | Same problem. P2 uses public job-board APIs instead |
| WhatsApp bots | WhatsApp's official API needs Meta Business verification, too heavy to start with. Telegram first |
| Summarising other people's YouTube videos | Transcript access breaks often. b10 uses your own recordings |

---

## Showing your work

- Build in your own n8n. Commit each project's exported workflow (see [SETUP.md, "Exporting your work safely"](<SETUP.md>)), its Sheet template and a README to **your own** GitHub repository.
- Record a 60-second demo of it firing end to end.
- Post it as a **#BuildWithNor** update.
- **Capstone 1:** a project as written here is not a capstone. It can seed yours if your spec adds something new (a trigger, a source or an action). Each document's section 15 suggests options.

---

## About this repository

This repository is published from the course's source material. **Don't open pull requests here**: they'd be overwritten by the next update. Found a mistake, or a link or API that has changed? Tell your instructor.

Versions and prices were checked on **2026-10-16**. See [CONVENTIONS.md](<CONVENTIONS.md>).
