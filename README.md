# Automation & QA Developer — Skills Assessment

**Candidate:** Anilkumar  
**Role:** Automation & QA Developer  
**Submitted:** June 2026

---

## Repository Structure

```
├── Task1_QA_Report_Anilkumar.docx     # Bug report for RealWorld demo app
├── Task2_Workflow_Anilkumar.json      # n8n Morning Brief workflow
├── Bonus_UptimeMonitor_Anilkumar.json # n8n Uptime Monitor workflow (bonus)
├── README_Task2_Anilkumar.md          # Detailed workflow documentation
└── README.md                          # This file
```

---

## Task 1 — Web App QA & Debug Report

**App tested:** [demo.realworld.io](https://demo.realworld.io)  
**Scope:** Functional, Security, UX, Performance, Accessibility

### Bugs Found (6 total)

| # | Title | Severity |
|---|-------|----------|
| 1 | No client-side validation on Sign-Up form | High |
| 2 | JWT stored in localStorage — XSS risk | **Critical** |
| 3 | Duplicate article submission on double-click | High |
| 4 | Markdown not rendered in article preview | Medium |
| 5 | No pagination limit — 500+ articles cause UI freeze | High |
| 6 | Password field allows spaces as valid input | Medium |

**Root-cause deep-dive:** Issue #2 (JWT in localStorage) — see the full report for a 5-sentence analysis covering what, why, and how to fix it.

**Full report:** `Task1_QA_Report_Anilkumar.docx`

---

## Task 2 — n8n API Integration Workflow

**Workflow:** HackerNews Morning Brief  
**Trigger:** Schedule (every 1 hour)

### Architecture

```
Schedule → Fetch HN Top IDs → Slice Top 5 → Fetch Story Details → IF score≥100 → Label HOT/TRENDING → Build Digest → Post to Discord
                                                                                                              ↑
                                                                                              Error Handler (all nodes)
```

### Key Design Choices
- **HackerNews API** — no credentials required; immediately runnable
- **Two API calls** — first fetches IDs, second enriches each with full story data (title, score, author, URL, comment count)
- **Threshold = 100 points** — aligns with typical HN front-page virality threshold
- **Discord webhook via n8n Credentials** — no secrets in the workflow JSON
- **continueOnFail on all HTTP nodes** — errors routed to Error Handler, never silent

See `README_Task2_Anilkumar.md` for full documentation.

---

## Bonus — Uptime Monitor

**Workflow:** `Bonus_UptimeMonitor_Anilkumar.json`

Pings `demo.realworld.io` every 5 minutes. If response is not HTTP 200 (or connection fails), posts a `🚨 UPTIME ALERT` to the same Discord channel. Uses the same credentials as Task 2.

---

## How to Run

### Task 2 / Bonus Workflows

1. Install n8n: `npx n8n` or Docker (`docker run -it --rm --name n8n -p 5678:5678 n8nio/n8n`)
2. Open `http://localhost:5678`
3. Import workflow JSON: Workflows → Import → select the JSON file
4. Set up Discord Webhook credential (Credentials → New → Discord Webhook)
5. Activate the workflow

### Running Tests Manually
Hit the workflow manually in n8n's UI to verify the execution before scheduling.

---

## Tech Stack

- **n8n** (self-hosted or cloud) — workflow automation
- **HackerNews Firebase API** — free public data source
- **Discord Webhooks** — notification channel
- **JavaScript (n8n Code nodes)** — transformation and formatting logic
