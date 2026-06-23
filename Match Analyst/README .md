# ⚽ Match Analyst — AI-Powered Football Analysis Platform

> Type two team names and a match date. Get a full AI-generated match report in seconds.

**Live Site:** [ai-match-analyst.netlify.app](https://ai-match-analyst.netlify.app)
**Backend:** [omar7055.app.n8n.cloud](https://omar7055.app.n8n.cloud)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Tech Stack](#tech-stack)
- [API Reference](#api-reference)
- [Workflow Logic](#workflow-logic)
  - [Workflow 1 — Match Analysis (35 nodes)](#workflow-1--match-analysis-35-nodes)
  - [Workflow 2 — Head-to-Head (14 nodes)](#workflow-2--head-to-head-14-nodes)
  - [Workflow 3 — Player Comparison (28 nodes)](#workflow-3--player-comparison-28-nodes)
  - [Error Detection Workflow (4 nodes)](#error-detection-workflow-4-nodes)
- [AI Architecture](#ai-architecture)
- [Frontend Integration](#frontend-integration)
- [Credential Setup](#credential-setup)
- [Deployment](#deployment)
- [Error Handling](#error-handling)
- [Known Limitations](#known-limitations)
- [Testing](#testing)
- [Changelog](#changelog)
- [Team](#team)

---

## Overview

**Match Analyst** is a graduation project built at Fayoum University, Faculty of Computers & Artificial Intelligence (2026). It is a fully serverless, AI-powered football analysis web platform built on a three-layer architecture:

```
USER BROWSER (Netlify)
       │
       │  POST /webhook/{path}
       ▼
n8n CLOUD BACKEND (omar7055.app.n8n.cloud)
  ├── Workflow 1 — Match Analysis     (35 nodes)
  ├── Workflow 2 — Head-to-Head       (14 nodes)
  ├── Workflow 3 — Player Comparison  (28 nodes)
  └── Error Detection                 (4 nodes)
       │                    │
       ▼                    ▼
API-Football v3      Google Gemini 2.5 Flash
```

**Zero custom server code. Zero database. Zero deployment pipeline.**

---

## Features

| Feature | Description |
|---------|-------------|
| ⚽ **Match Analysis** | Full breakdown: score, stats, lineups, injuries, form, ratings, MOTM, AI tactical report & timeline |
| ⚖️ **Head-to-Head** | Auto-loads the moment both teams are typed — no button click needed (900ms debounce) |
| 👥 **Player Comparison** | Match-contextual AI comparison — only players from the analyzed fixture's squads are valid |
| 🚨 **Error Monitoring** | Email alert on any node failure — includes failed node name, error message & direct execution link |

---

## System Architecture

### Layer 1 — Frontend
- Single `index.html` file — no framework, no build step
- Vanilla JavaScript with `fetch()` for webhook calls
- Hosted on Netlify CDN with automatic HTTPS
- Three webhook URL constants at the top of the file

### Layer 2 — Backend (n8n Cloud)
Four workflows, **81 nodes total**, all triggered via webhook POST requests and responding synchronously via `Respond to Webhook` nodes.

### Layer 3 — External Services
- **API-Football v3** — authenticated via `x-apisports-key` HTTP header
- **Google Gemini 2.5 Flash** — accessed via n8n AI Agent nodes with Structured Output Parser
- **Gmail OAuth2** — used exclusively by the Error Detection workflow

---

## Tech Stack

| Technology | Role | Why |
|-----------|------|-----|
| **n8n Cloud** | Backend automation | Replaces a custom server — handles webhook routing, HTTP calls, data transformation, and AI orchestration with no backend code |
| **API-Football v3** | Data source | Real-time fixtures, stats, events, lineups, players, injuries, H2H. Free tier: 100 req/day |
| **Google Gemini 2.5 Flash** | AI engine | Free tier. Constrained JSON output via Structured Output Parser. 3 agents across 2 workflows |
| **Netlify** | Frontend hosting | Zero-config CDN. Deployed by drag-and-drop. Automatic HTTPS |
| **HTML / CSS / JS** | Frontend | No framework. Single file. `fetch()` for webhook calls. Bebas Neue + DM Sans + DM Mono typefaces |
| **Gmail OAuth2** | Error alerting | HTML email alerts via n8n Error Trigger on any workflow failure |

---

## API Reference

All endpoints accept `POST` with `Content-Type: application/json` and return JSON. All responses include a top-level `success` boolean.

> ⚠️ **Production vs Test URLs**
> Production URLs (`/webhook/`) work 24/7 once published.
> Test URLs (`/webhook-test/`) only respond while the workflow is open in the n8n editor with "Listen for test event" active.

---

### `POST /webhook/match-analysis`

**Workflow 1 — Match Analysis**

#### Request Body

```json
{
  "team1": "Canada",
  "team2": "Qatar",
  "matchDate": "2026-06-18"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `team1` | string | ✅ | Name of the first team |
| `team2` | string | ✅ | Name of the second team. Must differ from `team1` |
| `matchDate` | string | ✅ | Date in `YYYY-MM-DD` format |

#### Success Response `200`

```json
{
  "success": true,
  "fixture": {},
  "score": { "home": 6, "away": 0 },
  "competition": { "name": "FIFA World Cup", "round": "Group Stage" },
  "venue": { "name": "SoFi Stadium", "city": "Inglewood" },
  "stats": [],
  "events": [],
  "lineups": [],
  "form": {
    "team1": { "results": ["W", "W", "W", "D", "W"], "points": 13 },
    "team2": { "results": ["L", "L", "D", "L", "L"], "points": 1 }
  },
  "injuries": [],
  "playerRatings": {
    "bestGoalkeeper": { "name": "Milan Borjan", "rating": 7.4 },
    "bestDefender":   { "name": "Alistair Johnston", "rating": 7.8 },
    "bestMidfielder": { "name": "Stephen Eustaquio", "rating": 8.1 },
    "bestAttacker":   { "name": "Jonathan David", "rating": 9.1 }
  },
  "motm": { "name": "Jonathan David", "rating": 9.1, "goals": 2, "assists": 1 },
  "timeline": [
    { "minute": "11'", "type": "goal", "event": "Jonathan David scored from close range..." }
  ],
  "analysis": {
    "tacticalDifferences": "...",
    "keyTurningPoints": "...",
    "impactOfAbsences": "...",
    "bestPerformers": "..."
  },
  "headToHead": {
    "team1": "Canada", "team2": "Qatar",
    "totalMeetingsFound": 2,
    "wins1": 2, "wins2": 0, "draws": 0,
    "meetings": []
  }
}
```

---

### `POST /webhook/head-to-head`

**Workflow 2 — Head-to-Head**

#### Request Body

```json
{
  "team1": "Canada",
  "team2": "Qatar"
}
```

**Business Rules:**
- Only `FT`, `AET`, `PEN` completed matches are included
- Results sorted newest first
- Capped at 10 meetings if ≥ 10 exist
- Win/draw counts computed from `team1`'s perspective

---

### `POST /webhook/player-comparison`

**Workflow 3 — Player Comparison**

#### Request Body

```json
{
  "player1": "Jonathan David",
  "player2": "Cyle Larin",
  "fixture_id": 1035037
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `player1` | string | ✅ | Player name or partial name |
| `player2` | string | ✅ | Player name or partial name. Must differ from `player1` |
| `fixture_id` | number | ✅ | API-Football fixture ID. **Sent automatically by the frontend — users never type this** |

---

### Error Response Format

```json
{
  "success": false,
  "errors": ["No completed fixture found between Canada and Qatar on 2026-06-19"]
}
```

| Status | When |
|--------|------|
| `200` | Success |
| `400` | Invalid input (missing fields, same team, bad date format) |
| `404` | Data not found (team not resolved, fixture not found, player not in squad) |

---

## Workflow Logic

### Workflow 1 — Match Analysis (35 nodes)

The full pipeline, node by node:

```
Webhook → Normalize Input → Validate Input → Is Input Valid?
                                                    │
                              ┌─────────────────────┘
                              │ TRUE
                              ▼
                    Search Team 1 → Search Team 2 → Resolve Team IDs → Are Teams Resolved?
                                                                                │
                                                          ┌─────────────────────┘
                                                          │ TRUE
                                                          ▼
                                              Fetch Fixtures By Date → Find Fixture → Is Fixture Found?
                                                                                              │
                                                                        ┌─────────────────────┘
                                                                        │ TRUE
                                                                        ▼
                                              ┌─────────────────────────────────────────────────────┐
                                              │           DATA COLLECTION (all neverError: true)     │
                                              │  Fetch Statistics  → Fetch Events → Fetch Players    │
                                              │  Fetch Lineups → Fetch Injuries                      │
                                              │  Fetch Team 1 Form → Fetch Team 2 Form               │
                                              └─────────────────────────────────────────────────────┘
                                                                        │
                                                                        ▼
                                              Calculate Team Form → Calculate Ratings & MOTM
                                                                        │
                                                                        ▼
                                              Build Raw Timeline → Fetch H2H Meetings → Calculate H2H Record
                                                                        │
                                                                        ▼
                                              AI Tactical Analysis Agent (Gemini 2.5 Flash)
                                                                        │
                                                                        ▼
                                              AI Match Timeline Agent (Gemini 2.5 Flash)
                                                                        │
                                                                        ▼
                                              Assemble Final Response → Respond Match Analysis (200)
```

#### Key Node Details

| Node | Type | What it does |
|------|------|-------------|
| `Normalize Input` | Set | Handles both `$json.body.team1` and `$json.team1` request shapes |
| `Validate Input` | Code | Checks non-empty, differ, `YYYY-MM-DD` regex. Returns `isValid` + `errors[]` |
| `Is Input Valid?` | IF | `looseTypeValidation: true` — coerces string to boolean |
| `Search Team 1/2` | HTTP | `GET /teams?search={name}` against API-Football |
| `Resolve Team IDs` | Code | Exact case-insensitive match first, falls back to first result |
| `Fetch Fixtures By Date` | HTTP | `GET /fixtures?date={YYYY-MM-DD}` — returns ALL global fixtures that day |
| `Find Fixture` | Code | Filters by team IDs (either order) AND status in `['FT','AET','PEN']` |
| `Fetch Statistics` | HTTP | `GET /fixtures/statistics?fixture={id}` — `neverError: true` |
| `Fetch Events` | HTTP | `GET /fixtures/events?fixture={id}` — `neverError: true` |
| `Fetch Players` | HTTP | `GET /fixtures/players?fixture={id}` — `neverError: true` |
| `Fetch Lineups` | HTTP | `GET /fixtures/lineups?fixture={id}` — `neverError: true` |
| `Fetch Injuries` | HTTP | `GET /injuries?fixture={id}` — `neverError: true` |
| `Fetch Team 1/2 Form` | HTTP | `GET /fixtures?team={id}&last=5` — `neverError: true` |
| `Calculate Team Form` | Code | W/D/L per match from each team's perspective. Points accumulated |
| `Calculate Ratings & MOTM` | Code | Sorts by `rating → goals → assists → keyPasses` DESC per position (G/D/M/F). Overall MOTM across all positions |
| `Build Raw Timeline` | Code | Maps raw events to `{minute, type, detail, team, player, assist}` |
| `Fetch H2H Meetings` | HTTP | `GET /fixtures/headtohead?h2h={id1}-{id2}` — `neverError: true` |
| `Calculate H2H Record` | Code | Filter FT/AET/PEN → sort desc → cap at 10 → count wins/draws from team1 perspective. Spreads `...ctx` to carry all pipeline data forward |
| `AI Tactical Analysis Agent` | AI Agent | Gemini 2.5 Flash. Schema: `{tacticalDifferences, keyTurningPoints, impactOfAbsences, bestPerformers}`. Max 250 words |
| `AI Match Timeline Agent` | AI Agent | Gemini 2.5 Flash. Schema: `{timeline: [{minute, type, event}]}`. One sentence per event |
| `Assemble Final Response` | Code | Merges all node outputs into the final JSON shape |
| `Respond Match Analysis` | Respond | HTTP 200 |

> 🔴 **Error paths:**
> - Invalid input → `Respond Validation Error` (HTTP 400)
> - Teams not resolved → `Respond Team Resolution Error` (HTTP 404)
> - Fixture not found → `Respond Fixture Not Found` (HTTP 404)

---

### Workflow 2 — Head-to-Head (14 nodes)

**Fully deterministic. No AI.**

Shares the same input validation and team resolution pattern as Workflow 1, then:

```
Fetch H2H Meetings → Calculate H2H Record → Respond Head To Head (200)
```

**`Calculate H2H Record` logic:**
```javascript
// 1. Filter completed matches only
const completed = all.filter(f => ['FT','AET','PEN'].includes(f.fixture.status.short));

// 2. Sort newest first
completed.sort((a, b) => new Date(b.fixture.date) - new Date(a.fixture.date));

// 3. Cap at 10
const limited = completed.length >= 10 ? completed.slice(0, 10) : completed;

// 4. Count from team1's perspective
for (const f of limited) {
  const team1IsHome = f.teams.home.id === team1Id;
  if (homeGoals === awayGoals) draws++;
  else if ((team1IsHome && homeGoals > awayGoals) || (!team1IsHome && awayGoals > homeGoals)) wins1++;
  else wins2++;
}
```

> **Key difference from the `Calculate H2H Record` node inside Workflow 1:**
> In Workflow 1, this node is a data enrichment step that spreads `...ctx` (all accumulated pipeline data) forward. The output is embedded inside a 20+ field match analysis response.
> In Workflow 2, this IS the entire response. No accumulated context, no AI, no fixture lookup.

---

### Workflow 3 — Player Comparison (28 nodes)

**Match-contextual. Requires `fixture_id`.**

```
Webhook → Validate Input (fixture_id required) → Fetch Fixture Players
       → Build Match-Based Metrics → AI Comparison Agent → Assemble Response (200)
```

**Squad validation logic:**
```javascript
// Fuzzy match — case-insensitive substring
const found = allPlayers.find(p =>
  p.player.name.toLowerCase().includes(inputName.toLowerCase())
);

// If not found → HTTP 404
// "X is not in either squad for this match"
```

**11 stats extracted per player:**
`minutesPlayed` · `rating` · `goals` · `assists` · `shotsTotal` · `shotsOnTarget` · `passesTotal` · `keyPasses` · `passAccuracy` · `tackles` · `duelsWon`

**AI Comparison Agent:** Gemini 2.5 Flash · max 512 tokens · schema `{comparison: string}` · max 3 sentences · uses only supplied stats

**Frontend behavior:**
- Compare button starts **disabled**
- After a successful Workflow 1 response, `window._currentFixtureId` is set automatically
- Button enables. `fixture_id` is passed invisibly with every Compare request — the user only types player names

---

### Error Detection Workflow (4 nodes)

**Monitors all 3 main workflows 24/7.**

Registered as the "Error Workflow" inside each main workflow's Settings.

```
Error Trigger → Build Email Content → Send Error Email (Gmail OAuth2)
```

| Node | What it does |
|------|-------------|
| `Error Trigger` | Fires automatically when any registered workflow crashes. Supplies `workflow.name`, `execution.id`, `error.message`, `error.node.name` |
| `Build Email Content` | Extracts all error fields. Detects workflow number from name. Computes Cairo time (`Africa/Cairo`). Builds HTML email |
| `Send Error Email` | Gmail OAuth2 → `omarbusiness10000@gmail.com`. Subject: `"🚨 Error in {WorkflowName} — Node: {FailedNode}"` |

**Email includes:**
- Workflow name + number (1, 2, or 3)
- Exact failed node name
- Full error message
- Execution ID
- Cairo local timestamp
- Clickable link directly to the n8n execution

---

## AI Architecture

### Core Rule

> **AI agents NEVER fetch data, resolve IDs, count wins, calculate statistics, or filter records.**
> All of that happens in HTTP Request, Code, and IF nodes.
> AI receives pre-computed structured data and generates natural-language text only.
> This prevents hallucination at the architectural level.

### Agent Specifications

| Agent | Workflow | Model | Max Tokens | Output Schema |
|-------|----------|-------|------------|---------------|
| Tactical Analysis | 1 | gemini-2.5-flash | 1,024 | `{tacticalDifferences, keyTurningPoints, impactOfAbsences, bestPerformers}` |
| Timeline Narration | 1 | gemini-2.5-flash | 1,024 | `{timeline: [{minute, type, event}]}` |
| Player Comparison | 3 | gemini-2.5-flash | 512 | `{comparison: string}` |

### System Message Design — 3 Constraints Per Agent

1. **Data grounding:** `"Use ONLY the supplied data. Do not invent any statistic not present in the data."`
2. **Length constraint:** Tactical = 250 words max. Comparison = 3 sentences max.
3. **JSON enforcement:** `"Return ONLY raw JSON. No markdown code fences. No backticks. No text before or after the JSON object."` *(Added to prevent Gemini's default behavior of wrapping JSON in triple backticks, which breaks the Structured Output Parser)*

### Structured Output Parser

Each agent uses `outputParserStructured` (v1.3) with `schemaType: fromJson`. If the model's output doesn't match the schema, n8n raises a parse error which is caught by the Error Detection workflow.

---

## Frontend Integration

### Webhook Constants

```javascript
const WEBHOOK_URL         = 'https://omar7055.app.n8n.cloud/webhook/match-analysis';
const H2H_WEBHOOK_URL     = 'https://omar7055.app.n8n.cloud/webhook/head-to-head';
const COMPARE_WEBHOOK_URL = 'https://omar7055.app.n8n.cloud/webhook/player-comparison';
```

### Match Analysis Call

```javascript
const res = await fetch(WEBHOOK_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ team1, team2, matchDate: date })
});
const data = await res.json();
if (data.success === false) throw new Error((data.errors || []).join(' '));

// Store fixture ID for Player Comparison — user never sees this
window._currentFixtureId = data.fixture?.id || null;
document.getElementById('compareBtn').disabled = !window._currentFixtureId;
```

### H2H Auto-Trigger (900ms Debounce)

```javascript
let h2hDebounce = null;

function onTeamInput() {
  clearTimeout(h2hDebounce);
  h2hDebounce = setTimeout(triggerH2H, 900);
}

async function triggerH2H() {
  const t1 = document.getElementById('team1').value.trim();
  const t2 = document.getElementById('team2').value.trim();
  if (!t1 || !t2 || t1.toLowerCase() === t2.toLowerCase()) return;
  const res = await fetch(H2H_WEBHOOK_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ team1: t1, team2: t2 })
  });
  const data = await res.json();
  renderH2H(data, t1, t2);
}
```

### Player Comparison — Invisible `fixture_id`

```javascript
async function runCompare() {
  if (!window._currentFixtureId) return; // guard — button is disabled anyway

  const res = await fetch(COMPARE_WEBHOOK_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      player1: document.getElementById('cmpPlayer1').value.trim(),
      player2: document.getElementById('cmpPlayer2').value.trim(),
      fixture_id: window._currentFixtureId  // ← invisible to user
    })
  });
  const data = await res.json();
  if (data.success === false) throw new Error((data.errors || []).join(' '));
  renderComparisonReal(data);
}
```

---

## Credential Setup

### Required Credentials

| Name in n8n | Type | Used In | Key Fields |
|-------------|------|---------|------------|
| `API-Football` | Header Auth | WF 1, 2, 3 | Name: `x-apisports-key` · Value: key from [dashboard.api-football.com](https://dashboard.api-football.com) |
| `Google Gemini` | Google Gemini(PaLM) Api | WF 1, 3 | API Key from [aistudio.google.com](https://aistudio.google.com) |
| `Gmail OAuth` | Gmail OAuth2 | Error Detection | Google account sign-in |

> ⚠️ **API-Football vs RapidAPI**
> Use the **direct dashboard key** with `x-apisports-key` header at `v3.football.api-sports.io`.
> Do NOT use a RapidAPI key — it uses different headers and a different base URL.

### Node-to-Credential Map

| Credential | Nodes |
|-----------|-------|
| `API-Football` | Search Team 1, Search Team 2, Fetch Fixtures By Date, Fetch Statistics, Fetch Events, Fetch Players, Fetch Lineups, Fetch Injuries, Fetch Team 1 Form, Fetch Team 2 Form, Fetch H2H Meetings, Fetch Fixture Players |
| `Google Gemini` | Gemini Model - Tactical, Google Gemini Chat Model (WF1), Google Gemini Chat Model (WF3), Google Gemini Chat Model1 (WF3) |
| `Gmail OAuth` | Send Error Email |

---

## Deployment

### Backend (n8n Cloud)

1. Create account at [n8n.io](https://n8n.io)
2. Create all 3 credentials in the Credentials panel (exact names matter)
3. Build or import all 4 workflows
4. **Critical:** On every IF node across all 3 main workflows, enable **"Convert types where required"** toggle
   > Without this, workflows fail at runtime with: `"Wrong type: '' is a string but was expecting a boolean"`
   > There are **11 IF nodes** across the 3 main workflows that all require this toggle
5. In each main workflow's Settings (`⋯ → Settings`), set **Error Workflow** → `Error Detection — Match Analyst`
6. Publish all 4 workflows

### Frontend (Netlify)

1. Update the 3 webhook URL constants in `index.html` to match your n8n instance
2. Go to [netlify.com](https://netlify.com) → drag and drop `index.html`
3. Site is live in ~10 seconds

### Live URLs

| Resource | URL |
|---------|-----|
| Frontend | https://ai-match-analyst.netlify.app/ |
| n8n Dashboard | https://omar7055.app.n8n.cloud |
| Webhook — Match Analysis | https://omar7055.app.n8n.cloud/webhook/match-analysis |
| Webhook — Head-to-Head | https://omar7055.app.n8n.cloud/webhook/head-to-head |
| Webhook — Player Comparison | https://omar7055.app.n8n.cloud/webhook/player-comparison |

---

## Error Handling

### Workflow-Level Strategy

- Critical IF nodes route failures to dedicated `Respond to Webhook` error nodes (400/404)
- Secondary data nodes (`Fetch Statistics`, `Fetch Events`, etc.) use `neverError: true` + `continueRegularOutput` — a failed secondary call never crashes the workflow

### Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `No completed fixture found between X and Y on Z` | API-Football free plan date restriction (~2–3 day rolling window) | Use a date within the allowed window or upgrade plan |
| `The service is receiving too many requests` | 100 req/day free tier limit hit | Wait for midnight UTC quota reset |
| `Could not resolve team1: X` | No team matched the search term | Check spelling or try a shorter name |
| `"X" is not in either squad for this match` | Player not found in the fixture's squad list | Use a player who appeared in the analyzed match |
| `fixture_id is required` | Player Comparison called without first analyzing a match | Analyze a match first |
| `Model output doesn't fit required format` | Gemini wrapped JSON in markdown code fences | System message enforces raw JSON — retry usually resolves |

---

## Known Limitations

| Limitation | Cause | Resolution |
|-----------|-------|------------|
| Date window ~2–3 days | API-Football free plan restriction on `/fixtures?date=` | Upgrade to paid plan |
| 100 API requests/day | Free tier quota. WF1 uses ~11 per execution (~9 analyses/day) | Upgrade to paid plan |
| Season player stats blocked | `/players?season=` restricted on free plan | Upgrade to paid plan |
| Squad-only player comparison | By design — ensures fixture data is available | Not a bug |
| No live match support | Platform only handles FT/AET/PEN status | Add a live-match path |

---

## Testing

### Test Commands (PowerShell)

```powershell
# Match Analysis
Invoke-RestMethod `
  -Uri "https://omar7055.app.n8n.cloud/webhook/match-analysis" `
  -Method Post -ContentType "application/json" `
  -Body '{"team1":"Canada","team2":"Qatar","matchDate":"2026-06-18"}'

# Head-to-Head
Invoke-RestMethod `
  -Uri "https://omar7055.app.n8n.cloud/webhook/head-to-head" `
  -Method Post -ContentType "application/json" `
  -Body '{"team1":"Canada","team2":"Qatar"}'

# Player Comparison
Invoke-RestMethod `
  -Uri "https://omar7055.app.n8n.cloud/webhook/player-comparison" `
  -Method Post -ContentType "application/json" `
  -Body '{"player1":"Jonathan David","player2":"Cyle Larin","fixture_id":1035037}'
```

### Test Commands (cURL)

```bash
# Match Analysis
curl -X POST https://omar7055.app.n8n.cloud/webhook/match-analysis \
  -H "Content-Type: application/json" \
  -d '{"team1":"Canada","team2":"Qatar","matchDate":"2026-06-18"}'
```

### Testing Strategy

| Level | Method |
|-------|--------|
| **Pin Data Testing** | Injected simulated match data (France 2–1 Brazil) into all HTTP nodes in n8n. Tested full pipeline including AI agents without consuming API quota |
| **Production Live Testing** | Real requests sent to production webhooks. Canada 6–0 Qatar (World Cup, June 18 2026) used as primary test fixture |
| **Error Case Testing** | Invalid inputs, out-of-window dates, off-squad players tested across all error paths |

### Testing the Error Detection Workflow

1. Temporarily set the API-Football credential value to something invalid (e.g. `invalid_key_test`)
2. Send any request to Workflow 1
3. The HTTP node crashes → Error Detection fires → email arrives within ~30 seconds
4. Restore the real credential


---

<div align="center">

Built with ⚽ and ☁️ — zero custom servers harmed in the making of this project.

**[Live Site](https://ai-match-analyst.netlify.app) · [n8n Backend](https://omar7055.app.n8n.cloud)**

</div>
