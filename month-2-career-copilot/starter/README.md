# Career Copilot · starter

Copy everything in this folder into your own `career-copilot` repo, **including the hidden `.gitignore`**, then follow [Getting started](../README.md#getting-started) in the project overview.

| File | What it's for |
|---|---|
| `.gitignore` | Keeps `.env`, `node_modules`, `dist` and runtime files out of git. It must be in your **first** commit |
| `.env.example` | The settings each milestone needs. Copy it to `.env` and fill it in |
| `package.json` | Scripts for every milestone, and the packages for Milestone 1 (OpenAI and Zod; if your poster uses Gemini, also run `pnpm add @google/genai`). Later milestones tell you what to add |
| `tsconfig.json` | Strict TypeScript, Node-style modules, builds `src/` into `dist/` |
| `data/notes.json` | 10 fixed "what I learned" notes for scoring your post drafter (Milestone 1). Don't edit them, so your scores stay comparable |
| `data/posts.example.json` | The format for `data/posts.json` (Milestone 2). Replace the examples with your own public posts |

**Imports end in `.js`.** With these settings, a file importing `src/post/build.ts` from the same folder writes `import { buildPost } from "./build.js";`. That looks odd, but it's how TypeScript and Node agree on file names: `tsx` finds the `.ts` file while you develop, and the built `dist/` files point at the `.js` files.

The scripts expect files you'll create in each milestone (`src/post/score.ts`, `src/agent/cli.ts`, …). A script fails until its file exists. That's expected.
