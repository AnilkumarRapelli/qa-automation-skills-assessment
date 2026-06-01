# Task 2 — n8n API Integration Workflow

**Author:** Anilkumar  
**Date:** June 2026  
**Workflow file:** `Task2_Workflow_Anilkumar.json`

---

## What This Workflow Does

This workflow delivers an automated **HackerNews Morning Brief** to a Discord channel every hour. It fetches the top trending tech stories, enriches them with full story data, labels them by popularity, and posts a formatted digest.

---

## APIs Used & Why

| API | Endpoint | Reason |
|-----|----------|--------|
| HackerNews Firebase API | `/v0/topstories.json` | Free, no key required, returns live ranked story IDs |
| HackerNews Firebase API | `/v0/item/{id}.json` | Second call — enriches each ID with title, score, author, URL |
| Discord Webhook | Incoming Webhook URL | Free, instant delivery, no bot setup needed |

HackerNews was chosen because it requires zero API credentials for read access, making this workflow immediately runnable by any reviewer without setup friction.

---

## Workflow Flow

```
Schedule (every 1h)
  → Fetch top story IDs  [First HTTP Request]
  → Slice Top 5 IDs      [Code node — transformation]
  → Fetch story details  [Second HTTP Request — enrichment]
  → IF score >= 100      [Conditional branch]
      True  → Label: HOT 🔥
      False → Label: TRENDING 📈
  → Build Digest Message [Code node — merge + format]
  → Post to Discord      [Output]
```

---

## Transformation Logic

The **Slice Top 5 IDs** Code node:
- Takes the raw array of 500+ story IDs returned by the first API call
- Slices the first 5 (these are already ranked by HN's algorithm)
- Outputs 5 individual items so the next node loops over each

The **Build Digest Message** Code node:
- Merges outputs from both branches (HOT and TRENDING)
- Formats a human-readable Markdown message with title, score, author, comment count, and link

---

## Conditional Branch

The **IF Score >= 100** node routes each story:
- **True branch (score ≥ 100):** Story labelled `🔥 HOT` — indicates strong community engagement
- **False branch (score < 100):** Story labelled `📈 TRENDING` — still noteworthy but less viral

This threshold was chosen because HN's median front-page story scores around 100–200 points within an hour.

---

## Error Handling

Every HTTP Request node has **Continue On Fail** enabled and connects to a dedicated **Error Handler** code node on its error output. The Error Handler:

1. Logs the failed node name, error message, and timestamp to the n8n execution log
2. Formats a structured error object for inspection in the execution history
3. Does **not** silently swallow errors — all failures are surfaced in the execution panel

This means a single story fetch failure does not crash the entire workflow; the remaining stories are still processed and delivered.

---

## Credentials Setup

The Discord webhook URL is stored as an **n8n Credential** of type `Discord Webhook`. It is referenced by name in the Discord node — no secrets are hardcoded in the workflow JSON.

To set up:
1. In n8n → Credentials → New → Discord Webhook
2. Paste your Discord channel webhook URL
3. Name it `Discord Webhook (Morning Brief)`
4. Save — the workflow will automatically use it

---

## Bonus: Uptime Monitor

See `Bonus_UptimeMonitor_Anilkumar.json`. Pings `demo.realworld.io` every 5 minutes, checks for HTTP 200, and posts an `🚨 UPTIME ALERT` to the same Discord channel if the app is down or returns any non-200 status. Connection timeouts are also treated as failures via the error output branch.
