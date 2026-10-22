# Gill Sewing Machine Parts: website chatbot

A one-page website with an AI chat assistant for **Gill Sewing Machine Parts**, a sewing machine spare parts shop and repair workshop in Ludhiana, Punjab.

> **Demo project.** Gill Sewing Machine Parts is a fictional business made for a classroom exercise. All names, phone numbers and email addresses are placeholders.

## What the chatbot does

- Answers questions about parts, repairs, opening hours, location and shipping, using only the shop's own information.
- Helps customers find the right part by asking for the machine type, brand and model (customers often know only the model).
- Takes enquiries for parts, repairs and on-site servicing, then sends them to the owner as structured data (Google Sheet and/or Telegram).
- Marks dealer orders, on-site servicing for garment units, and servo motor orders of 10 or more as **high priority**.

It never quotes prices or repair costs, never promises stock or that a part will fit, never gives repair steps for electrical faults, and politely declines off-topic questions.

## Project files

| File | What it is |
|---|---|
| `worker.js` | The chatbot backend (a Cloudflare Worker). Section 2 holds the business facts, Section 3 the bot's rules. |
| `site/index.html` | The one-page website, with its own CSS and no frameworks. |
| `site/chat-widget.js` | The chat bubble, configured at the bottom of `index.html` with `window.SalesDeskConfig`. |
| `run-tests.mjs` | Sends the 15 test questions to the live bot (Node.js, no extra packages). |
| `test-results.md` | Results of the last test run. |

## How it works

```
Website (site/)  ──►  Cloudflare Worker (worker.js)  ──►  AI model (OpenAI-compatible API)
                              │
                              └──►  Enquiries to Google Sheet / Telegram (optional)
```

The browser only talks to the worker. The AI API key stays in the worker's settings and never reaches the website.

## Setup

### 1. Deploy the bot (Cloudflare Worker)

1. In the Cloudflare dashboard, create a Worker, paste in `worker.js`, and deploy.
2. Under **Settings → Variables and Secrets**, add:

| Name | Example | Notes |
|---|---|---|
| `LLM_BASE_URL` | `https://api.groq.com/openai/v1` | Any OpenAI-compatible API |
| `LLM_MODEL` | `openai/gpt-oss-120b` | |
| `LLM_API_KEY` | *your key* | Type: **Secret** |
| `ALLOWED_ORIGINS` | `https://your-site.pages.dev` | Comma-separated; `*` while testing |
| `LEAD_WEBHOOK_URL` | *Apps Script URL* | Optional: sends enquiries to a Google Sheet |
| `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` | | Optional: sends enquiries to Telegram |

Opening `https://<your-worker>.workers.dev/chat` in a browser should show `{"ok":true, ...}`.

### 2. Connect the website

In `site/index.html`, set `endpoint` in `window.SalesDeskConfig` to your worker's address ending in `/chat`.

### 3. Publish the website

Upload the `site` folder to any static host, for example **Cloudflare Pages → Upload assets**. Then add the site's address to `ALLOWED_ORIGINS`.

To preview locally:

```bash
cd site && python3 -m http.server 8765
```

Then open http://localhost:8765/. The chat does not load if you open `index.html` directly as a file.

## Testing

Requires Node.js 18 or later.

```bash
node run-tests.mjs
```

This runs tests 1 to 14, each as a new conversation, waiting 5 seconds between requests. Other options:

```bash
node run-tests.mjs only 7 9
```

```bash
node run-tests.mjs turn "message"
```

The first reruns selected tests. The second sends the next message of the full-enquiry conversation (test 15); `node run-tests.mjs reset` starts that conversation again. If the free AI plan's limit is hit (error 502), wait longer between requests with `WAIT_MS=20000`. Raw replies are saved in `test-output/`, which is not committed.

Last result: **15 / 15** (see `test-results.md`).

## Security

- Never put an API key in this repository or in the website. Keys belong only in the Cloudflare (or Vercel) settings.
- `.env` and `.dev.vars` files are ignored by git.

## Built with

The Sales Desk starter kit: plain HTML, CSS and JavaScript, a Cloudflare Worker, and an OpenAI-compatible language model.
