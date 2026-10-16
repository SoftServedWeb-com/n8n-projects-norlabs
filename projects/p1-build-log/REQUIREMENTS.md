# P1 · Build-log → LinkedIn with Approval — Requirements

| | |
|---|---|
| **Category** | Build in public |
| **Difficulty** | ★★☆ Intermediate |
| **Time** | 6–8 hours across 3 milestones |
| **Ready after** | Week 2 Day 3 (LLM nodes & branching) |
| **Runs on a laptop** | ✅ Yes. Approve from Telegram Desktop on the same laptop (see §14) |
| **Reference solution** | Planned: `reference/workflow.json` |
| **Status** | Requirements v1 · node names checked against n8n 2.43.7 on 2026-10-16 · workflow not yet built |

---

## 1. Problem

You build something almost every day, but your LinkedIn doesn't show it. Either you skip posting because writing takes time, or an auto-poster publishes generic AI text that could have come from anyone. Recruiters skim past generic posts. A post that names the real thing you built today, in your own voice, is the one that gets read.

This project turns your real work, your GitHub commits, into a draft post in your voice. You approve, edit or skip it from Telegram, and only then does it go out. If you already have an auto-poster, this project becomes its input. It doesn't replace it.

## 2. Who it's for & how they use it

**User:** you, a student in this program who commits code most days and posts on LinkedIn.

**User story:** *As a student building in public, I want a draft post written from what I actually committed, so that I post consistently without posting filler.*

**A day with it running:**
1. You commit lab work during the day as usual.
2. In the evening (8 pm, or the first hour your laptop is on after that, until 11 pm) a Telegram message arrives: a 150-word draft that mentions the commits from today's Week 2 lab, written the way you write.
3. You tap **Approve**, or fix one sentence and send it, or tap **Skip**.
4. If you approved, the post publishes through your existing poster, or comes back to you as clean text to paste.
5. On a day with no commits, you get one line saying "no commits since yesterday, no draft today". Nothing is posted.

## 3. Scope

### In scope
- Commits from repositories you own, public and private, on their default branches.
- One draft per run, in your voice, based on example posts you provide.
- Telegram approval with four outcomes: approve, approve with edits, skip (keep for tomorrow), discard (never post these commits).
- Handing the approved text to a publisher: your existing poster, or back to Telegram for manual pasting.
- A log of every draft and decision.

### Out of scope
- Posting without approval, under any setting.
- Images, carousels or video.
- Posting to X/Twitter (see [b06 · Post repurposer](<../b06-post-repurposer/REQUIREMENTS.md>)).
- LinkedIn analytics or engagement tracking.
- Commits on non-default branches, and repositories you don't own (forks, org repos), in v1–v3.

## 4. Functional requirements

| ID | Requirement |
|---|---|
| FR-1 | The workflow is triggered every hour from 20:00 to 23:00 IST, but acts **at most once per calendar day**: it stores `last_run_date` in `p1_state` and exits immediately if today's date is already there. n8n doesn't replay a schedule missed while the laptop slept, so a laptop opened at 21:30 still gets that evening's draft. It can also be run manually for testing; a manual run ignores `last_run_date`. |
| FR-2 | It reads a stored marker, `last_included_commit_at`, the timestamp of the newest commit already handled. On first run, with no marker, it uses "24 hours ago". |
| FR-3 | It fetches commits you authored after the marker, across every repository you own that was pushed after the marker. The look-back window is capped at 7 days, and at most 50 commits are used (newest first). |
| FR-4 | If there are zero commits, it sends one Telegram message ("No commits since `<date>`, no draft today") and stops. The marker does not move. |
| FR-5 | For each commit it keeps only: repository name, whether the repo is private, commit message (first line) and timestamp. It never sends code or diffs to the model. |
| FR-6 | A **private** repository's name is replaced with a generic phrase ("a private project") in the model's input, unless that repo is on your allowlist (`public_names` table, §5). |
| FR-7 | The AI step produces one draft as JSON matching the schema in §6. |
| FR-8 | If the AI output is not valid JSON or fails the schema, the workflow retries once. If it fails again, it sends Telegram "Draft failed" with the raw output and stops without moving the marker. |
| FR-9 | The draft is sent to your Telegram chat with an approval step offering: **Approve**, **Approve with edits** (a text field pre-filled with the draft), **Skip**, **Discard**. |
| FR-10 | If you don't respond within 12 hours, the run is treated as **Skip** and you get a message saying so. |
| FR-11 | **Approve / Approve with edits:** the final text (yours if edited) is handed to the publisher (§9). On success, the marker moves to the timestamp of the newest commit *included in the draft*. Commits made while the draft waited are therefore picked up tomorrow. |
| FR-12 | **Skip:** nothing is posted and the marker does not move, so the same commits roll into tomorrow's draft. |
| FR-13 | **Discard:** nothing is posted and the marker moves past these commits. They are never drafted again. |
| FR-14 | If the publisher fails, you get a Telegram message with the final text, so you can paste it yourself. The marker does not move. |
| FR-15 | Every run that reaches the AI step appends one row to `post_log` (§5), whatever the outcome, including `draft_failed` and `publish_failed`. |
| FR-16 | Any unhandled error anywhere in the workflow triggers a separate error workflow that sends you a Telegram alert naming the failed node. |
| FR-17 | The bot responds only to your own Telegram chat ID. Messages from any other chat are ignored. |

## 5. Data

**Inputs**
- GitHub: the repositories you own (name, private flag, `pushed_at`) and their commits (message, author, timestamp).
- Your voice examples: 3–5 of your own past LinkedIn posts, pasted into the `voice_examples` sheet.

**Storage**

| Store | Where | Columns / keys | Purpose |
|---|---|---|---|
| `p1_state` | n8n Data Table (or a Google Sheet tab) | `key`, `value` with rows `last_included_commit_at` (the marker) and `last_run_date` (the once-a-day guard, FR-1) | Survives restarts and laptop sleep |
| `voice_examples` | Google Sheet | `id`, `post_text` | Teaches the model your voice |
| `public_names` | Google Sheet | `repo_name` | Private repos whose names may appear in posts |
| `post_log` | Google Sheet | `run_at`, `commit_count`, `repos`, `draft`, `final_text`, `decision`, `published`, `execution_id` | Audit trail; also good source material for your Capstone post. `decision` is always exactly one of `approve`, `approve_edit`, `skip`, `discard`, `timeout`, `draft_failed`, `publish_failed`. Other projects (e.g. b06) read these values, so don't invent new ones |

**Outputs**
- A Telegram approval message.
- Approved text handed to the publisher.
- One `post_log` row per draft.

## 6. AI steps

**Step: draft the post**

| | |
|---|---|
| Model | Default tier (`gpt-5-mini`, per [CONVENTIONS.md](<../../CONVENTIONS.md>)), low reasoning effort |
| Input | The commit list (after FR-5 and FR-6), today's date, your voice examples |
| Output | JSON, schema below |

```json
{
  "hook": "string — first line, under 120 characters, no hashtags",
  "body": "string — 100 to 200 words, first person",
  "hashtags": ["3 to 5 strings, each starting with #"],
  "claims": ["each factual claim in the post, one per item"]
}
```

**Prompt rules (the prompt must say all of these):**
1. Write in the first person, matching the tone and sentence length of the voice examples. Do not copy their content.
2. Mention only work that is visible in the commit list. No invented results, metrics, users or opinions from others.
3. Every item in `claims` must be traceable to at least one commit message.
4. No emoji walls (at most 2 emoji), no "excited to announce", no "game-changer".
5. If the commits are trivial (typos, formatting), say so plainly and keep the post short. Don't inflate them.

**Fallback:** see FR-8. An invalid draft is never shown as if it were valid.

## 7. Non-functional requirements

| ID | Requirement |
|---|---|
| NFR-1 Catch-up safe | A laptop that was off for up to 7 days produces one correct draft covering the missed commits on its next run. No duplicate drafts for the same commits. |
| NFR-2 Cost | At most 2 model calls per day (1 plus 1 retry). Expected cost is well under ₹1 a day at the default tier. |
| NFR-3 Privacy | No code or diffs leave your machine. Private repo names are masked unless allowlisted (FR-6). Commit messages do go to OpenAI, so don't put secrets in commit messages (you shouldn't anyway). |
| NFR-4 Platform rules | Publishing goes only through LinkedIn's official API (your poster) or by hand. At most one post per day. No automated likes, comments, connections or messages. |
| NFR-5 Human control | No path in the workflow publishes without an explicit approval (FR-9 to FR-13). |
| NFR-6 Secrets | Exported workflow JSON contains no tokens, chat IDs or email addresses. |

## 8. Credentials & setup

All set up once in [SETUP.md](<../../SETUP.md>):

| Credential | Scope needed |
|---|---|
| GitHub fine-grained personal access token | All your repositories · **Metadata: read** · **Contents: read**. Nothing else. |
| Telegram bot token + your chat ID | Sending messages and the approval step |
| OpenAI API key | Model calls |
| Google OAuth (Sheets) | Read/write your project Sheet |
| LinkedIn | Only inside your existing poster. This project never holds a LinkedIn credential itself |

## 9. Suggested n8n building blocks

```
Schedule Trigger (hourly, 20:00–23:00 IST)
  → Data Table: get last_run_date → If already today: end   (else store today)
  → Data Table: get marker
  → HTTP Request: GET /user/repos?sort=pushed  → Filter: pushed_at > marker
  → HTTP Request: GET /repos/{owner}/{repo}/commits?author=…&since=…   (one per repo)
  → Code / Set: keep 4 fields, mask private names, cap at 50
  → If: zero commits? → Telegram "no draft today" → end
  → OpenAI (message a model, JSON output)  → Code: validate schema → If invalid: retry once → Telegram "Draft failed"
  → Telegram: Send and Wait for Response (custom form: decision + edited text)
  → Switch on decision
       Approve / Edit → Execute Workflow (your poster, input { text }) → Data Table: move marker
       Skip           → (nothing)
       Discard        → Data Table: move marker
  → Google Sheets: append post_log row
Separate workflow: Error Trigger → Telegram alert
```

| Need | n8n node (present in 2.43.7) |
|---|---|
| Daily run | Schedule Trigger |
| Marker storage | Data Table (or Google Sheets) |
| GitHub calls | HTTP Request with the GitHub credential, or the GitHub node |
| Draft | OpenAI, or Basic LLM Chain + Structured Output Parser |
| Approval | Telegram → *Send and Wait for Response* |
| Branching | Switch, If |
| Publisher hand-off | Execute Workflow (caller) + Execute Workflow Trigger (in your poster) |
| Log | Google Sheets |
| Alerts | Error Trigger |

**Hand-off contract with your existing poster.** Add an *Execute Workflow Trigger* to your poster that accepts one field, `text`, and publishes it. **Publish (activate) the poster workflow after every change.** Scheduled runs call the *published* version of a sub-workflow, and fail with "Workflow is not active" if it was never published; only manual test runs use your unsaved draft. Your poster may return `{ "published": true, "url": "…" }`; this project logs whatever comes back. If you have no poster, replace the Execute Workflow node with a Telegram message containing the final text.

## 10. Milestones

| Milestone | Builds | Done when |
|---|---|---|
| **v1 · Commit digest** | FR-1 to FR-5, FR-17; marker storage | In the evening run you get a plain list of today's commits in Telegram, or "no commits" on an empty day. Re-running immediately does not list the same commits again (after you move the marker manually). |
| **v2 · AI draft** | FR-6 to FR-8 | The Telegram message is now a draft post that passes the schema check, and a deliberately broken prompt triggers the "Draft failed" path. |
| **v3 · Approval & publish** | FR-9 to FR-16 | All four decisions behave as specified, the marker moves only where FR-11/FR-13 say it should, and every draft has a `post_log` row. |

## 11. Acceptance tests

| # | Scenario | Setup / input | Expected |
|---|---|---|---|
| 1 | Empty day | No commits since marker | One "no draft today" message; marker unchanged; no `post_log` row |
| 2 | Normal day | 4 commits in 2 public repos | Draft mentions only those repos' work; valid JSON; approval message arrives |
| 3 | Private repo, not allowlisted | 1 commit in private repo `client-acme` | Draft says "a private project"; the string `client-acme` appears nowhere in the draft |
| 4 | Private repo, allowlisted | Same repo listed in `public_names` | Repo name may appear |
| 5 | Big backlog | 80 commits over 3 days | Only the newest 50 are used; one draft |
| 6 | Laptop off 3 days | Marker 3 days old, commits on each day | One draft covering all 3 days on the next run |
| 7 | Broken AI output | Temporarily make the prompt ask for plain text | One retry, then "Draft failed" with raw output; marker unchanged |
| 8 | Approve | Tap Approve | Poster receives exactly the draft text; marker = newest included commit; log `decision=approve` |
| 9 | Approve with edits | Change one sentence | Poster receives *your* text, not the draft |
| 10 | Skip then next day | Skip today, commit once tomorrow | Tomorrow's draft covers both days |
| 11 | Discard | Tap Discard | Nothing posted; tomorrow's draft excludes these commits |
| 12 | Timeout | Ignore the message for 12 h | Treated as Skip; "skipped (no response)" message |
| 13 | Publisher down | Deactivate the poster | You receive the final text to paste; marker unchanged |
| 14 | Stranger messages the bot | Message from another account | Ignored; nothing runs |
| 15 | Laptop asleep at 20:00 | Open the laptop at 21:30 | One draft arrives during the 22:00 run; the 23:00 run does nothing (`last_run_date` = today) |

## 12. Stretch goals

1. **Thursday mode:** on Thursdays, draft the weekly #BuildWithNor post instead. It covers the whole week, uses the program's full tag stack and mentions, and follows the Day-4 template.
2. **Beyond commits:** include your evening check-in notes from [b05 · Evening check-in](<../b05-evening-check-in/REQUIREMENTS.md>), so non-code work counts too. Follow b05's contract (its §5):
   - add a second `p1_state` row, `last_included_checkin_stored_at`;
   - read `checkins` rows with a `stored_at` later than it, capped at 7 days and 20 rows;
   - use `tidy_bullets` when `tidy_status` is `ok`, otherwise `raw_text`;
   - move this marker on approve or discard, not on skip, exactly like the commit marker.
3. **Learn from edits:** store each "Approve with edits" pair (draft → your version) and add the last 3 to the prompt as corrections.

## 13. Deliverables

1. Exported workflow JSON (main + error workflow) with credentials removed.
2. Your Sheet's column headers as a CSV template.
3. A README: what it does, a demo GIF, setup steps, and what you learned.
4. A 60-second screen recording: commit → evening draft → approve → post live.
5. A #BuildWithNor post about the project, approved by the project itself.

## 14. Risks & gotchas

| Risk | What to do |
|---|---|
| The approval link opens an n8n page at `localhost`, which your phone can't reach | Approve from Telegram Desktop on the same laptop, or set up the optional tunnel in [SETUP.md](<../../SETUP.md>) |
| *Send and Wait for Response* has had a reported bug where it kept waiting after a reply ([n8n#13331](https://github.com/n8n-io/n8n/issues/13331)) | Test it on your version first. Fallback: send the draft with inline Approve/Skip/Discard buttons and handle the button press in a second workflow |
| Google OAuth credentials in "Testing" status expire after about 7 days | Reconnect the credential weekly, or follow the setup lab's note on publishing your personal OAuth app |
| The LinkedIn token inside your poster expires periodically | That's the poster's job to handle; FR-14 makes sure you never lose a post when it fails |
| GitHub search vs. per-repo commits | Use the per-repo commits endpoint (§9). The commit search API is less complete for private repos |
| Commits made while a draft waits for approval | Handled by FR-11: the marker is the newest *included* commit, not "now" |
| n8n doesn't replay schedules missed while the laptop slept | Handled by FR-1's hourly trigger with a once-a-day guard |

## 15. Capstone 1 fit

As built here, this project is **not** a capstone, because it's a provided project. It can seed your Capstone 1 if your spec adds something genuinely new, for example:
- a second source (merged pull requests, finished lab READMEs, or check-in notes);
- a second channel with its own approval (X, Hashnode or dev.to);
- the "learn from edits" loop (stretch 3) with a before/after evaluation.
