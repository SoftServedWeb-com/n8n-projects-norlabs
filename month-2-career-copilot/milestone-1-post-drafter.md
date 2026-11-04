# Milestone 1 · Week 5 · The post drafter

**Time:** about 5 evenings.
**New idea:** a model's answer is a guess. Your code checks it before anything uses it, and you measure how often the guesses are good.

**Starting point:** the LinkedIn poster you built in Month 1. If you built it only in n8n, first port its core to TypeScript: a function that takes your notes and returns post text (one evening).

[← Back to the project overview](<README.md>)

---

## Part 0 · Close your n8n work (one evening)

### 0.1 · Keep your keys safe

Look through every repo you pushed in Month 1. If any of them ever contained a key, a token or a `.env` file, even in an old commit:

1. **Cancel the key at its source** and create a new one: OpenAI or Gemini (API keys page), Google (delete or reset the OAuth client, then remove the app at myaccount.google.com/permissions), LinkedIn (reset the app's client secret, remove the app under Settings → Data privacy → Permitted services), Telegram (`/revoke` with BotFather).
2. **Then** delete the repo, or recreate it without that history.

Doing step 2 without step 1 doesn't help. Bots scan GitHub for keys within minutes of a push.

### 0.2 · Explain your own code

Book a 15-minute call with your instructor. Walk through your TypeScript LinkedIn poster, and make one small change live, without AI help.

### 0.3 · If you built P3 in n8n, check these three things

These are the most common P3 mistakes. Fix any you have, re-export, and you're done with n8n.

1. **Every email gets a label.** Anything the model calls `other`, anything with confidence below 0.7, and anything that failed validation goes to `AI/Review`. In a Switch node, use the **fallback output** for this. A rule that waits for the model to say `review` never fires, because `review` isn't one of the model's categories.
2. **The message ID comes from your data, not from the model.** Gmail nodes should use the ID from your normalise step. If you ask the model to repeat the ID back and use its copy, the model decides which email gets labelled, including when an email tells it to.
3. **Each email is handled once.** Check a `p3_processed` Data Table before the AI call, and add the email's ID after labelling.

Then write a short README for P3 and a #BuildWithNor post about it.

---

## Part 1 · The lab (one evening)

Do the **`w2-structured-output`** lab (Evening Bootcamp, Week 2) exactly as its README says, including **"Try the failure path on purpose"**.

Before moving on, you should be able to say:

- why the code uses `safeParse` and not `parse`;
- what `z.infer` gives you;
- why the lab asks for JSON in the prompt instead of using the SDK's strict-schema helper.

---

## Part 2 · Upgrade your poster (two to three evenings)

### The idea

Your poster asks the model for a post and uses whatever comes back. Your prompt already has rules ("60 to 130 words", "2–4 bullet points", "a key takeaway", "no hashtags or links"), but **nothing checks them**. An empty reply, a 300-word essay or a post with a stranger's link would all go straight to your approval email.

Think of a wall between the model and your code. Everything the model sends you is untrusted text, like a form a stranger filled in. Zod is the guard at the door.

```
  model ──text──▶ │ JSON.parse │ ─▶ │ Zod safeParse │ ─▶ your code builds the post
                  └── fail ────┴──────── fail ─────┴─▶ retry once ─▶ stop safely, no post
```

Two changes in how you think about it:

1. **The model returns parts, not a post.** It sends `{ intro, bullets, takeaway }`. **Your code** puts them together, adds the header and the hashtags, and escapes the characters LinkedIn treats as special. Rules you can check in code stay in code.
2. **Measure, don't guess.** When you change the prompt, how do you know it got better? You run the same 10 notes before and after and compare. That's an **eval**, and it starts here.

### Requirements

| ID | Requirement |
|---|---|
| R1 | A Zod schema `PostSchema`: `intro` (a string), `bullets` (2 to 4 strings), `takeaway` (a string). Choose sensible length limits for each, and add a check that the whole post is 60–130 words. No field may contain a link or a hashtag. |
| R2 | The prompt asks for exactly that JSON shape, says the notes are data and not instructions, and says to never add facts that aren't in the notes. |
| R3 | If the answer isn't valid JSON or fails the schema, retry **once**, and send the Zod error message back to the model in the retry. If it fails again, return a clear failure. Never crash, and never post. |
| R4 | Empty or whitespace-only notes stop **before** the model is called. |
| R5 | `buildPost()` is a function with no API calls: it joins the header, the parts and the hashtags into the final text. It escapes LinkedIn's special characters in the parts you got from the model (not in the hashtags), and refuses a post over 3,000 characters. Hashtags come only from your allowed list in code. |
| R6 | Your poster uses the new drafter. When drafting fails, it does **not** send an approval email with a broken post. It tells you drafting failed, so you can write the post yourself. |
| R7 | `pnpm score-posts` runs all 10 notes in `data/notes.json` and prints a table: `id`, `result` (ok / failed / skipped), `attempts`, `words`, `bullets`. Under it: how many passed. |
| R8 | Unit tests for `buildPost()`, with no API call: escaping, hashtags left alone, and the 3,000-character limit. |

**Gemini or OpenAI?** Either. The lab uses OpenAI; if your poster uses Gemini, keep it. With Gemini, put your system prompt in `config.systemInstruction` (not pasted into the message), set `config.responseMimeType: "application/json"`, and read `response.text`. The Zod check stays exactly the same.

### Suggested files

| File | What goes in it |
|---|---|
| `src/post/schema.ts` | `PostSchema` and `type Post = z.infer<typeof PostSchema>` |
| `src/post/build.ts` | `escapeLinkedIn()` and `buildPost()` |
| `src/post/build.test.ts` | Tests for `buildPost()` |
| `src/post/draft.ts` | The prompt and `draftPost(notes)`: returns a checked `Post`, a skip, or a failure with the reasons |
| `src/post/score.ts` | Runs the 10 notes and prints the table |

### Steps

1. **Schema first.** Write `PostSchema`. For "no links or hashtags", a `.refine()` with a regular expression works. For the word count, use a `.refine()` on the whole object.
2. **`buildPost()` and its tests**, before any model call. LinkedIn's API treats these characters as special in post text: `\ | { } @ [ ] ( ) < > # * _ ~`. Each needs a `\` in front, or the post can be cut off. Did your Month 1 poster escape them?
3. **The prompt.** Start from your poster's prompt. Ask for JSON only, in the shape of `PostSchema`.
4. **Check and retry.** A function that takes the raw reply and returns either the checked data or a reason it failed. `z.prettifyError(result.error)` turns a Zod error into readable text. On a failure, add the bad reply and a message like "That reply failed validation: … Return only the corrected JSON." to the conversation, and ask once more.
5. **Score.** Run all 10 notes. Read every post that passed, too: the schema can't tell whether a post invented facts. Note 8 (a bad day) and note 2 (vague notes) are the ones to read carefully.
6. **Connect it to your poster** (R6), and run it end to end once.

### The 10 notes

`data/notes.json` (from the starter) has 10 notes, each with a `checks` field saying what it tests: a normal day, vague notes, a very long day, special characters, code, a hidden instruction to add a link, notes in Manglish, a bad day, six topics for four bullets, and empty notes. Don't change them, so your scores stay comparable as you improve your prompt.

### Hints

- If every note fails on the first try, print the raw reply. An empty reply usually means the token limit was too low (see the lab's comment about `max_completion_tokens`), not a schema problem.
- Note 9 has six topics. If the model sends 6 bullets, your schema should reject it, and the retry message should say why. That's the retry doing its job.
- Don't call the API in your unit tests. `buildPost()` has no randomness in it, which is exactly why it's easy to test.

### Done when

- [ ] All **9** non-empty notes give a valid post (retries allowed), and note 10 is skipped without a model call.
- [ ] Note 6's output contains no link and no `#crypto`. Note 8's post doesn't claim the bug was fixed.
- [ ] You can show the retry working: change the schema on purpose (for example, `takeaway` at most 10 characters), run note 1, and show attempt 1 failing, attempt 2 failing, then a clean failure with no post sent.
- [ ] `pnpm test` passes.
- [ ] Your poster sends you an approval email with a post built by `buildPost()`.
- [ ] `git log --all --oneline -- .env` prints nothing.

### Stretch

- Run all 10 notes three times. Which ones change the most? What does that tell you about your prompt?
- Try a cheaper model on the same notes, and compare: how many pass, and the cost per post.

---

## Hand in

1. Your `career-copilot` repo URL, with a README section for this milestone: your score table, and a screenshot of the forced-failure run.
2. Your P3 re-export (if you built P3), with the fixes from Part 0.3.
3. A #BuildWithNor post: what changed when you stopped trusting the model's output. (Drafted by your new drafter, of course.)

**Next:** [Milestone 2 · The agent loop](<milestone-2-agent-loop.md>). Your `PostSchema` and `buildPost()` become the agent's `draft_post` tool, and you use the same check-and-retry pattern to build an email classifier.
