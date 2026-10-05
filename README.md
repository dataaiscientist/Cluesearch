# Clue Search Lab

**Find a photo from the parts you're sure of.**

Clue Search Lab is a working prototype for one problem: people know a photo exists in their library, but they can't recall *when*, *where*, *which album* or *the exact words* to search for. Instead of treating every word as required, Clue Search turns a half-remembered description into separate **clues**, each with its own **confidence** (sure · fairly · guess). It ranks photos on the evidence and shows exactly which clues each result matches.

> "Arka swimming with a blue kickboard, his first lap without armbands. The swim coach sent it on WhatsApp, I think 2019."
> → **Who** Arka (sure) · **What** swimming, blue kickboard (sure) · **Source** WhatsApp (sure) · **When** 2019 (guess)

<p align="center">
  <img src="docs/img/desktop.png" alt="Clue Search Lab on desktop: scenario strip, search box, clue board" width="68%">
  <img src="docs/img/mobile.png" alt="Clue Search Lab on a phone: single column with bottom tab bar" width="26%">
</p>

---

## Why it exists

| Memory people have | What standard search does | What Clue Search does |
|---|---|---|
| Who, what and a visual detail, held with confidence | Treats them the same as everything else | Weights them highest (*sure*) |
| The year or month, often wrong | Treats the date as a hard filter | Treats it as a *guess* that bends |
| WhatsApp copies, scans, restored phones | Uses the file date, which is the import date | Reads source (WhatsApp / scan / restored) as its own clue |
| Mixed English, Indonesian, Hindi | Misses non-English words | Runs an ID + HI lexicon before parsing |
| One wrong detail | Returns an empty screen | Loosens the least-supported clue, names it, offers Undo |

## Results (v2, offline quick reader, 56-item sample library)

| Benchmark | n | v1 → v2: right photo #1 | Empty screens v2 | Standard search empty |
|---|---|---|---|---|
| Set A (tuning) | 33 | 21 → **33** | **0** | 27 |
| Set B (blind, written before v2 ran) | 22 | 11 → **22** | **0** | 20 |
| Indonesian / code-mix (A+B) | 22 | 1 → **22** | **0** | — |

The benchmark runs live in the **Launch** tab every time the page opens. Please read the limits at the bottom before you quote these numbers.

---

## Quick start

It is a single static file with no build step and no dependencies to install.

```bash
git clone https://github.com/<your-username>/clue-search-lab.git
cd clue-search-lab
python3 -m http.server 8000      # or: npx serve .
# open http://localhost:8000
```

Opening `index.html` directly in a browser also works.

### Publish on GitHub Pages
1. Push this repo to GitHub on the `main` branch.
2. Go to **Settings → Pages → Build and deployment → Source: GitHub Actions**.
3. The included workflow (`.github/workflows/pages.yml`) deploys on every push to `main`. The site appears at `https://<your-username>.github.io/clue-search-lab/`.

---

## Two run modes

```mermaid
flowchart LR
    U[User memory<br/>free text, EN/ID/HI] --> L[Lexicon<br/>ID + HI → EN]
    L --> R[Quick reader<br/>rule parser]
    R --> G[Grounding<br/>vs library vocabulary]
    G --> S[Scorer<br/>confidence × type weights]
    S --> X{Zero results?}
    X -- yes --> RL[Auto-loosen least-supported clue<br/>+ Undo]
    RL --> S
    X -- no --> V[Ranked tiles with ✓ ≈ ✗ evidence<br/>+ recovery banner]
    U -. on claude.ai only .-> AI[Claude reader<br/>JSON clues] -.-> G
```

| Capability | GitHub Pages / local | Opened as a claude.ai artifact (signed in) |
|---|---|---|
| Sample family library (56 items, 8 years) and all scenarios T1–T5, H1–H5 | ✅ | ✅ |
| Quick reader (rule parser, lexicon, grounding, auto-loosen) | ✅ | ✅ |
| Live regression benchmark (Launch tab) | ✅ | ✅ |
| Claude reader (LLM clue extraction, replaces the quick reader when ready) | — | ✅ |
| **My photos**: describe your own 10–60 photos | — (needs Claude) | ✅ |
| Shared research log across testers | — (kept on this device for the visit) | ✅ |
| CSV / JSON export of attempts | ✅ | ✅ |

The page detects `window.claude` and falls back cleanly when it is absent, so nothing breaks on GitHub Pages. You only lose the AI-powered parts. See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for how to connect your own model backend.

---

## Repository layout

```
clue-search-lab/
├── index.html                  # the whole app: HTML + CSS + JS, sample library, engine, benchmark
├── docs/
│   ├── ARCHITECTURE.md         # engine, scoring, data model, extension points
│   ├── PRIVACY.md              # what data goes where
│   └── img/                    # screenshots used in this README
├── .github/workflows/pages.yml # GitHub Pages deploy
├── CHANGELOG.md
├── LICENSE                     # MIT
├── .nojekyll
└── .gitignore
```

## Tabs

- **Search**: scenario strip, memory card, search box, clue board (tap a clue to change its confidence, × to drop it), ranked results with ✓ ≈ ✗ evidence, a detail sheet with name-a-face.
- **My photos**: drop 10–60 of your own JPEG/PNG/WebP photos. They are described by Claude, and dates come from EXIF via [exifr](https://github.com/MikeKovarik/exifr).
- **Results**: every search attempt logged (found / time / edits / gave up), research mode toggle, CSV/JSON export.
- **Launch**: launch gates, the v1→v2 changes, the live regression and the rollout plan. Open it directly with `#launch`.

## Keyboard & accessibility
- `Ctrl/Cmd + Z` undoes the last clue change.
- Marks are readable without colour (✓ ≈ ✗ symbols and dashed borders for guesses), touch targets are 44 px, and the page supports light and dark themes.

## Limits (read before quoting numbers)
- The library is a synthetic 56-item family. All 55 benchmark descriptions come from one writer.
- Part 6 usability testers were AI personas, so real-user validation is still pending (Stage 0).
- "Standard search" is an approximation, not any production ranking.
- Exact-word grounding does not scale. Production would need embedding-threshold grounding (target p95 < 1.5 s).

## Roadmap
- [ ] Stage 0: 5 real participants in research mode (found ≥ 4/5, median < 60 s)
- [ ] Market pull: ≥ 40 % "very disappointed" (Sean Ellis), ≥ 40 answers
- [ ] Optional backend adapter so the Claude reader and My photos work outside claude.ai
- [ ] Embedding-based grounding for 5,000+ item libraries

## License
[MIT](LICENSE) © 2026 Fahmi Adam, MBA

---

**Author:** Fahmi Adam, MBA
