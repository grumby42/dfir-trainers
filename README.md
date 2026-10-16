# DFIR Trainers

Interactive, **single-file, fully-offline** HTML trainers that walk you through
real intrusion investigations **turn-by-turn** — the reasoning, the frameworks,
the bias checks, and the detection engineering — not just a list of IOCs.

Each trainer is built to a shared template and a strict fact-check standard (see
**[`TRAINER_SPEC.md`](TRAINER_SPEC.md)**). Every technical claim is checked
line-by-line against primary vendor reporting; indicators shown in-app are
**synthetic placeholders**, not real IOCs; and all research credit belongs to the
teams who did the original work.

Open any trainer's `index.html` in a modern browser — no install, no
dependencies, no network calls, no telemetry.

---

## Trainers

| Trainer | Case | Status |
|---|---|---|
| **[Squidoor / FinalDraft](squidoor/)** | CL-STA-0049 — suspected Chinese espionage backdoor (endpoint + cloud dead-drop C2) | ✅ Published |
| **[ToolShell / SharePoint](toolshell/)** | CVE-2025-49704/49706 + 53770/53771 — on-prem SharePoint RCE; stolen MachineKey; multi-actor | ✅ Published |
| **[Salt Typhoon](salt-typhoon/)** | PRC-affiliated intrusions into telecom core (network-device DFIR, no EDR, LOTL, long dwell) | ✅ Published |

---

## Who these are for

Threat hunters, SOC analysts (Tier 1–3), incident responders / DFIR analysts,
detection engineers, and CTI analysts — with secondary value for SOC / hunt team
leads, red / purple teamers, and MSSP / MDR service-delivery managers and
consultants.

These teach **how to think** through an investigation — relevance and sufficiency
calls, framework reasoning, bias discipline, and turning findings into
detections. They are **not** hands-on labs; pair them with a real range
(DetectionLab, Atomic Red Team, CyberDefenders, your own VMs) for execution
against live tools and data.

---

## How to use

- **Locally:** download the repo (or just a trainer's `index.html`) and open it
  in a browser.
- **GitHub Pages:** enable Pages (Settings → Pages → deploy from `main` / root).
  Each trainer serves at `/<trainer>/` — e.g. `…/squidoor/`.

Keyboard: `←` / `→` to navigate a walkthrough.

---

## Building new trainers

New trainers must meet **[`TRAINER_SPEC.md`](TRAINER_SPEC.md)** — the template
contract, the fact-check rigor bar, the credit/licensing rules, and the repo
structure. The **[`_template/`](_template/)** folder explains how to start one
from the Squidoor reference implementation.

The guiding principle, same for every case: **no hallucinated, fabricated, or
guessed facts; ≥2 primary sources read first-hand; no fabricated IOCs; credit
where it's due.**

---

## License

Code and original content are released under the **MIT License** (see
[`LICENSE`](LICENSE)). The license covers *these trainers* — it does not extend to
the cited third-party research, trademarks, or report content, which remain the
property of their respective owners. Not affiliated with, sponsored by, or
endorsed by any party named in a trainer.

---

*Index generated 2026-10-16T01:36Z · Squidoor trainer build `HUNT-20261015-2250Z`.*
