# Squidoor DFIR Hunt & IR Trainer
*Endpoint intrusion — Squidoor / FinalDraft (CL-STA-0049): web-shell foothold → renamed-LOLBin loader → Graph/Outlook dead-drop C2.*

An interactive, **single-file, fully-offline** training tool that walks you through the
investigation of the **Squidoor / FinalDraft** backdoor (activity cluster **CL-STA-0049**,
a suspected Chinese espionage operation) from the first faint hunting signal all the way to
attribution, eradication, and structured intelligence logging.

Open `index.html` in any modern browser. No install, no dependencies, no network calls, no
telemetry. Everything runs locally.

---

## ⚠️ Credit where it's due

**This is an independent *educational reconstruction*. I did not discover, analyze, or
report on Squidoor.** All of the underlying research and every technical fact belongs to
the teams below, and this trainer exists only to help defenders learn from their work.

- **Palo Alto Networks — Unit 42** · *"Squidoor: Suspected Chinese Threat Actor's Backdoor
  Targets Global Organizations"* by **Lior Rochberger** and **Tom Fakterman** (27 Feb 2025),
  and their talk at the **SANS DFIR Summit 2026**.
  <https://unit42.paloaltonetworks.com/advanced-backdoor-squidoor/>
- **Elastic Security Labs** · *"You've Got Malware: FINALDRAFT Hides in Your Drafts"* by
  **Cyril François, Jia Yu Chan, Salim Bitam, and Daniel Stepanic** (13 Feb 2025), and the
  REF7707 campaign analysis *"From South America to Southeast Asia: The Fragile Web of
  REF7707."*
  <https://www.elastic.co/security-labs/finaldraft>

The frameworks the trainer teaches are likewise other people's work, credited in the tool
and here: the **Pyramid of Pain** and the **PEAK** framework (David Bianco; PEAK with Ryan
Fetterman / Splunk SURGe), the **Diamond Model of Intrusion Analysis** (Sergio Caltagirone,
Andrew Pendergast, Christopher Betz), **MITRE ATT&CK** and the **Center for Threat-Informed
Defense** (Summiting the Pyramid, Attack Flow), **HEARTH** (THOR Collective), **NIST SP
800-61r3**, **TaHiTI** (Dutch financial sector), and **Analysis of Competing Hypotheses**
(Richards J. Heuer Jr.).

**This project is not affiliated with, sponsored by, or endorsed by Unit 42, Elastic, SANS,
or any other party named here.**

---

## What it is

A reconstruction of a real hunt-to-IR workflow, built so you can *practice the reasoning*,
not just read about it. Three modes:

### 1. Case walkthrough
19 turns from mission brief to eradication, with:
- A **Defender ⇄ Attacker** view toggle on every phase — see the operator's likely
  reasoning for each move (framed as inference, not a confession) to practice anticipating
  the next step.
- A per-step **reasoning layer** — Observe → Orient → **Relevance** (is this related, and
  on what basis?) → **Sufficiency** (do I have enough, or need more?).
- **Evolving framework meters** — Cyber Kill Chain, NIST IR phase, PEAK, Pyramid of Pain,
  and the Diamond Model vertices lighting up as you discover them.
- A **confirmation-bias thread** — bias checks at the moments bias actually bites, an ACH
  (Analysis of Competing Hypotheses) gate, and a bias watchlist. The rule throughout:
  *notice it, note it, seek disconfirming evidence — never lean on it.*
- A **Hunt Lead persona** who tasks you, asks pointed questions, and demands SITREPs.
- Interactive decision points, a live confidence meter, and a collection-gap tracker.

### 2. Exercise generator
A "prompt book" that spins up randomized reps — distillation, relevance calls, sufficiency
gates, behavioral inference, bias-spotting, SITREPs, scoping, pyramid-climbing, and
structured logging — grounded in seven real, current campaigns (Squidoor, Salt Typhoon,
Scattered Spider, Akira/SonicWall, ToolShell/SharePoint, DPRK "Contagious Interview", and
ClickFix).

### 3. Case log
A fixed-schema, fillable intelligence-logging form (enumerated fields + ATT&CK IDs + Diamond
vertices) that structurally prevents the "everyone uses different columns" problem, with a
find-matches workflow ("I see X → who else did X?") and JSON/TSV export for carrying into
MISP / OpenCTI / a spreadsheet.

---

## Who is this for?

**Built for (the bullseye):**
- **Threat hunters** — the PEAK loop, hypothesis-driven hunting, distillation, pivoting, and
  livehunts are the daily craft here.
- **SOC analysts (Tier 1–3)**, especially those moving toward hunting/detection — triage,
  signal-vs-noise relevance calls, the sufficiency gate, and reporting up to a Hunt Lead.
- **Incident responders / DFIR analysts** — the full arc: order of volatility,
  containment-vs-scope, ACH, simultaneous eradication, and lessons-to-detections.
- **Detection engineers** — turning TTPs into durable, evasion-resistant detections
  (Pyramid-climbing, PE OriginalFileName, non-browser-to-Graph).
- **CTI analysts** — Diamond pivoting, calibrated attribution language, F3EAD, and a
  structured case log that maps to MISP / OpenCTI.

**Also useful to:**
- **SOC / hunt team leads** — the Hunt Lead persona and SITREP moments model tasking and
  receiving work; a ready-made onboarding and team-drill tool.
- **Red / purple teamers** — the Defender ⇄ Attacker lane shows how blue reasons about
  detection.
- **MSSP / MDR service delivery managers & consultants** — for understanding, QA-ing,
  scoping, and explaining analyst work to clients and teams.
- **Students, career-changers, and instructors** — it's pedagogical by design.

**Not really built for** executives/leadership as a learning tool — it's intentionally
in-the-weeds. (A one-time skim can still build literacy on why "are we contained?" and
attribution confidence are layered questions.)

> **This is a "how to think" trainer, not a hands-on lab.** It builds investigative reasoning
> and methodology. Pair it with a real range — DetectionLab, Atomic Red Team, CyberDefenders,
> DFIR NetWars, or your own VM — for hands-on execution against real tools and data.

---

## Detection engineering appendix

For detection engineers and hunters who want to go from *concept* to *their own rules*, see
**[`DETECTIONS.md`](DETECTIONS.md)**. In keeping with the "how to think" goal, it gives
**starter detections as tool-agnostic pseudocode** (renamed-LOLBin execution, the
rename/masquerade meta-hunt, sanctioned-SaaS dead-drop C2, GUI-as-injection-sleeve, fileless
memory/registry artifacts, and the pcap/NDR angle) with the real published rules to anchor
against — then, for each, **how to evolve it, how to baseline and tune it yourself, and the
trend signals to watch** (what a spike, a drop-off, a first-seen, or a beacon pattern means,
and the adjacent patterns to pivot to). It closes with a reusable "write your own" recipe.
You supply the field names, thresholds, and query syntax for your stack — that's the point.

---

## Build a range (hands-on next step)

The trainer builds the reasoning; **[`RANGE.md`](RANGE.md)** builds the hands-on — a free endpoint range where you **emulate the TTPs** with open tooling (Caldera / Atomic Red Team) and hunt the Sysmon telemetry you generate — behavioral fidelity, no live malware. Every artifact is self-generated and sterile by construction.

---

## How to use

- **Locally:** download the repo (or just `index.html`) and open it in a browser.
- **GitHub Pages:** enable Pages on this repo (Settings → Pages → deploy from `main` / root).
  This trainer then serves at `…/squidoor/` — no build step. (See the repo root
  [`README`](../README.md) and [`TRAINER_SPEC.md`](../TRAINER_SPEC.md) for the
  full collection and the build standard.)

Keyboard: `←` / `→` to navigate the walkthrough.

---

## Accuracy & honesty notes

- Every console/evidence panel uses **synthetic, illustrative data** — fake hostnames,
  RFC 5737 documentation IPs, example domains, and placeholder hashes — so you never mistake
  training data for a real indicator. **Pull real IOCs from the Unit 42 and Elastic reports
  and the repos linked inside the tool.**
- Technical facts were **fact-checked line-by-line against the two primary reports.**
  Investigative-narrative details that appear only in the researchers' SANS talk (the
  one-unpatched-host story, the VirusTotal submitter pivots, the YARA livehunt) are
  **labeled as talk-sourced** rather than presented as if from the written reports.
- Attacker-view reasoning is **plausible inference from observed behavior**, not a confession.
- Generator scenarios **paraphrase public reporting** for training — verify specifics against
  the cited sources before any operational use.

If you spot an inaccuracy, please open an issue — accuracy matters more than polish.

---

## Build / provenance

- Build ID: `HUNT-20261015-2250Z` · design-rev 2026-10-16 (fact-checked revision)
- Single file, offline, no external dependencies.
- Built collaboratively with Claude (Anthropic) as a coding/research assistant; all research
  credit belongs to the teams named above.

## License

Code and original content in this repository are released under the **MIT License** (see
`LICENSE`). The license covers *this trainer* — it does not extend to the cited third-party
research, trademarks, or report content, which remain the property of their respective owners.
