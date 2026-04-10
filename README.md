# Conversational AI using RLHF - Multi-Model Prompt Analyzer

> Built this to explore how RLHF (Reinforcement Learning from Human Feedback) actually works under the hood - not just read about it.

The idea was simple: take a raw user prompt, run it through a live AI model, get token-level NLP breakdown, score it, and generate 3 improved versions. Then let the user rate which version they prefer. Over time, that feedback builds a personal preference profile - which is the core loop behind how models like ChatGPT are fine-tuned.

Supports **Claude**, **Gemini**, and **Groq (Llama3 / Mixtral)** - all with free tiers. Zero npm dependencies. Just Node.js.

---

## What it actually does

When you type a prompt like `"make a website"`, the app sends it to whichever AI model you've selected. The model returns structured JSON with:

- **Token tagging** - every word gets labeled: `intent`, `entity`, `utterance`, `filler`, or `ambiguous`
- **Clarity / Specificity / Context scores** - computed by the model, scored 0-100
- **3 rewritten versions** of your prompt:
  - `Structured` - clean, formatted, no ambiguity
  - `Role-Framed` - adds expert persona + context
  - `Chain-of-Thought` - breaks it into step-by-step reasoning
- **RLHF feedback loop** - you 👍 or 👎 each version, and the app learns which style you prefer

The dashboard tracks everything: model usage, intent distribution, content domains, and your highest-rated prompt style.

---

## Tech decisions (and why)

**Zero dependencies** - I deliberately avoided Express, Axios, etc. Everything runs on Node's built-in `http`, `https`, `fs`, and `path`. This keeps the project lightweight and deployable anywhere Node runs.

**Single HTML file frontend** - No bundler, no framework. Pure vanilla JS with CSS variables and keyframe animations. Fast to load, easy to read.

**localStorage for RLHF state** - Ratings are persisted in the browser so your preference profile survives page refreshes without needing a database.

**Multi-model architecture** - All three providers (Anthropic, Google, Groq) use the same internal `callModel()` router. Adding a new model is just adding one function and a route.

---

## Getting started

You only need **one** API key to get it running. All three providers offer free tiers.

| Provider | Get your key | Env variable |
|----------|-------------|-------------|
| Google Gemini | [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey) | `GEMINI_API_KEY` |
| Anthropic Claude | [console.anthropic.com](https://console.anthropic.com) | `ANTHROPIC_API_KEY` |
| Groq (Llama3) | [console.groq.com/keys](https://console.groq.com/keys) | `GROQ_API_KEY` |

**Windows (PowerShell):**
```powershell
$env:GEMINI_API_KEY="AIza..."
node server.js
```

**Mac / Linux:**
```bash
export GEMINI_API_KEY="AIza..."
node server.js
```

Then open **http://localhost:3000** in your browser. 

---

## Project structure

```
conversational-ai-rlhf-multimodel/
├── server.js          - Node.js backend, zero npm deps
├── package.json
├── .gitignore
├── README.md
└── public/
    └── index.html     - Complete UI: chat, NLP panel, dashboard, about
```

---

## Models supported

**Gemini (Google)**
- `gemini-2.5-flash` - default, fast and capable
- `gemini-2.5-pro` - smarter, longer reasoning
- `gemini-2.0-flash` - reliable, stable
- `gemini-flash-latest` - points to latest flash

**Claude (Anthropic)**
- `claude-haiku-4-5` - fastest, lowest cost
- `claude-sonnet-4` - stronger reasoning

**Groq (open-source, free)**
- `llama3-8b-8192` - fastest
- `llama3-70b-8192` - smartest
- `llama-3.1-8b-instant`
- `llama-3.3-70b-versatile`
- `mixtral-8x7b-32768`
- `gemma2-9b-it`

---

## API

The backend exposes two endpoints:

```
POST /api/analyze
Body: { "message": "your prompt", "provider": "gemini", "model": "gemini-2.5-flash" }
Returns: { success, data: { tokens, overall_scores, intent_type, content_domain, improved_prompts } }

GET /api/health
Returns: { status: "ok", keys: { claude, gemini, groq } }
```

---

## What I learned building this

Getting the RLHF feedback loop right was the most interesting part. The `localStorage`-based preference tracking is a simplified version of reward modeling - instead of training weights, we track which prompt style scores highest per user. It's also where the real RLHF intuition clicked for me: the model doesn't learn from one rating, it learns from *patterns* across many.

The multi-model comparison was eye-opening too - the same vague prompt analyzed by Gemini vs Groq produces noticeably different token distributions and rewrite styles.

---

## License

MIT - use it, fork it, build on it.
