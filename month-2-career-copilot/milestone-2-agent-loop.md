# Milestone 2 · Week 6 · The email classifier and the agent loop

**Time:** about 5 evenings: 2 for the classifier, 3 for the loop.
**New ideas:** reusing a pattern (the check-and-retry from Milestone 1, made general), and the **agent loop**. The model asks for a tool, your code checks the request and runs it, the result goes back to the model, and this repeats until there's an answer. You set the limits.

[← Back to the project overview](<README.md>)

---

## Part A · The email classifier (two evenings)

Your agent needs to read your inbox. Here you build the piece that decides what each email is, using the same wall you built in Milestone 1.

### Make your check-and-retry reusable

In Milestone 1 you wrote parse → Zod check → retry once with the error → fail cleanly, for posts. You now need the same thing for emails. Don't copy it. Turn it into one function that works with **any** schema:

```ts
// src/lib/generate-checked.ts
export type Turn = { role: "user" | "assistant"; content: string };
export type Ask = (turns: Turn[]) => Promise<string>; // calls your model, returns its raw reply

// Asks the model, checks the reply against `schema`, retries once with the Zod error.
export async function generateChecked<S extends z.ZodType>(
  schema: S,
  firstMessage: string,
  ask: Ask,
): Promise<{ ok: true; data: z.infer<S>; attempts: number } | { ok: false; reasons: string[] }>
```

Two TypeScript ideas make this work. `<S extends z.ZodType>` is a **generic**: the return type follows whatever schema you pass in, so posts give you a `Post` and emails give you a `Triage`. And `ask` is a **function you pass in**: one `ask` calls Gemini, another calls OpenAI, and `generateChecked` doesn't care which.

Then change your Milestone 1 drafter to use it, and check that `pnpm score-posts` gives the same results as before. That's a **refactor**: same behaviour, better structure, and your score table proves nothing broke.

### The classifier

| ID | Requirement |
|---|---|
| C1 | A Zod schema: `category` is one of `recruiter`, `interview`, `course`, `newsletter`, `other`; `confidence` is a number from 0 to 1; `summary` is a string of at most 200 characters. The answer has **no** `id` or `message_id` field: your code already knows which email it's classifying. |
| C2 | The prompt follows the six prompt rules in [P3's requirements, §6 Step 1](https://github.com/SoftServedWeb-com/n8n-projects-norlabs/blob/main/projects/p3-inbox-triage/REQUIREMENTS.md). The email goes inside `<email>…</email>` tags, marked as data, not instructions. The body is cut to 2,000 characters. |
| C3 | It uses `generateChecked()`. If both attempts fail, the result is `null`, and the email goes to review with the reason `couldn't classify`. |
| C4 | Your code decides the label, in a function `routeLabel(result)`: `recruiter` or `interview` with confidence ≥ 0.7 → `AI/Recruiter`; `course` or `newsletter` with confidence ≥ 0.7 → that label; anything else, including `null` → `AI/Review`. |
| C5 | `pnpm score-emails` runs the 10 samples in `data/emails.json` and prints `id`, `expected`, `predicted`, `confidence`, `label`, `✓/✗`, then the accuracy. |
| C6 | Unit tests for `routeLabel`, with no API call: confidence 0.69 vs 0.7, an `other` email, and `null`. |

**Done when:** at least **9 of 10** are correct, samples 9 (job scam) and 10 (prompt injection) get `AI/Review`, and `pnpm test` passes.

You can't unit-test a prompt, but you can unit-test `routeLabel`. That's why the 0.7 threshold lives in code, not in the prompt.

---

## Part B · The agent loop (three evenings)

### The idea

```
 messages ─▶ model ─▶ tool calls? ── no ──▶ final answer, stop
                          │ yes
                          ▼
              for each call: find the tool → parse the arguments → Zod check → run it
                          │                         (errors become results too)
                          ▼
                 add the results to messages ─▶ next step ─▶ (stop at 8 steps)
```

Four things to understand before you write code:

1. **The model never runs anything.** It writes a request: "call `classify_email` with `{"email_id": "sample-03"}`". Your code decides whether and how to run it. It's the same wall as Milestone 1, now for tool arguments.
2. **Errors are results.** If a tool fails, tell the model in plain words and keep going. A model that reads "GitHub returned 403" can explain the problem to the user. A crashed program can't.
3. **Limits make it safe.** A maximum number of steps, ranges on every argument (`days` from 1 to 7), and **no dangerous tools**.
4. **Each part sees only what it needs.** `list_emails` returns id, sender, subject and date, never the body. `classify_email` returns only the three checked fields. So the email bodies, including sample 10's "ignore your instructions…", only ever reach the classifier, and the classifier has no tools. The injection has nothing to work with. This is called **least privilege**, and it's the most important design idea this month.

**Write the loop yourself.** Use your model's SDK only: no LangChain, no Vercel AI SDK, no `runTools` helper. Libraries hide this loop, and you need to have seen it once so you can debug it when a library does something strange. In Milestone 4 you may use a library.

Start from the Week 5 Day 3 lab (`tool-agent.ts`, the weather and note tools), if you did it. Look at how it handles arguments: `JSON.parse(call.function.arguments)` with no `try`, and `args: any`. What happens if the model sends `{"days": "seven"}`? Your version fixes that.

### The tools

| Tool | What it does | Arguments (Zod) |
|---|---|---|
| `list_emails` | Lists the sample emails: id, sender, subject, date. **Never the body** | `{ limit?: number }`, an integer from 1 to 20 |
| `classify_email` | Runs your Part A classifier on one email, and adds the label from `routeLabel()` | `{ email_id: string }` |
| `get_recent_commits` | Your GitHub commit messages from the last N days, from the repos listed in `.env` | `{ days: number }`, an integer from 1 to 7 |
| `get_previous_posts` | Your last N LinkedIn posts, from `data/posts.json` | `{ limit: number }`, an integer from 1 to 10 |
| `draft_post` | Builds a post with your Milestone 1 `buildPost()` and saves it as a **draft** in `data/drafts.json`. It does not publish | Your Milestone 1 `PostSchema`, plus `hashtags` (only from your allowed list) |

`draft_post` reuses Milestone 1 completely: the model fills `intro`, `bullets` and `takeaway`, your `PostSchema` checks them, and `buildPost()` builds the text. The rules you wrote in Week 5 now protect the agent too.

For `data/posts.json`: copy 3–5 of your real, already-public LinkedIn posts into the format of `data/posts.example.json`. No email addresses or phone numbers.

For `get_recent_commits`: use GitHub's REST API, `GET /repos/{owner}/{repo}/commits?author={user}&since={ISO date}`. Put `GITHUB_USER` and `GITHUB_REPOS` (comma-separated) in `.env`. Without a token, GitHub allows 60 requests an hour; that's enough. If you add a token, make it fine-grained and read-only.

### Requirements

| ID | Requirement |
|---|---|
| A1 | Each tool's arguments are checked with its Zod schema **before** the tool runs. Bad arguments go back to the model as an error message it can read, not as a crash. |
| A2 | The loop stops after at most **8 steps**. If it hits the limit, it prints "stopped: step limit" and what it has so far. |
| A3 | If a tool fails (GitHub down, a missing file), the error goes back to the model, and the loop keeps going. |
| A4 | Every step is logged: step number, tool name, arguments, and a short version of the result. Run with `--verbose` to see it. |
| A5 | Email IDs come from your data. If the model asks for an ID that doesn't exist, the tool says so, and suggests calling `list_emails`. It never guesses. |
| A6 | CLI: `pnpm agent "your question"`, with `--verbose` and `--max-steps <n>` options. |

### Suggested files

| File | What goes in it |
|---|---|
| `src/agent/tools.ts` | The five tools. Each one is `{ name, description, schema, run }` |
| `src/agent/execute.ts` | `execute(name, rawArgs): Promise<string>`: finds the tool, parses and checks the arguments, runs it, and turns **every** failure into an error string. Keep it in its own file, with no code that runs on import: Milestone 3 reuses it |
| `src/agent/loop.ts` | `runAgent(question, options)`: the loop |
| `src/agent/cli.ts` | Reads the command-line arguments and calls `runAgent` |
| `src/agent/execute.test.ts` | Tests for `execute()` with bad arguments, with no API call |

### Steps

1. **A tool type.** Define `type Tool = { name; description; schema; run }`, where `run` receives `z.infer<typeof schema>`. Keep the tools in an array and also in a `Map` by name. (This week's Day 4 topic, maps and sets, fits here.)
2. **The tool list for the model.** With OpenAI, each tool becomes `{ type: "function", function: { name, description, parameters } }`. Generate `parameters` from your Zod schema with `z.toJSONSchema(schema)`, so you write each schema once.
3. **`execute()`.** Four ways it can fail, and each returns an error string, never a throw: unknown tool, arguments not valid JSON, arguments fail the Zod check, the tool itself throws.
4. **Two tools first.** Get `list_emails` and `classify_email` working with the loop. Ask "anything from recruiters?" with `--verbose`, and read the log line by line.
5. **The loop.** Call the model with `messages` and `tools`. If the reply has no tool calls, that's the answer. Otherwise add the reply to `messages`, run each call through `execute()`, and add each result as `{ role: "tool", tool_call_id, content }`. Skip any call whose `type` isn't `"function"`.
6. **The other three tools.** Then the step limit, and the tests.

**Using Gemini?** Gemini has tool calling too (`config.tools` with `functionDeclarations`, and `response.functionCalls`). The loop is the same idea. The Week 5 Day 3 lab uses OpenAI, so OpenAI is the easier path here; it's fine to use OpenAI for the agent and keep Gemini in your poster.

### Hints

- The system prompt should say what the agent is for, that it must use tools to get facts and never guess IDs or commits, and that it can save drafts but can't send or publish.
- Write good tool descriptions: what the tool does, when to use it, what it returns. The model chooses tools mainly by reading them.
- Your agent drafts a post without checking commits first? Improve the descriptions and the system prompt before adding code.
- If the model sends three tool calls in one reply, run all three before calling the model again. Decide whether your step limit counts model calls or tool calls, and be ready to say why.

### Done when

- [ ] Part A's "Done when" passes.
- [ ] "Anything from recruiters?" calls `list_emails`, then `classify_email` for the right ones, and answers with the senders of samples 1–4. It doesn't treat sample 10's text as an instruction.
- [ ] "Write today's post from my commits, and don't repeat what I posted recently" calls `get_recent_commits` and `get_previous_posts` **before** `draft_post`, and a draft built by `buildPost()` appears in `data/drafts.json`.
- [ ] `--max-steps 2` on that same question prints "stopped: step limit", and nothing crashes.
- [ ] With a repo name that doesn't exist in `GITHUB_REPOS`, the answer mentions the GitHub error, and nothing crashes.
- [ ] Asking to classify `sample-99` gets "no email with that id".
- [ ] `pnpm test` passes, including `execute("get_recent_commits", '{"days": 30}')` returning an error string.
- [ ] No tool's name or code sends, publishes or deletes anything.

### Stretch

- Add a `--budget` option that stops when the total tokens used pass a limit.
- Ask a question that needs two tools that don't depend on each other. Does the model ask for both in one reply?

---

## Hand in

Push to your `career-copilot` repo. Add to the README: the classifier's score table, a diagram of your loop (a photo of a paper sketch is fine), and a `--verbose` log of the post question.

**Next:** [Milestone 3 · The MCP server](<milestone-3-mcp-server.md>).
