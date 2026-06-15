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
| `qwen/qwen3-coder:free` | Agentic coding, repo-scale edits | 1.05M |
| `qwen/qwen3-next-80b-a3b-instruct:free` | Long-context coding and RAG | 262K |
| `nvidia/nemotron-nano-9b-v2:free` | Fast reasoning/chat, simple coding | 128K |
| `nousresearch/hermes-3-llama-3.1-405b:free` | General assistant work, structured output | 128K |
| `liquid/lfm-2.5-1.2b-thinking:free` | Lightweight reasoning and extraction | 32K |
| `liquid/lfm-2.5-1.2b-instruct:free` | Fast lightweight chat | 32K |
| `openrouter/free` | Auto-picks a free model that supports your request | 200K |

Full current list at [openrouter.ai/collections/free-models](https://openrouter.ai/collections/free-models) — it changes as providers add and remove models.

There are also free non-chat utility models on OpenRouter:

| Model ID | Use case | Context |
|---|---|---|
| `nvidia/llama-nemotron-embed-vl-1b-v2:free` | Multimodal embeddings for text/images/documents | 131K |
| `nvidia/llama-nemotron-rerank-vl-1b-v2:free` | Multimodal reranking for RAG | 10K |
| `nvidia/nemotron-3.5-content-safety:free` | Prompt and response moderation | 128K |

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

OpenRouter's `:free` models use a request cap, not unlimited free usage:

- 50 requests per day if your account has bought less than $10 in credits
- 1000 requests per day after your account has bought at least $10 in credits
- 20 requests per minute for free model variants

Hit the per-minute limit and Claude Code usually shows `Retrying in 35s`. Wait a few minutes or switch models.

For normal daily work — fixing bugs, writing features, reviewing code — 50 prompts can go faster than you expect because agentic coding burns requests on tool loops. Long sessions on large codebases are where it bites hardest.

You can check your current key limits with OpenRouter's key endpoint:

```bash
curl https://openrouter.ai/api/v1/key \
  -H "Authorization: Bearer $OPENROUTER_API_KEY"
```

Free endpoints are for trial use. OpenRouter says prompts and outputs on free endpoints are logged to improve the provider's model and product, so don't send secrets, private code, customer data, or anything business-critical through them.

---

## What about Ollama Cloud?

Ollama Cloud is useful if you want bigger Ollama models without running them on your own GPU. It works like a remote Ollama host: your existing Ollama tools can call cloud models through Ollama's API.

Sign in first:

```bash
ollama signin
```

Then run a cloud model from the CLI:

```bash
ollama run gpt-oss:120b-cloud
```

For direct API access, create an Ollama API key and set it:

```bash
export OLLAMA_API_KEY=your_api_key
```

List available remote models:

```bash
curl https://ollama.com/api/tags \
  -H "Authorization: Bearer $OLLAMA_API_KEY"
```

Important Claude Code caveat: don't just replace `ANTHROPIC_BASE_URL` with `https://ollama.com/api`. Ollama Cloud exposes Ollama's API shape, while Claude Code expects Anthropic's API shape. To use Ollama Cloud behind Claude Code, you need an Anthropic-compatible proxy that translates Claude Code requests to Ollama requests.

For most people, OpenRouter is still the cleaner Claude Code setup. Ollama Cloud makes more sense if you are already using Ollama-native tools, or if you are willing to run a proxy layer.

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

*Last updated June 2026*
