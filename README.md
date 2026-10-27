# SimuDrive Timer ⏳

A simple, reliable countdown timer that keeps every candidate on a driving-simulator training session within their allotted time (45 minutes by default).

No framework, no build step, no dependencies: a single `index.html` file (HTML + CSS + vanilla JavaScript).


---

## Features

- **Candidate queue**: add names, start the next candidate with one click.
- **45-minute countdown** per candidate, adjustable (1–180 min) between sessions.
- **Pause / resume / finish early**, plus **delete** a paused candidate.
- **Visual phases**: green while running, orange during the last 5 minutes, red once time is up.
- **Overtime tracking**: the timer keeps counting after 00:00 (`+02:15`) so overruns are visible and recorded.
- **Audio and vibration alerts**: one beep at 5 minutes remaining, three beeps at the end.
- **Session history**: time used per candidate, with overtime highlighted.
- **Persistence**: state survives page refreshes (stored in the browser).
- **Responsive** (phone, tablet, desktop), RTL Arabic interface, automatic dark mode, keyboard-accessible, `prefers-reduced-motion` respected.

## How to use

1. Type a candidate's name and press **Add**.
2. Press **Start** on their row (or **Start next** in the timer panel).
3. Use **Pause** if the session is interrupted. A **Delete** button appears next to a paused candidate.
4. Press **Finish** when the candidate is done. They move to the history list with the time they used.

> Browsers block sound until the first user interaction. The first press on **Start** unlocks audio alerts.

## How it works

### Accurate timekeeping
The timer never counts ticks. Each running session stores a start timestamp, and the remaining time is computed on every refresh:

```
elapsed   = accumulatedMs + (now - startedAt)
remaining = duration - elapsed
```

The display therefore stays correct even if the tab is in the background, throttled, or the page is reloaded.

### Architecture

The code in `index.html` is organized in seven clearly separated sections:

| Section | Responsibility |
|---|---|
| **A. Config** | Constants: storage key, warning threshold, ring geometry |
| **B. State + Storage** | One `state` object, `load()` / `save()` to `localStorage` |
| **C. Timer engine** | `start`, `pause`, `finish`, `elapsed`: timestamp-based logic only |
| **D. Alerts** | Web Audio beeps and vibration, each fired once per session |
| **E. Render** | `render()` (lists and buttons) and `renderClock()` (250 ms refresh) |
| **F. Events** | Form, buttons and list actions |
| **G. Boot** | `load()`, first `render()`, clock interval |

### Data model

```js
state = {
  duration: 45,                 // minutes
  list: [{
    id, name,
    status: "waiting" | "running" | "paused" | "done",
    accMs,                      // time accumulated before the current run
    startedAt,                  // timestamp of the current run
    usedMs,                     // final time used (when done)
    warned, ended               // alert flags
  }]
}
```

Only one candidate can be `running` or `paused` at a time (one simulator, one session).

## Project structure

```
.
├── index.html        # the whole app
├── manifest.json     # PWA metadata
├── service-worker.js # minimal service worker
├── icons/            # app icons used by manifest + favicon
└── README.md
```

## Run locally

Open `index.html` in any modern browser. To serve it over HTTP instead:

```bash
npx serve .
# or
python3 -m http.server 8000
```

## Deploy on Vercel

1. Push this repository to GitHub, with `index.html` at the root.
2. On [vercel.com](https://vercel.com): **Add New → Project**, then import the repository.
3. Set **Framework Preset** to **Other**. Leave Build Command and Output Directory empty.
4. Click **Deploy**. Every push to the main branch redeploys automatically.

## Configuration

Edit the constants at the top of the `<script>` block in `index.html`:

| Constant | Default | Meaning |
|---|---|---|
| `WARN_MIN` | `5` | Minutes remaining when the warning phase and beep trigger |
| `KEY` | `simudrive-timer-v1` | `localStorage` key (change it to reset saved data for everyone) |
| `state.duration` | `45` | Default session length in minutes (also editable in the UI) |

## Limitations

- Data is stored **per browser and per device**. It is not synced between devices.
- Clearing the browser data erases the queue and history.
- Fonts are loaded from Google Fonts; without a connection the app falls back to system fonts and works the same.

## Ideas for next versions

- Export the history to CSV / Excel (name, time used, overtime).
- Editable names and reordering of the queue.
- Per-candidate custom duration.
- Optional shared state (e.g. a database) to follow sessions from several devices.
