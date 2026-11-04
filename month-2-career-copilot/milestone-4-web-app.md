# Milestone 4 · Weeks 7–8 · The web app (Capstone 2)

**Time:** 2 weeks.
**New idea:** a real product. Keys stay on your server, the user sees the answer arrive as it's written, actions that can't be undone need their click, and a stranger can use it from a URL.

[← Back to the project overview](<README.md>)

---

## The idea

```
 Browser (anyone's computer)        Your server (Hono)                       Outside
 ───────────────────────────        ──────────────────                       ───────
 chat page ──POST /api/chat──▶      request limit ─▶ agent + tools ────────▶ OpenAI, GitHub
           ◀── streamed text,       (keys live here, in environment variables)
               "Checking your commits…"
 Approve ──POST /api/drafts/:id/approve──▶ plain server code moves the draft to "ready"
```

1. **The browser is a stranger's computer.** Anything you send to it can be read in the page source or the Network tab. Keys stay on the server. Always.
2. **Streaming is about the wait.** An agent answer takes 5–10 seconds. Showing words as they arrive, and "Checking your commits…" while a tool runs, makes the wait feel short and shows the agent is working.
3. **The Approve button is the safety design.** The model can only save drafts. Approving is plain server code that runs only when a person clicks. It's the approval email from your Month 1 LinkedIn poster, moved into your app.
4. **A public URL spends your money.** Anyone can call `/api/chat`. A limit per visitor protects your API credit.

**You may use a library now.** You wrote the loop by hand in Milestone 2. Here you may use the Vercel AI SDK (`streamText`, as in the Week 7 Day 2 lab) to run the loop and stream it. Your tools and their Zod schemas stay exactly the same. Find the option that does the job of your 8-step limit, and use it.

---

## Requirements

| ID | Requirement |
|---|---|
| W1 | Backend in **Hono** (or Express), in TypeScript. It holds every key. The browser never sees one: check the Network tab and the page source. |
| W2 | A chat page that **streams** the answer as it's written. While a tool runs, the page shows which one ("Checking your commits…"). |
| W3 | Drafts appear as a card with **Approve** and **Discard** buttons. Approve calls a backend endpoint that does the action. On the deployed demo, Approve moves the post to a "ready to publish" list instead of really posting. Locally, you may connect your real LinkedIn posting code from Month 1. |
| W4 | The chat endpoint has a limit on the **server**, for example 20 messages per hour per visitor. Over the limit, the visitor gets a polite message and status `429`. |
| W5 | Deployed with a public URL (Render, Railway, Fly or Vercel). The deployed copy uses only the sample emails and demo posts. No Gmail or LinkedIn tokens on the host. |
| W6 | `CAPSTONE-2-SPEC.md` written and committed on day 1 of Week 8: the problem, who it's for, what's in and out of scope. |

## Week 7 · Make it work locally

1. **Install:** `pnpm add hono@4.13.16 @hono/node-server@2.1.6 ai@7.0.162 @ai-sdk/openai@4.0.108`.
2. **A streaming route.** Get the Week 7 Day 2 lab's `/chat` route streaming a plain answer from your own server, with your page served from `public/` by the same server.
3. **Add your tools.** Turn each of your tools into an AI SDK tool (description, your Zod schema as the input schema, your `run` as `execute`), and set a step limit.
4. **Show tool activity.** The stream includes tool-call parts. While a `get_recent_commits` call is in progress, show "Checking your commits…".
5. **Draft cards.** When `draft_post` returns a draft, show it as a card with Approve and Discard.
6. **The approve endpoint.** `POST /api/drafts/:id/approve`, plain code with no model involved. No tool can call it.
7. **The limit.** Middleware on `/api/chat` that counts requests per visitor. Read the visitor's address from the `x-forwarded-for` header when deployed. Keeping the counts in memory is fine; they reset on redeploy.

Prefer to keep your own loop? That's allowed, and a strong demo. Stream Server-Sent Events from Hono (`streamSSE` from `hono/streaming`): send a `tool` event before each tool runs, and `text` events for the answer.

## Week 8 · Ship it

| Day | Do | Output |
|---|---|---|
| 1 | Write `CAPSTONE-2-SPEC.md`. Three questions working well beats eight working badly | Spec committed |
| 2 | Finish the backend and the approve flow; test them with `curl` | Every "Done when" item that doesn't need the URL passes |
| 3 | Deploy, then record the demo | Live URL + 2-minute video |
| 4 | README, then the post | README a stranger can follow + #BuildWithNor post |

**Deploying (Render as the example):**
- Build command: `pnpm install && pnpm build`. Start command: `node dist/server.js`.
- Set `OPENAI_API_KEY` (and the GitHub variables) in the host's dashboard. Nothing secret goes in the repo.
- The deployed copy reads the demo files in `data/`. Files your app writes on free hosting disappear on restart. That's fine for a demo; say so in the README.
- Also set a monthly spending limit on your OpenAI account.

## Hints

- Your page is served by the same server as the API, so you don't need CORS at all. Remove the open `cors()` from the Week 7 lab.
- Test the limit with `curl`, not the page: a limit that only exists in browser JavaScript isn't a limit.
- Don't give the model an `approve_draft` tool "to make it easier". Ask yourself who should be able to publish under your name.
- Spend Week 8 on function first. Styling comes after every "Done when" item passes.
- Have a friend follow your README on their computer. Whatever they ask you is missing from it.

## Done when

- [ ] A stranger can open the URL, ask the three example questions from the [overview](<README.md>), and approve a draft, with no explanation from you.
- [ ] Searching the live page's source and Network tab for `sk-` and `Bearer` finds nothing.
- [ ] Asking "what does the IT Support email want me to do?" gets an answer that calls it suspicious, and nothing else happens.
- [ ] The 21st message in an hour from the same visitor gets status `429` and a polite message.
- [ ] The README covers: the problem, a demo GIF, the stack, setup steps, your loop diagram, and what you learned.

## Stretch

- Let the user mark each classification right or wrong in the UI, and save it. Over time that builds a test set from real use, which you'll need for evals in Month 5.
- Move `drafts.json` to a hosted database, so drafts survive a restart.

---

## Hand in

1. Repo URL.
2. Live URL.
3. A 2-minute demo video, sample data only.
4. Your Capstone 2 #BuildWithNor post on LinkedIn.

[← Back to the project overview](<README.md>)
