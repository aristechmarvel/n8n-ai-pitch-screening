*AI-assisted pitch screening for an early-stage investor, built with n8n, Google Gemini, Google Sheets, and Gmail.*

Founders submit a pitch through a public form. The system acknowledges them, scores the pitch against the investor's thesis, replies to the founder, alerts the CEO about exceptional pitches, and delivers a ranked morning digest of the best ones.

> **Status:** v1, single investor, tested on a small pilot. See [Results](#results) and [Limitations](#limitations).

---

## The problem

Investors receive far more pitches than they can read. Reading each one is slow, and replying to each one is slower. Founders, meanwhile, expect at least an acknowledgment, and most never get one.

This architecture was built to cut the triage workload without losing a good pitch or leaving founders in silence.

## Architecture

![AI Pitch Screening & Investor Automation architecture](docs/architecture/ai-pitch-screening-architecture.png)

The system combines three connected n8n workflows:

* **Intake & Scoring** — captures, validates, scores, and routes incoming pitches
* **Daily Digest** — ranks qualifying pitches and delivers a morning decision-support digest
* **Error Notifier** — provides cross-workflow failure detection and admin alerts

The architecture separates **AI evaluation from deterministic business rules**, while adding validation, fallback handling, persistence, routing, and monitoring.

## What it does

1. **Acknowledges** the founder within seconds of submission
2. **Saves** the pitch to a Google Sheet, which doubles as the deal-flow database
3. **Scores** the pitch against the investor's thesis using an AI agent
4. **Replies** to the founder after scoring, using fixed templates
5. **Alerts** the CEO immediately for exceptional pitches (9+ out of 10)
6. **Digests** the day's best pitches (7+) into a ranked email at 7am

## Screenshots

### Founder submission form

![Submission form](docs/screenshots/submission-form.png)

### Intake and Scoring workflow (n8n)

![Intake and Scoring workflow in n8n](docs/screenshots/intake-workflow.png)

### Scored submissions in the Sheet

![Scored submissions in Google Sheets](docs/screenshots/sheet-scored-rows.png)

*Sample data, hand-entered for illustration. Some columns are hidden for readability.*

### Daily Digest workflow (n8n)

![Daily Digest workflow in n8n](docs/screenshots/daily-digest-workflow.png)

### Workflow 1: Intake and Scoring

```mermaid
flowchart TD
    A[Public n8n form] --> B[Build Payload]
    B --> C[Remove Duplicates]
    C --> D[Acknowledge Applicant]
    C --> E[Append Row to Sheet]
    E --> F[AI Agent scores pitch]
    F -->|success| G[Score and Flatten]
    F -->|error| H[Mark Failed]
    H --> I[Alert admin]
    G --> J[Update Row]
    J --> K[Final Response to founder]
    J --> L{Score 9 or higher}
    L -->|yes| M[Alert CEO]
```

### Workflow 2: Daily Digest

```mermaid
flowchart TD
    A[Schedule 7am] --> B[Get scored rows]
    B --> C[Filter last 24h and rank]
    C --> D{Any pitches}
    D -->|yes| E[AI Agent writes digest]
    E --> F[Send digest to CEO]
    D -->|no| G[Send quiet day note]
```

### Workflow 3: Error Notifier

An Error Trigger workflow, linked to both workflows above, emails the admin when anything fails unexpectedly.

## Repository structure

```text
n8n-ai-pitch-screening/
├── README.md
├── workflows/
│   ├── intake-and-scoring.json
│   ├── daily-digest.json
│   └── error-notifier.json
└── docs/
    ├── architecture/
    │   └── ai-pitch-screening-architecture.png
    └── screenshots/
        ├── submission-form.png
        ├── intake-workflow.png
        ├── sheet-scored-rows.png
        └── daily-digest-workflow.png
```
