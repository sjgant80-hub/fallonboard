# ◊ FallOnboard

**Sovereign FCA-shaped client onboarding for UK financial-advisory firms.**
KYC · AML · source-of-funds · vulnerable-customer (FG21/1) · document capture · audit chain.

One HTML file. Runs from `file://`. No server. Client data never leaves the device.

> **Disclaimer.** FallOnboard supports FCA-shaped client onboarding for UK financial-advisory firms. It is a tool, not a compliance audit; final responsibility rests with the firm's compliance officer / MLRO.

---

## For end users (advisers, paraplanners, compliance)

### What it does

Onboards a new client across **9 wizard steps**, capturing everything FCA / JMLSG / MLR 2017 expects from an IFA's CDD/EDD file:

1. **Identity** — title, names, DOB, nationality, NINO, UTR, tax residency
2. **Contact** — email, phone, current address, address history (last 3 yrs for AML)
3. **Relationships** — link to existing clients (spouse, partner, dependants, POA)
4. **KYC core** — PEP screening + sanctions status with deep-link to OFSI consolidated list
5. **Source of funds + source of wealth** — structured + notes
6. **Vulnerable customer assessment** — FCA FG21/1 four-driver framework
7. **Document capture** — drag-drop or pick. Files stored as Blobs in IndexedDB. SHA-256 hash on capture. Expiry computed per type.
8. **AML risk grade** — auto-suggestion (PEP / sanctions / jurisdiction / source) + manual override
9. **Confirm & commit** — writes to IDB, broadcasts on the bundle mesh, appends audit chain entry.

Beyond onboarding:

- **Clients list** — sortable, with filter chips by KYC status, risk grade, adviser, and overdue review
- **Dashboard** — portfolio KPIs · PEPs · sanctions reviews · vulnerable count · clients due for refresh
- **Firm setup** — legal name, FCA ref, Companies House, PI insurance, professional body
- **Advisers** — SM&CR roles (SMF1 / 17 / 22 / etc.), FCA refs
- **Client detail** — edit any field, re-trigger sanctions, add documents, generate KYC certificate (Markdown), soft-archive
- **AML help (Q&A)** — 12 hardcoded T0 rules (PEP, JMLSG, POCA, SAR, Consumer Duty, COBS, FG21/1, retention…). Optional BYOK Claude / GPT / Gemini / OpenRouter for nuanced T3 grounded in your firm's context.
- **Audit chain** — every write hashed + chained (prevHash · docHash · reasoning · adviserId · configVersion). Exportable for compliance review.
- **Cross-tool mesh** — opens `BroadcastChannel('fall-client')`; sync.request on boot; any client/adviser/firm write broadcasts the full record. Sibling tools (FallAdviser v2, FallPaper, FallPractice) merge by `updatedAt`.
- **Backup** — full JSON export, optionally AES-GCM-256 encrypted with PBKDF2-derived key. Import merges.

### Getting started

1. Open `index.html` in a modern browser (Chrome 113+, Edge, Firefox, Safari 16.4+).
2. First run shows a **DEMO · Marcus Osei · overwrite me** client so the UI isn't empty.
3. Click **Firm** → fill in your firm record.
4. Click **Advisers** → add at least one adviser.
5. Click **+ client** to launch the onboarding wizard.
6. The first real client onboarded automatically purges the demo records.

### Add-to-Home-Screen

The PWA manifest is baked via a `data:` URL. On Chrome / Edge: address bar → install. On iOS Safari: share → add to Home Screen. Standalone app, no server required.

### Privacy / sovereignty

- All data is stored in your browser's IndexedDB (`fallonboard.v1`).
- Document files are stored as Blobs in IDB — never uploaded.
- localStorage holds a mirror of state for fast recovery.
- Audit chain is append-only; entries cap at 50,000 (rough 5,000 client-years for a 1-10 person firm).
- Optional BYOK API keys for T3 are stored locally and only sent to the provider you choose when you ask a question.
- Wiping the device (Settings → danger zone) is permanent.

### FCA / regulatory framing

| Reference | What FallOnboard captures |
|---|---|
| MLR 2017 reg.28 (CDD) | Identity, address, beneficial ownership (relationships), purpose |
| MLR 2017 reg.33-36 (EDD) | PEP, source of funds **and** wealth, enhanced documentation |
| MLR 2017 reg.35 (PEPs) | Explicit PEP flag + details |
| MLR 2017 reg.40 (retention) | 5+ year retention assumed; audit chain capped at 50k entries (~7 yrs) |
| FCA FG21/1 | Vulnerable customer flag + 4 categories (health / life-event / resilience / capability) |
| FCA PRIN 2A (Consumer Duty) | Vulnerability lens, ongoing review cadence by risk grade |
| FCA SYSC 9.1.1R | Audit chain (prevHash + docHash + reasoning + configVersion) |
| COBS 9 (suitability) | ATR / capacity for loss / knowledge / horizon captured per shared schema |
| JMLSG Part I §5 | Two-source CDD evidence guidance (in T0 AML help) |
| POCA 2002 s.330 / s.333A | SAR / tipping-off guidance (in T0 AML help) |

### IFA bundle siblings

FallOnboard is one of four tools in the IFA suite. They share the same client schema via `BroadcastChannel('fall-client')`:

| Tool | Prime | Role |
|---|---|---|
| **falladviser** v2 | 719 | Multi-client tax / pension / portfolio planning |
| **fallonboard** | **727** | KYC / AML capture (this) |
| **fallpaper** | 733 | FCA-shaped document generator (suitability reports, ToB, etc.) |
| **fallpractice** | 739 | Adviser fee ledger + CASS |

Open any two side by side — they keep each other in sync.

---

## For developers

### Architecture

Single-file vanilla JS. No build step. No framework. No dependencies (Google Fonts CDN is graceful — system fonts fall back). Approximately ~2400 lines, ~85KB.

```
┌──────────── index.html ────────────┐
│  HEAD                              │
│   · manifest (data: URL)           │
│   · style (oxblood / brass / cream │
│     / void, ~3KB inline CSS)       │
│  BODY                              │
│   · <header> brand + nav + tools   │
│   · <main id="view">               │
│   · #modalRoot, #toast             │
│  SCRIPT (vanilla, ~2200 LOC)       │
│   · constants                      │
│   · state (single object)          │
│   · util / IDB / audit             │
│   · BroadcastChannel mesh          │
│   · factories                      │
│   · AML risk scoring               │
│   · CRUD (audited + broadcast)     │
│   · demo seed                      │
│   · T0/T3 cascade + 12 rules       │
│   · render() router                │
│   · views: clients / dash / firm / │
│     advisers / qa / wizard / detail│
│   · modals: settings / audit       │
│   · postMessage API                │
│   · KONOMI shim                    │
│   · boot                           │
└────────────────────────────────────┘
```

### IndexedDB schema

Database: `fallonboard.v1` (version 1). Object stores:

| Store | KeyPath | Contents |
|---|---|---|
| `state` | (out-of-line, key `'main'`) | The single state snapshot: `firm`, `advisers[]`, `clients[]`, `chat[]`, `settings`, `active`, `schemaVersion` |
| `audit` | `i` (sequential integer) | Audit chain entries: `{i, ts, tool, adviserId, clientId, action, reasoning, configVersion, prevHash, docHash, payload}` |
| `documents` | `id` | Document blobs: `{id, clientId, filename, type, mime, size, sha256, capturedAt, note, blob}` |

A localStorage mirror of `state` is kept under `fallonboard.v1.state` as a fallback if IDB is unavailable (private browsing on some browsers, etc.).

### Shared client schema

Conforms exactly to `IFA-BUNDLE-SHARED-SCHEMA.md` v1.0. The `Client`, `Adviser`, `Firm` records have the field shapes other bundle tools expect. Cross-tool sync is broadcast-based; merge strategy is "later `updatedAt` wins" on the `id` keypath.

### BroadcastChannel mesh

```js
// fall-client : the bundle data plane
{ v:1, type, ts, source:'fallonboard', payload }
```

Types emitted by FallOnboard:

- `client.created` · `client.updated` · `client.archived`
- `adviser.created` · `adviser.updated` · `adviser.archived`
- `firm.updated`
- `sync.request` (on boot) · `sync.snapshot` (in response to others)

Receiver discipline: replace the local record if `payload.updatedAt > local.updatedAt`. Re-render.

Debounce: 300ms per `type|id` key on outgoing messages (rapid edits aren't broadcast every keystroke).

```js
// fall-signal : the wider estate plane
{ source:'fallonboard', type:'hello'|'ping'|'pong', prime:727, version, ts }
```

### Audit chain · hash discipline

```js
docHash = sha256( prevHash || ts || action || clientId || payload )
prevHash = previous entry's docHash
```

Entry shape per `IFA-BUNDLE-SHARED-SCHEMA.md`:

```js
{ i, ts, tool, adviserId, clientId, action, reasoning,
  configVersion: 'fallonboard@1.0.0', prevHash, docHash, payload }
```

External audit: replay the chain, recompute each `docHash`, compare. Any mismatch invalidates that entry and all downstream entries.

### Cascade · T0 → T3

```
T0 (offline)  → 12 hardcoded rules (PEP, JMLSG, POCA, FG21/1, COBS, retention, …)
T3 (BYOK)     → Anthropic / Gemini / OpenAI / OpenRouter
```

`Cascade.detectTier()` returns `'T0'` or `'T3'` based on which keys exist in settings. `answer(q)` tries T0 first (regex match), then T3 with firm context as system prompt.

### postMessage API (for si-didy / sibling tools)

```js
parent.postMessage({target:'fallonboard', action:'get-clients'}, '*');
// → {target:'…', responseTo:'get-clients', data:{ok:true, clients:[...]}}

// Actions:
// 'ping' | 'get-clients' | 'get-advisers' | 'get-firm' | 'ask' (+question)
```

### Console hooks

```js
window.FALLONBOARD = { version, prime, state, answer, Cascade }
window.KONOMI      = { active, tier:'sovereign', prime, check }
```

### File layout

```
fallonboard/
├── index.html        ← the deliverable
├── README.md         ← this file
├── LICENSE           ← MIT
└── .nojekyll         ← tells GitHub Pages to skip Jekyll
```

### Deploy (GitHub Pages, estate pattern)

```bash
git init
git add -A
git commit -m "ship fallonboard v1.0.0"
git branch -M main
git remote add origin git@github.com:sjgant80-hub/fallonboard.git
git push -u origin main
# Then: Settings → Pages → Source: legacy · branch=main · path=/
```

Live URL: `https://sjgant80-hub.github.io/fallonboard/`

### 14-pt gate compliance

| # | Check | Status |
|---|---|---|
| 1 | Single HTML file · works from `file://` | ✓ |
| 2 | <400KB target | ✓ (~85KB) |
| 3 | L1 FACE — domain-appropriate views (KYC wizard, not generic CRM) | ✓ |
| 4 | L2 SWARM — Ω + named functions (Cascade, Audit, Mesh, Risk) | ✓ |
| 5 | L3 CASCADE — T0 offline · T3 optional | ✓ |
| 6 | L4 BLOOM — no LLM-required paths | ✓ |
| 7 | L5 PERSIST — IDB primary · localStorage fallback · JSON export/import | ✓ |
| 8 | L6 SKIN — mobile-first · oxblood/brass/cream/void · serif headings | ✓ |
| 9 | L7 ASS — demo client seeded so first run isn't blank | ✓ |
| 10 | KONOMI sovereign shim baked | ✓ |
| 11 | fallmesh — fall-signal + fall-client BroadcastChannel | ✓ |
| 12 | PWA manifest via data: URL · installable | ✓ |
| 13 | Two-audience README · MIT LICENSE | ✓ |
| 14 | Push + Pages live (verify after push) | (you, on push) |

### Mansoor P3 hardening (client-data tools)

| | Check |
|---|---|
| P1 | Dashboard reads same `state.clients` as wizard writes — no derived cache |
| P2 | Saves are event-driven (form submit → save → broadcast); no polling |
| P3 | Audit chain on by default, prevHash + reasoning + docHash + configVersion |
| P4 | Surface tailored: 9-step KYC wizard, not generic "add record" |
| P5 | JSON export/import + optional AES-GCM passphrase + BroadcastChannel sync |
| P6 | No KCC paywall — sovereign tier baked, professional context |

### Modifying

Just edit `index.html` and reload. There is no build step.

To add a T0 rule, push into `T0_RULES`:

```js
T0_RULES.push({
  match: /your regex/i,
  title: 'Short title for the chip',
  answer: () => `**Markdown answer** with citations.`
});
```

To bump the schema, increment `SCHEMA_V` and add a migration path inside `loadState`.

### Browser support

- Chrome 113+ (primary target — full IDB, BroadcastChannel, SubtleCrypto)
- Edge 113+ (same engine)
- Firefox 115+
- Safari 16.4+
- iOS Safari / Android Chrome — fully functional, drag-drop replaced by tap-to-pick

### License

MIT — see `LICENSE`.

---

`◊ fallonboard@1.0.0 · prime 727 · schema v1 · sovereign`
