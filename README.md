# Claude Code for Free — Setup Guide

Use Claude Code CLI with free Gemini models via a local routing proxy.

---

## Before you start

Claude Code was designed to work with Anthropic's own models. It sends a very large system prompt on startup, and free models like Gemini may struggle with it — responses can be slow, incomplete, or off.

**This guide is for getting a feel for the Claude Code CLI experience, not for serious work.**

If you want the real thing:
- Get a [Claude.ai](https://claude.ai) account and use Anthropic's models directly — they are significantly better for coding tasks.
- If Claude.ai is not available to you, consider pairing Claude Code with a paid model from another provider: ChatGPT (OpenAI), Gemini (Google), DeepSeek, or others — the router supports custom providers.

---

## Requirements

- [Node.js](https://nodejs.org/) installed
- A free Gemini API key from [Google AI Studio](https://aistudio.google.com/apikey)

---

## Step 1 — Install the packages

```bash
npm i @anthropic-ai/claude-code -g
npm i @musistudio/claude-code-router -g
```

> **Note:** After installation you can use the shorthand commands `claude` and `ccr`.
> If shorthands are not available in your terminal, use the full forms:
> `npx @anthropic-ai/claude-code` and `npx @musistudio/claude-code-router`.

> **Windows:** If `claude` or `ccr` are not recognized right after install, restart your terminal — npm global binaries don't always appear in PATH until you do.

---

## Step 2 — Configure Claude Code in your project

Create the file `.claude/settings.local.json` in one of two places:

- **Per project:** `.claude/settings.local.json` inside your project folder — only affects that project.
- **Global:** `%USERPROFILE%\.claude\settings.local.json` on Windows, `~/.claude/settings.local.json` on Linux/macOS — applies to all projects, so you only need to do this once.

```json
{
  "hasCompletedOnboarding": true,
  "env": {
    "ANTHROPIC_BASE_URL": "http://127.0.0.1:3456",
    "ANTHROPIC_AUTH_TOKEN": "any-random-string"
  }
}
```

- `hasCompletedOnboarding` skips the onboarding screen.
- `ANTHROPIC_AUTH_TOKEN` can be any non-empty string — it tricks Claude Code into thinking you are logged in.
- `ANTHROPIC_BASE_URL` points Claude Code to the local router instead of Anthropic's servers. Remember the host and port — they must match the router config in Step 4.

---

## Step 3 — Generate the router config

Run the router once:

```bash
npx @musistudio/claude-code-router
```

It won't do anything visible, but it will create the config file at:

```
%USERPROFILE%\.claude-code-router\config.json   # Windows
~/.claude-code-router/config.json               # Linux/macOS (unconfirmed)
```

---

## Step 4 — Edit the router config

Open the generated `config.json` and replace its contents with the following (using Gemini as a free provider):

```json
{
  "LOG": true,
  "LOG_LEVEL": "info",
  "HOST": "127.0.0.1",
  "PORT": 3456,
  "API_TIMEOUT_MS": 600000,
  "Providers": [
    {
      "name": "gemini",
      "api_base_url": "https://generativelanguage.googleapis.com/v1beta/models/",
      "api_key": "YOUR_GEMINI_API_KEY",
      "models": ["gemini-2.5-flash", "gemini-2.5-flash-lite"],
      "transformer": {
        "use": ["gemini"]
      }
    }
  ],
  "Router": {
    "default": "gemini,gemini-2.5-flash",
    "background": "gemini,gemini-2.5-flash-lite",
    "think": "gemini,gemini-2.5-flash",
    "longContext": "gemini,gemini-2.5-flash",
    "longContextThreshold": 60000
  }
}
```

> **Gemini API key:** Get a free key at [Google AI Studio](https://aistudio.google.com/apikey).
> Replace `"YOUR_GEMINI_API_KEY"` with your key, or set it as the environment variable `GEMINI_API_KEY`.

> **Important:** `HOST` and `PORT` here must match `ANTHROPIC_BASE_URL` in Step 2.
> The default example uses `http://127.0.0.1:3456` in both places.
> If port `3456` is already in use on your machine, pick any free port — but update it in **both** `config.json` and `settings.local.json`.

> **Free tier limits:** The Gemini API free tier (Google AI Studio) allows roughly 15 requests/min and 500 requests/day for Gemini 2.5 Flash. For light use this is plenty, but heavy sessions will hit the limit.

---

## Step 5 — Start the router

> **Important:** Keep this terminal open the entire time you use Claude Code. If you close it, Claude Code will stop working.

```bash
npx @musistudio/claude-code-router start
```

Or open the UI:

```bash
npx @musistudio/claude-code-router ui
```

Verify the router is running by visiting `http://127.0.0.1:3456` (or whatever host/port you configured) in your browser.

---

## Step 6 — Launch Claude Code and verify

```bash
npx @anthropic-ai/claude-code
```

Inside Claude Code, run:

```
/status
```

If you see `ANTHROPIC_BASE_URL: http://127.0.0.1:3456` in the output — everything is connected correctly.

---

## Step 7 — Test the model

Send any message to Claude Code. It will respond as if it were Claude, but under the hood it is Gemini. Setup is complete.

---

## Troubleshooting

| Problem | Check |
|---|---|
| Claude Code says not logged in | Make sure `ANTHROPIC_AUTH_TOKEN` is set in `settings.local.json` |
| Connection refused | Confirm the router is running and HOST/PORT match in both configs |
| Gemini returns auth errors | Double-check your API key in `config.json` |

---

## Video instruction (Russian)

https://www.youtube.com/watch?v=2fyCCIQJ3gs
