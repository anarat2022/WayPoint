# Waypoint

**Daily bite-sized lessons on anything, built for the in-between moments.**

Waypoint turns any topic into a short daily lesson and a quick 4-question quiz — read on a commute, during a break, whenever there's five spare minutes. Each day builds on the last, so progress feels like a real curriculum instead of a pile of disconnected trivia.

**Live app:** [add your GitHub Pages link here]

---

## Why this exists

A friend wanted something like Duolingo, but for *anything* — process engineering one week, data science the next — not locked to a fixed subject. She also said she'd use it daily, which meant the design had to survive real, repeated, indefinite use without becoming expensive to run.

## What it does

- **Any topic, typed in** — not a fixed curriculum, whatever the learner is curious about
- **Progressive daily lessons** — each day builds on what came before, tracked via compact day-by-day summaries, not a fixed lesson bank
- **Auto-graded quizzes** — 4 multiple-choice questions per lesson, instant feedback with explanations, graded entirely on-device
- **Streaks** — consecutive days tracked per topic, to make showing up daily feel worth something
- **Multiple topics in parallel** — each tracked independently, like separate routes on a departure board
- **Free to reread, forever** — a topic's lesson is generated once per day and cached; reopening it ten times costs nothing extra
- **Zero accounts, zero database** — everything lives in the browser

## Architecture

![Waypoint architecture diagram](./waypoint-architecture.svg)

The core design constraint: **one AI call per topic, per day** — not per interaction. A Cloudflare Worker asks Gemini for the whole day's content in a single structured request (lesson, quiz with answers, and a one-line summary for tomorrow's context), and everything after that — rereading, retaking, checking answers — happens for free in the browser.

### Why context is compressed, not accumulated

Naively, "day 12 should remember days 1–11" means sending the full lesson history with every request — that grows and gets expensive fast. Instead, each generation call returns a single-sentence summary of what that day covered, and only the last several of those summaries (not full lesson text) get sent as context for the next day. The curriculum stays coherent while every request stays small and cheap, no matter how many days deep a topic goes.

### Why grading is client-side

The quiz's answer key comes back with the same generation call that creates the lesson. Checking whether a tap matches the correct index is pure logic — no reason to spend another API call confirming something the app already knows.

## Tech stack

| Layer | Choice |
|---|---|
| Frontend | Vanilla HTML/CSS/JS, single file, no build step |
| Hosting | GitHub Pages (free, static) |
| Backend | Cloudflare Worker (serverless, no server to maintain) |
| Content generation | Google Gemini API (`gemini-flash-latest`) |
| Persistence | `localStorage` — topics, streaks, lesson history |

## Project structure

```
waypoint.html          — the entire app: UI, state, API calls
waypoint-worker.js      — Cloudflare Worker: proxies Gemini,
                          keeps the API key server-side
waypoint-architecture.svg — the diagram above
```

## Running your own copy

1. **Get a Gemini API key** — [Google AI Studio](https://ai.google.dev), on a project with no billing attached, to stay on the free tier
2. **Deploy the Worker** — create a Cloudflare Worker, paste in `waypoint-worker.js`, add `GEMINI_API_KEY` as an encrypted secret
3. **Point the app at it** — open `waypoint.html`, set `WORKER_URL` near the top of the script to your deployed Worker's URL
4. **Host it** — push `waypoint.html` (renamed to `index.html`) to a GitHub repo, enable GitHub Pages in the repo settings

No database, no signup flow, no server to keep running.

## What I'd build next

- Optional reminder notification for the day's lesson
- A weekly digest showing streaks and topics at a glance
- Difficulty self-rating so lesson pacing can adapt, similar to how Praatje adjusts to a learner's level

---

Built for a friend who said she'd actually use it every day — designed to make that sustainable, not just possible.
