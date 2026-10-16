# Setup: n8n on your laptop

Do this once, before your first project. It takes about an hour, most of it creating accounts and keys.

By the end you'll have:
- n8n running in Docker at `http://localhost:5678`, on your laptop only;
- credentials for OpenAI, Telegram, Google and GitHub, saved inside n8n;
- a `.env` file for the bot tokens that some projects read.

Every version number used here lives in [CONVENTIONS.md](<CONVENTIONS.md>).

---

## Before you start

- A laptop with **8 GB RAM or more**, and about 5 GB of free disk space.
- Admin rights to install software.
- A Google account, a Telegram account, a GitHub account, and an OpenAI account with billing set up.

---

## Step 1: Install Docker Desktop

Download it from [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/) and install it.

- **Windows:** the installer asks to use **WSL 2**. Say yes. If it says WSL isn't installed, open PowerShell as administrator, run `wsl --install`, restart, and run the installer again.
- **macOS:** pick the Apple-silicon or Intel download to match your Mac.
- **Linux:** install Docker Engine and the Compose plugin with your distribution's packages.

**Check:** open a terminal and run

```bash
docker --version
docker compose version
```

Both should print a version. Docker Desktop must be running (whale icon in the tray or menu bar) whenever you use n8n.

---

## Step 2: Get this repository

```bash
git clone https://github.com/SoftServedWeb-com/n8n-projects-norlabs.git
cd n8n-projects-norlabs/setup
```

No Git yet? Use **Code → Download ZIP** on GitHub, unzip it, and open a terminal in the `setup` folder.

---

## Step 3: Create your `.env` file

```bash
# macOS / Linux / Git Bash
cp .env.example .env
```

```powershell
# Windows PowerShell
Copy-Item .env.example .env
```

Leave it as it is for now. You'll add bot tokens in Step 6, one project at a time.

> **`.env` holds secrets. Never commit it, screenshot it, or paste it into a chat.** If you copy this repository's files into your own repo, check that `.env` is in your `.gitignore`.

---

## Step 4: Start n8n

From the `setup` folder:

```bash
docker compose up -d
```

The first start downloads n8n (about 1 GB) and takes a few minutes. Then open **http://localhost:5678**.

1. n8n asks you to create an **owner account**. This account exists only on your laptop. Use your email and a password you'll remember.
2. Skip the survey and any paid-plan offers. Everything here works on the free Community Edition.

**Check:** you see the n8n canvas. In the terminal, `docker ps` shows a container named `norlabs-n8n`.

**Every day after this:**

| To | Run (in `setup/`) |
|---|---|
| Start n8n | `docker compose up -d` |
| Stop n8n | `docker compose down` (your workflows are kept) |
| See what n8n is doing | `docker compose logs -f` (Ctrl+C to leave) |

n8n only runs while your laptop is on and Docker is running. Every project is designed for that: when your laptop wakes up, it catches up on what it missed.

---

## Step 5: Add your credentials in n8n

In n8n, go to **Credentials → Add credential**. Do the four below now. Each project's requirements (section 8) list exactly which ones it uses and with what permissions.

### OpenAI

1. Go to [platform.openai.com](https://platform.openai.com/) → **API keys** → **Create new secret key**. Copy it.
2. Make sure billing is set up. A few hundred rupees of credit is plenty for all the projects.
3. In n8n: **Add credential → OpenAI**. Paste the key, then click **Save**. n8n tests it for you.

### Telegram (one bot to start)

1. In Telegram, open **@BotFather**, send `/newbot`, and follow the prompts. It gives you a **token** like `123456789:AA...`.
2. Open a chat with your new bot and send it any message, such as "hi".
3. **Find your chat ID.** In your browser, open `https://api.telegram.org/bot<TOKEN>/getUpdates`, with your token in place of `<TOKEN>`. In the reply, find `"chat":{"id":` and copy the number after it. That's your chat ID. Projects use it so the bot only talks to you.
4. In n8n: **Add credential → Telegram API**. Paste the token and save.

This bot is for projects that **send** you messages. Projects that also **receive** messages each need their own bot (Step 6).

### Google (Sheets, Gmail, Drive, Calendar, Docs)

Your own n8n needs your own Google "OAuth app". You set it up once, and every Google credential uses it.

1. Go to [console.cloud.google.com](https://console.cloud.google.com/) and create a project, for example `n8n-norlabs`.
2. **APIs & Services → Library.** Enable the APIs your projects use: **Google Sheets API**, **Gmail API**, **Google Drive API**, **Google Calendar API**, **Google Docs API**.
3. **APIs & Services → OAuth consent screen.** Choose **External**. Fill in the app name and your email. Under **Test users**, add your own Gmail address.
4. In n8n: **Add credential**, choose for example **Google Sheets OAuth2 API**. Copy the **OAuth Redirect URL** it shows. It looks like `http://localhost:5678/rest/oauth2-credential/callback`.
5. Back in Google Cloud: **APIs & Services → Credentials → Create credentials → OAuth client ID** → **Web application**. Under **Authorized redirect URIs**, paste the URL from step 4. Create it, then copy the **Client ID** and **Client secret**.
6. In n8n, paste the Client ID and Client secret, click **Sign in with Google**, choose your account, and allow access. Google warns that the app isn't verified: click **Continue**. It's your own app.
7. Repeat steps 4 and 6 for each Google service you need (Gmail OAuth2, Google Drive OAuth2, and so on). They reuse the same Client ID and secret.

> **Credentials stop working after about 7 days.** Google expires refresh tokens for apps in "Testing" status. When a Google node starts failing, open the credential and click **Sign in with Google** again. Some projects ask for narrower permissions (scopes) than n8n's default; their section 8 says which.

### GitHub

1. GitHub → **Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token**.
2. **Repository access:** only the repos the project needs, or "All repositories" for P1 and b07.
3. **Permissions:** read-only. Usually **Metadata: Read** and **Contents: Read**; the project's section 8 lists them.
4. Set an expiry date. Fine-grained tokens expire, and you make a new one when this one does.
5. In n8n: **Add credential → GitHub API**, and paste the token as the access token.

---

## Step 6: Bot tokens for projects that receive messages

Projects marked 📨 in the [project list](<README.md#all-projects>) read your Telegram messages by polling. Your laptop has no public address, so it asks Telegram every minute. That needs two things:

1. **A new bot for each 📨 project.** Two workflows polling the same bot steal each other's messages. Create it with @BotFather, as in Step 5.
2. **Its token in `.env`**, under the name the project uses, for example:

   ```
   P4_BOT_TOKEN=123456789:AA...
   ```

   The project's workflow reads it as `{{ $env.P4_BOT_TOKEN }}`, so the token is never typed into a workflow and never ends up in an export. Also save the same token as a Telegram credential in n8n, for sending replies.

After any change to `.env`, restart n8n so it picks the change up:

```bash
docker compose up -d --force-recreate
```

---

## Exporting your work safely

To put a finished workflow in your own GitHub repo:

1. In the workflow, open the **⋯** menu → **Download**. You get a `.json` file.
2. Open it in a text editor and search for your bot tokens, your chat ID, email addresses and phone numbers. None should be there. Credentials themselves are never exported, only their names. Projects that follow their requirements keep tokens in `.env`, so the export is clean.
3. Commit the `.json` to **your own** repository, with a README, as each project's section 13 describes.

---

## Optional: a public address (tunnel)

You don't need one for any project's default path. Some stretch goals do: instant Telegram replies through the Telegram Trigger, or approving from your phone. A tunnel gives your laptop's n8n a temporary public HTTPS address.

1. Install `cloudflared` from [developers.cloudflare.com](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/).
2. Run `cloudflared tunnel --url http://localhost:5678`. It prints an address like `https://something-random.trycloudflare.com`.
3. In `.env`, set `WEBHOOK_URL=` to that address, with a `/` at the end, and restart n8n with `docker compose up -d --force-recreate`.
4. Re-activate any workflow with a webhook or Telegram Trigger, so it registers the new address.

The address changes every time you restart the tunnel. While it's running, **anyone with the address can reach your webhooks**, so stop it when you're done (Ctrl+C), remove `WEBHOOK_URL` from `.env`, and restart n8n. Never send real customer data through it.

---

## Updating n8n

Only update when [CONVENTIONS.md](<CONVENTIONS.md>) changes the version.

1. Download every workflow first (see "Exporting your work safely").
2. Change the image tag in `setup/docker-compose.yml`.
3. Run `docker compose pull`, then `docker compose up -d`.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `port 5678 is already allocated` | Another n8n is running. Run `docker ps`, then `docker stop <name>` for the old one, or restart Docker Desktop |
| `Cannot connect to the Docker daemon` / `docker: command not found` | Start Docker Desktop and wait until it says it's running |
| `env file .env not found` | You skipped Step 3. Create `.env` in the `setup` folder |
| `{{ $env.X_BOT_TOKEN }}` comes out empty, or the URL reads `.../bot/getUpdates` | The name in `.env` doesn't match exactly, or you didn't restart with `--force-recreate` |
| Telegram `getUpdates` returns `409 Conflict` | The bot has a webhook set, usually from a Telegram Trigger. Deactivate that workflow, open `https://api.telegram.org/bot<TOKEN>/deleteWebhook` once, and keep one bot per project |
| Google: "Access blocked" or "app has not completed verification" | Add your own Gmail address under **Test users** on the OAuth consent screen |
| Google: `redirect_uri_mismatch` | The redirect URI in Google Cloud must match n8n's OAuth Redirect URL exactly, including `http://` and the port |
| A Google node suddenly fails with an auth error | The 7-day expiry. Open the credential and sign in again |
| Scheduled workflows run at the wrong time | Set the workflow's timezone (**Settings → Timezone**) to Asia/Kolkata |
| "Workflow is not active" when one workflow calls another | Publish (activate) the called workflow. Scheduled runs use the published version |
| n8n feels slow or the laptop runs out of memory | Close other heavy apps. In Docker Desktop → Settings → Resources, give Docker at least 4 GB |
