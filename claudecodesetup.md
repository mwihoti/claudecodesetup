# Claude Code for free — using OpenRouter

I cancelled my Claude subscription last week. Still using Claude Code every day.

Turns out Claude Code lets you swap the API endpoint. Point it at OpenRouter, which has a free tier with genuinely capable models, and you're done. Same CLI, same workflow, zero cost.

This is the guide I wish existed when I was figuring it out. Took me an embarrassing amount of trial and error.

---

## What you need before starting

- Linux, macOS, or Windows with WSL
- Node.js v18 or higher — check with `node --version`
- A terminal

That's it.

---

## Step 1 — Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

Check it installed:
```bash
claude --version
```

If you hit a permissions error on Linux or macOS:
```bash
sudo npm install -g @anthropic-ai/claude-code
```

---

## Step 2 — Get your free OpenRouter key

Go to [openrouter.ai](https://openrouter.ai), sign up, and create an API key under your account dashboard. No credit card. The key starts with `sk-or-v1-...`.

Keep it somewhere — you'll need it in the next step.

---

## Step 3 — Configure Claude Code

This is the part that tripped me up. Claude Code has a config file at `~/.claude/settings.json`. Create it if it doesn't exist:

```bash
mkdir -p ~/.claude
nano ~/.claude/settings.json
```

Paste this in (swap `your-key-here` for your actual OpenRouter key):

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://openrouter.ai/api",
    "ANTHROPIC_AUTH_TOKEN": "your-key-here",
    "ANTHROPIC_API_KEY": "",
    "ANTHROPIC_MODEL": "meta-llama/llama-3.3-70b-instruct:free"
  },
  "theme": "dark"
}
```

Save and close (`Ctrl+O`, Enter, `Ctrl+X` in nano).

**Why each line matters:**

- `ANTHROPIC_BASE_URL` — points Claude Code at OpenRouter instead of Anthropic
- `ANTHROPIC_AUTH_TOKEN` — your OpenRouter key goes here, not in API_KEY
- `ANTHROPIC_API_KEY` — leave this empty or Claude Code tries to fall back to Anthropic
- `ANTHROPIC_MODEL` — the free model to use

> One thing that got me: the URL must be `https://openrouter.ai/api` — no `/v1` at the end. With `/v1` you get a silent empty response that's genuinely hard to debug.

---

## Step 4 — Test it

```bash
exec bash
claude
```

You should see the Claude Code welcome screen with the model name in the bottom left corner. Type `hi` — if it responds, you're good to go.

---

## Free models worth using

Start with Llama 3.3 — it follows Claude Code's tool-use format the most reliably. The others are good but occasionally quirky.

| Model ID | Good for | Context |
|---|---|---|
| `meta-llama/llama-3.3-70b-instruct:free` | Everything, most reliable | 128K |
| `nvidia/nemotron-3-super-120b-a12b:free` | Long codebases, big context | 1M |
| `deepseek/deepseek-r1:free` | Reasoning-heavy tasks | 64K |
| `google/gemma-3-27b-it:free` | Lightweight, fast | 128K |
| `mistralai/mistral-small-3.1-24b-instruct:free` | Quick tasks | 128K |

Full current list at [openrouter.ai/collections/free-models](https://openrouter.ai/collections/free-models) — it changes as providers add and remove models.

---

## Switching models

When a model gets throttled or disappears, just edit the config:

```bash
nano ~/.claude/settings.json
```

Change `ANTHROPIC_MODEL` to a different model ID, then `exec bash && claude`.

I keep 2-3 model IDs in a notes file so I can swap quickly without having to look them up.

---

## How the free limits work

There are two kinds of limits and they're easy to confuse:

**Rate limits** — how many requests per minute. Hit this and you get the `Retrying in 35s` message. Usually clears in a few minutes.

**Daily token budget** — each model has a daily allocation. Burn through it and you need to either wait for reset (24 hours) or switch to a different model. Limits are per model, so switching buys you a fresh budget.

For normal daily work — fixing bugs, writing features, reviewing code — you probably won't hit the daily limit. Where it bites you is long sessions on large codebases where Claude reads hundreds of files.

---

## Making your tokens last longer

The biggest token drain in Claude Code is context — every file it reads, every message in the session, all of it adds up.

**Run `/clear` between tasks.** Starting a new task in the same session carries all the old context. `/clear` resets it.

**Be specific.** "Fix my auth system" causes Claude to explore your whole codebase. "Fix the JWT validation bug in `src/auth/middleware.js` around line 45" goes straight to the problem.

**Use `/compact`.** This compresses conversation history without losing context. Run it when a session gets long.

**Launch from your project folder.** Running `claude` from your home directory means it might scan everything. `cd` into your project first.

---

## Troubleshooting

### `API returned an empty or malformed response (HTTP 200)`

You have `/v1` at the end of your base URL. Change it to `https://openrouter.ai/api` with nothing after it.

---

### `400 — not a valid model ID`

Two things to check:

1. The model ID might be wrong or the model was renamed. Verify at [openrouter.ai/collections/free-models](https://openrouter.ai/collections/free-models).

2. Claude Code has a bug where it prepends `openrouter/` to your model name, turning `meta-llama/...` into `openrouter/meta-llama/...`. If you see that prefix in error messages, make sure you're setting the model via `ANTHROPIC_MODEL` in the `env` block only — remove any top-level `"model"` key from settings.

---

### `Retrying in 35s · attempt X/10`

Rate limited. Options:
- Wait a few minutes
- Switch to a different model
- Run `/clear` to reduce your context size before retrying

---

### `There's an issue with the selected model`

The model is offline or unavailable. Switch to `meta-llama/llama-3.3-70b-instruct:free` — it's the most consistently available one.

---

### Wrong model showing in the UI

A project-level config is overriding your global one. Find it:

```bash
find ~ -name "settings.json" -path "*/.claude/*" 2>/dev/null
```

Update or delete the conflicting file.

---

### Nothing is working

Run this to confirm what config Claude Code is actually reading:

```bash
cat ~/.claude/settings.json
exec bash
claude --version
```

Also check [openrouter.ai](https://openrouter.ai) directly — sometimes models go down on their end.

---

## Questions people ask

**Is this against Anthropic's ToS?**
No. Custom base URL support is built into Claude Code by Anthropic. You're using it as intended, just with a different backend.

**Do limits reset daily?**
Most do, every 24 hours. Some providers top up budgets less frequently. If a model seems permanently broken, it might have been retired — check the free models list and try another.

**Can I use this for real projects?**
Yes, within reason. Works well for personal projects and day-to-day development. For a production team that needs consistent availability, putting some credits on OpenRouter — even $5-10 — makes a big difference in rate limits.

**Why not use `openrouter/auto`?**
It rotates between free models automatically which sounds convenient, but you lose predictability. Model behavior varies enough that it's worth picking one you trust.

**It's slow. Is something wrong?**
Free models are shared infrastructure. Peak hours mean slower responses. Try a different model or come back later.

**How do I update Claude Code?**
```bash
npm update -g @anthropic-ai/claude-code
```

---

## Found something broken?

Model IDs change, new free models appear, old ones disappear. If something in this guide is outdated, open a PR or issue. Keeping it accurate helps everyone.

---

*Last updated May 2026*
