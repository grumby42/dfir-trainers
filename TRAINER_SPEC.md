# TRAINER_SPEC — the DFIR Trainers build contract

Every trainer in this repo meets this spec. The **Squidoor** trainer
(`squidoor/`) is the reference implementation; read it alongside this document.
The shell is reusable — the per-case work is the content and, above all, the
fact-check.

If a trainer can't meet a **MUST** here, it doesn't ship.

---

## A. Template contract (structure)

Every trainer MUST contain:

- **One self-contained offline HTML file** (`index.html`): vanilla JS data arrays
  + render functions. No external dependencies, no network calls, no telemetry.
  It must run from `file://`.
- **Three modes:**
  1. **Case walkthrough** — mission brief → eradication, with a per-step
     reasoning layer: **Observe → Orient → Relevance** (is this related, and on
     what basis?) **→ Sufficiency** (do I have enough, or do I need more?).
  2. **Exercise generator** — randomized reps (distillation, relevance calls,
     sufficiency gates, behavioral inference, bias-spotting, SITREPs, scoping,
     pyramid-climbing, structured logging) grounded in real, current campaigns.
  3. **Case log** — a fixed-schema, fillable intelligence-logging form
     (enumerated fields + ATT&CK IDs + Diamond vertices) with JSON/TSV export, so
     different analysts' logs stay comparable.
- **Evolving framework meters** that light up as the investigation progresses:
  Cyber Kill Chain, NIST IR phase, PEAK, Pyramid of Pain, Diamond Model vertices.
- **Confirmation-bias thread** — bias checks at the moments bias actually bites,
  an **ACH** (Analysis of Competing Hypotheses) gate, and a bias watchlist. The
  rule throughout: *notice it, note it, seek disconfirming evidence — never lean
  on it.*
- **Defender ⇄ Attacker toggle** on every phase, framed as **plausible inference
  from observed behavior, not a confession.**
- **Hunt Lead persona** who tasks the learner, asks pointed questions, and demands
  SITREPs.
- **Build stamping:** a `BUILD_ID` constant of the form `HUNT-YYYYMMDD-HHMMZ` and
  a `STAMP_UTC` constant, both surfaced in-app and echoed in the trainer's README.
- **Keyboard navigation:** `←` / `→` through the walkthrough.
- **Build-a-range guide** (`RANGE.md`): a free, mostly-local lab that reproduces
  the case's artifacts hands-on, with honest fidelity tiers and a clear
  **emulation boundary** — no live malware/exploit, synthetic artifacts only.

---

## B. Fact-check rigor bar (non-negotiable — before shipping)

- **≥2 independent primary sources**, actually fetched and read during the build
  session — **not** recalled from model memory, not summarized second-hand.
  Primary = the vendor/researcher reports that first documented the case.
- **No hallucinated, fabricated, or guessed facts.** Every technical claim must be
  traceable to a cited source.
- **Four-way provenance labeling** on claims that aren't plain report facts:
  - *written-report fact* — stated in a primary written report;
  - *conference-talk anecdote* — from a talk/slide, not the written report
    (label it as talk-sourced);
  - *analyst inference* — reasoned from observed behavior (e.g. attacker-view);
  - *synthetic training data* — invented for the trainer (all console/evidence
    panels).
- **ATT&CK technique IDs verified** against the sources — never assumed from the
  technique name.
- **No fabricated IOCs.** In-app indicators are synthetic placeholders only:
  RFC 5737 documentation IPs (`192.0.2.0/24`, `198.51.100.0/24`,
  `203.0.113.0/24`), `example.com`/`.org` domains, and clearly-fake placeholder
  hashes. Point learners to the cited reports for real IOCs.
- **Adversary-review test:** *could a high-profile DFIR SME poke holes in this?*
  If yes, plug it before shipping.

### Pre-ship checklist (copy into the PR / commit notes)

- [ ] ≥2 primary sources fetched and read this session; URLs listed in the README
- [ ] Every technical claim traceable to a source
- [ ] Non-report claims labeled (talk-sourced / inference / synthetic)
- [ ] All ATT&CK IDs verified against sources
- [ ] Zero real IOCs in-app; all indicators synthetic and marked
- [ ] Attacker-view framed as inference, not confession
- [ ] Credits-first README + non-affiliation statement present
- [ ] `DETECTIONS.md` present (pseudocode, evolve-it, baseline/tune, trends)
- [ ] `BUILD_ID` + `STAMP_UTC` set and consistent between app and README
- [ ] JS validated (`node --check` on the extracted script) and opens from `file://`
- [ ] Adversary-review pass done

---

## C. Credit & licensing (every trainer)

- **Credits-first README** that names the researchers and teams and links their
  work, before anything else.
- **Explicit non-affiliation statement** ("not affiliated with, sponsored by, or
  endorsed by …").
- **MIT `LICENSE`**, `Copyright (c) <year> Theodore "Ted" Gransbury` (the repo
  LICENSE at root covers all trainers unless a trainer adds its own).
- **A `DETECTIONS.md` appendix per trainer**: starter detections as tool-agnostic
  **pseudocode**, with the real published rules to anchor against, then — for each
  — *how to evolve it, how to baseline and tune it, and the trend signals to
  watch* (spike / drop-off / first-seen / beacon). Teach authoring; don't ship
  copy-paste rules.
- **Git author:** `Ted Gransbury <tgransbury@gmail.com>`. End every commit with:

  ```
  Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_01GyXeSgVfAdLFb8Vz45dvE1
  ```

---

## D. Repo structure

```
dfir-trainers/
├── README.md          index: what each trainer is, who it's for, the standard
├── TRAINER_SPEC.md    this file
├── LICENSE            MIT, covers all trainers
├── .gitignore
├── _template/         how to start a new trainer from the Squidoor base
├── squidoor/          index.html · README.md · DETECTIONS.md   (reference impl)
├── toolshell/         (planned)
└── salt-typhoon/      (planned)
```

- One subfolder per trainer; folder name is a short case slug.
- Each trainer folder MUST have `index.html`, `README.md`, and `DETECTIONS.md`.
- GitHub Pages serves each trainer at `/<slug>/`.

---

*Spec generated 2026-10-16T01:36Z · reference build `HUNT-20261015-2250Z`.*
