# Conventions

The one place this repository names a version, a model or a limit. When a project's requirements say "the model in CONVENTIONS.md", this is the file they mean. If something here changes, it changes once, here.

Last checked: **2026-10-16**.

---

## n8n

| Thing | Value | Notes |
|---|---|---|
| n8n image | `n8nio/n8n:2.43.7` | Latest stable release on 2026-10-16. Pinned in [`setup/docker-compose.yml`](<setup/docker-compose.yml>). n8n 3 is in nightly builds; stay on 2.43.x until this file says otherwise |
| Where it runs | Docker on your own laptop | See [SETUP.md](<SETUP.md>). No n8n Cloud: it has no free plan |
| Address | `http://localhost:5678` | Only your laptop can reach it (bound to `127.0.0.1`) |
| Timezone | `Asia/Kolkata` | Set in the compose file. Also set each workflow's timezone to Asia/Kolkata |
| Files folder | `/home/node/.n8n-files` inside n8n = `setup/files/` on your laptop | The only folder *Read/Write Files from Disk* may use by default |
| Execution history | Kept 7 days | Set in the compose file, so the database doesn't grow forever |

Checked on this setup on 2026-10-16:
- **Data Tables** work on self-hosted Community Edition 2.43.7. No licence needed.
- **`$env` access** works in expressions, because the compose file sets `N8N_BLOCK_ENV_ACCESS_IN_NODE=false`.
- **Local File Trigger** and **Execute Command** are disabled by default in 2.43.7. No project needs them.

## Models (OpenAI)

| Tier | Model ID | Use it for | Price (per 1M tokens, in / out) |
|---|---|---|---|
| Fast | `gpt-5-nano` | High-volume, simple steps: classify, filter, extract short fields | $0.05 / $0.40 |
| Default | `gpt-5-mini` | Every AI step unless a project says otherwise. It also accepts images, so use it for vision too (b14) | $0.25 / $2.00 |
| Transcription | `gpt-transcribe` | Voice notes and recordings (b04, b05, b10, P4 stretch), via HTTP Request to `/v1/audio/transcriptions` | $0.0045 per audio minute |
| Embeddings | `text-embedding-3-small` | b22 only | $0.02 |

At about ₹95 to the US dollar (rate on 2026-10-16), a typical project costs a few rupees a day. Each project's NFR table gives its own estimate.

**Don't use the OpenAI node's *Audio → Transcribe* operation.** In n8n 2.43.7 it always uses `whisper-1`, which OpenAI will remove on **2027-02-26**, along with the `gpt-4o-*-transcribe` models. Call the endpoint with HTTP Request, using your OpenAI credential as a *Predefined Credential Type*, and the model above.

**Reasoning budget.** `gpt-5-nano` and `gpt-5-mini` spend hidden "reasoning" tokens before they answer. Set reasoning effort to **low** for workflow steps, and don't set the max-tokens limit so low that the visible answer comes back empty.

## Telegram

| Fact | Value |
|---|---|
| Message length | 4,096 characters at most |
| Bot file downloads | 20 MB at most |
| Undelivered updates | Kept for **24 hours**, then lost. A polling bot catches up after a night off, not after a weekend |
| Rate limits | About 1 message a second per chat, about 20 a minute in a group |
| `getUpdates` and webhooks | Can't be used together on one bot. Every project that **receives** messages gets its own bot |

**Bot token naming.** Projects that receive messages read their token from `.env` as `<PROJECT>_BOT_TOKEN`, for example `B02_BOT_TOKEN` and `B05_BOT_TOKEN`. The full list is in [`setup/.env.example`](<setup/.env.example>). Sending messages uses n8n's Telegram credential as usual.

## Other services

| Service | Value | Used by |
|---|---|---|
| OpenAI audio upload limit | 25 MB per file | b04, b05, b10 |
| Qdrant (vector database) | `qdrant/qdrant:v1.19.2` (released 2026-10-05); REST and dashboard on port 6333 | b22 |
| Greenhouse jobs API | `https://boards-api.greenhouse.io/v1/boards/{token}/jobs` | P2, b02, b21 |
| Lever jobs API | `https://api.lever.co/v0/postings/{company}?mode=json` | P2, b02, b21 |
| Ashby jobs API | `https://api.ashbyhq.com/posting-api/job-board/{name}` | P2, b02, b21 |
| LinkedIn connection notes | 200 characters; 3 a month without Premium | b03 |
| Google OAuth "Testing" apps | Refresh tokens expire after about 7 days | Every project that uses Google |

## Verify before each cohort

Re-check these before relying on this file, and update the "Last checked" date:

- [ ] Latest stable n8n release (`https://github.com/n8n-io/n8n/releases`), and whether n8n 3 has shipped.
- [ ] The model IDs and prices above (OpenAI's models, pricing and deprecations pages).
- [ ] The transcription model is not on OpenAI's deprecations list.
- [ ] Qdrant's latest release.
- [ ] The three job-board endpoints still return JSON without logging in.
