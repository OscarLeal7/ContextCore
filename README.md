# ContextCore

> Your personal context server. Observes how you work, builds a real-time model of your cognitive state, and exposes an open API that any tool can consume.

```bash
npx context-core start
```

---

## The problem

You end every day not knowing exactly where your time went. Your calendar says one thing, your commits say another, your gut says a third. Existing productivity tools either require manual input (nobody sustains that) or record your screen (invasive and closed).

ContextCore takes a different approach: **it listens to signals you already generate** — commits, file opens, meetings, terminal commands — and builds an automatic, structured picture of your workday. No extra input. No screen recording. Your data stays on your infrastructure.

---

## How it works

```
[ collectors ]  →  [ pub/sub ]  →  [ inference engine ]  →  [ context API ]
   git                                  state classifier         REST + WS
   vscode                               pattern detector         open protocol
   teams                                anomaly detector
   outlook
   calendar
   terminal
```

A lightweight daemon runs in the background. Pluggable collectors emit normalized events to a local queue. The inference engine classifies your current cognitive state in real time — **deep work, shallow work, blocked, context switching** — using a model that learns from your own historical data. The Context API exposes everything over REST and WebSocket so any tool can consume it.

---

## Key design principles

**1. You own your data.**
All events are stored in your own Supabase instance. Nothing is sent to third-party servers unless you explicitly configure it. The LLM summarizer (Claude API) is called on-demand only — not in a continuous loop.

**2. Passive by default.**
Zero manual input required. ContextCore observes signals you already generate. If you stop using it, it stops collecting. No background drain.

**3. Open protocol.**
The Context API is not a walled garden. Any tool — your own scripts, other apps, the [Adaptive OS](https://github.com/OscarLeal7/adaptive-os) — can query your current state and build on top of it.

**4. Pluggable collectors.**
Each data source is an independent package implementing a common `Collector` interface. Install only what you need. Build your own.

---

## Architecture

```
context-core/
├── daemon/                  # Core daemon — Node.js CLI
│   └── src/
│       ├── cli.ts           # Entry point — `context-core start | stop | status`
│       ├── collectors/      # Collector registry + loader
│       ├── engine/          # State inference + pattern detection
│       │   ├── classifier.ts
│       │   ├── features.ts  # Sliding window feature extraction
│       │   └── patterns.ts  # Anomaly + rhythm detection
│       └── api/             # REST + WebSocket server (port 3001)
│           ├── server.ts
│           ├── routes.ts
│           └── ws.ts
│
├── collectors/              # Pluggable collector packages
│   ├── git/                 # Git hooks + polling
│   ├── vscode/              # VSCode extension
│   ├── teams/               # Microsoft Graph API
│   ├── outlook/             # Microsoft Graph API (mail + calendar)
│   ├── calendar/            # Google Calendar API
│   └── terminal/            # Shell history listener
│
└── web/                     # Next.js dashboard — deploy on GCP Cloud Run
    └── src/
        ├── app/             # App Router pages
        └── components/      # UI components
```

---

## The Context API

Once the daemon is running, any tool can query your state:

```bash
# Current cognitive state
GET http://localhost:3001/context/state

# Response
{
  "state": "deep_work",
  "confidence": 0.87,
  "since": "2026-03-23T14:30:00Z",
  "signals": {
    "commits_last_30m": 3,
    "context_switches_last_hour": 2,
    "meeting_active": false,
    "last_message_sent": "47m ago"
  }
}

# Today's timeline
GET http://localhost:3001/context/timeline?date=today

# WebSocket — subscribe to state changes
WS ws://localhost:3001/context/live
```

---

## Collectors

| Collector | What it captures | Auth required |
|-----------|-----------------|---------------|
| `@context-core/git` | Commits, branch switches, PR activity | None (local) |
| `@context-core/vscode` | File opens, active project, idle detection | VSCode extension |
| `@context-core/teams` | Messages sent/received, meetings, status | Microsoft Graph |
| `@context-core/outlook` | Emails sent, calendar events | Microsoft Graph |
| `@context-core/calendar` | Events, meeting duration, free/busy | Google OAuth |
| `@context-core/terminal` | Commands run, working directory | Shell plugin |

---

## Quick start

```bash
# Install daemon globally
npm install -g @context-core/daemon

# Initialize — creates ~/.context-core/config.json
context-core init

# Add collectors
context-core collector add @context-core/git
context-core collector add @context-core/vscode
context-core collector add @context-core/teams

# Start
context-core start

# Check status
context-core status
```

---

## Stack

- **Daemon**: Node.js · TypeScript · Express · WebSocket
- **Queue**: GCP Pub/Sub (cloud) · in-process queue (local mode)
- **Storage**: Supabase (PostgreSQL + pgvector)
- **Inference**: TensorFlow.js (local model) + sliding window features
- **Summarizer**: Claude API (on-demand)
- **Dashboard**: Next.js 14 · Tailwind · deploy on GCP Cloud Run

---

## Roadmap

- [x] Core daemon + CLI
- [x] Collector interface + registry
- [x] Git collector
- [x] State inference engine (rule-based v1)
- [ ] VSCode collector
- [ ] Teams + Outlook collectors (Microsoft Graph)
- [ ] Google Calendar collector
- [ ] ML classifier v2 (learns from your data)
- [ ] Daily/weekly insight summarizer (Claude API)
- [ ] Next.js dashboard
- [ ] Adaptive OS integration

---

## Philosophy

Most productivity tools want you to change your behavior to fit their system. ContextCore adapts to you. It observes what you already do, learns your patterns, and over time builds a model that is uniquely yours.

The goal is not surveillance. It's self-knowledge at a resolution that was never possible before.

---

## License

MIT — build whatever you want on top of this.
