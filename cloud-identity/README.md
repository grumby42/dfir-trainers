# Cloud Identity DFIR Hunt & IR Trainer — "Consent to Compromise"
*Cloud identity intrusion — Midnight Blizzard / APT29: password spray on a legacy no-MFA account → OAuth app abuse → self-granted `full_access_as_app` → corporate mailbox collection over EWS, all from residential proxies.*

An interactive, **single-file, fully-offline** training tool that walks you through an **identity-and-OAuth
cloud intrusion** in the shape of the **Midnight Blizzard / APT29** compromise of Microsoft's own corporate
tenant (disclosed Jan 2024) — from a quiet **password spray** against a forgotten legacy account, through the
realization that the account was never the prize (**an over-privileged OAuth application was**), to a mailbox
collection operation that leaves **every endpoint green** and defeats IP-based detection by design.

It is the cloud-identity counterpart to the endpoint, web-app, network, and insider cases in this repo: no
malware, no shell — the entire breach lives in **Entra sign-in logs, consent/app-role audit logs, and Exchange
mailbox-access telemetry**.

## ⚠️ Credit where it's due

**This is an independent *educational reconstruction*. I did not discover, analyze, or report on this
intrusion.** Every technical fact belongs to the teams below, and this trainer exists only to help defenders
learn from their work.

- **Microsoft — MSRC & MSTIC (Microsoft Threat Intelligence)** · *"Midnight Blizzard: Guidance for responders on
  nation-state attack"* (25 Jan 2024), and the MSRC disclosure *"Microsoft actions following attack by nation
  state actor Midnight Blizzard"* (19 Jan 2024).
  <https://www.microsoft.com/en-us/security/blog/2024/01/25/midnight-blizzard-guidance-for-responders-on-nation-state-attack/>
- **CISA** · *Emergency Directive 24-02 — Mitigating the Significant Risk from Nation-State Compromise of
  Microsoft Corporate Email System* (Apr 2024).
  <https://www.cisa.gov/news-events/directives/ed-24-02-mitigating-significant-risk-nation-state-compromise-microsoft-corporate-email-system>
- **MITRE ATT&CK** · *APT29 (G0016)* — used to **verify every technique ID** in this trainer.
  <https://attack.mitre.org/groups/G0016/>

The frameworks the trainer teaches are likewise other people's work, credited in the tool and here: the
**Pyramid of Pain** and **PEAK** (David Bianco; PEAK with Ryan Fetterman / Splunk SURGe), the **Diamond Model**
(Caltagirone, Pendergast, Betz), **MITRE ATT&CK** and the **Center for Threat-Informed Defense**, **NIST SP
800-61r3**, **CISA SCuBA**, and **Analysis of Competing Hypotheses** (Richards J. Heuer Jr.).

**This project is not affiliated with, sponsored by, or endorsed by Microsoft, MSTIC, CISA, MITRE, or any other
party named here.**

---

## What it is

Three modes, one case:

1. **Case walkthrough** — 15 phases from mission brief to post-incident, each with an **Observe → Orient →
   Relevance → Sufficiency** reasoning layer, a **Defender ⇄ Attacker** toggle (plausible inference, never a
   confession), evolving framework meters (Kill Chain, NIST IR, PEAK, Pyramid of Pain, Diamond), and a
   confirmation-/availability-bias thread with an ACH gate — tuned for the specific bias traps of a cloud case
   ("clean EDR = clean," "the report said APT29 so it's APT29").
2. **Exercise generator** — randomized reps (distillation, relevance calls, sufficiency gates, behavioral
   inference, bias-spotting, SITREPs, scoping, pyramid-climbing, structured logging) across a cross-case
   scenario pack that now includes this one.
3. **Case log** — a fixed-schema intelligence-logging form (enumerated fields + ATT&CK IDs + Diamond vertices)
   with JSON/TSV export.

The spine of the case: **password spray (T1110.003) → valid cloud account (T1078.004) → malicious OAuth apps +
consent user (T1098.001 / T1136.003) → self-granted `full_access_as_app` (T1098.003) → app-token auth
(T1550.001) → remote email collection over EWS (T1114.002), all behind residential proxies (T1090.002).**

## Who is this for?

Threat hunters, SOC analysts, incident responders, detection engineers, and CTI analysts — and, pointedly,
**proactive / advisory consultants and IR-readiness practitioners**: this case is one of the most-requested
tabletop and maturity-assessment scenarios in the field, and the trainer is built around the reasoning and the
exec-legible decisions that advisory work turns on.

## Detection engineering appendix

See **[`DETECTIONS.md`](DETECTIONS.md)** — pseudocode starters for the four behavior-based detections MSTIC
recommends (self-granted `full_access_as_app`; new service-principal credentials; an app reading mail via Graph
*and* EWS; offline-access/mail consent to unknown apps), with "evolve it," baseline/tune, and trend-signal
guidance. Teach authoring, not copy-paste rules.

## Build a range (hands-on next step)

The trainer builds the reasoning; **[`RANGE.md`](RANGE.md)** builds the hands-on — a **free, high-fidelity**
lab in a Microsoft 365 Developer tenant where you generate **real** Entra sign-in / consent / app-role logs by
emulating each step safely (no live attack infrastructure, no real mailbox theft). This is the most reproducible
case in the repo.

---

## How to use

- **Locally:** download the repo (or just this trainer's `index.html`) and open it in a browser.
- **GitHub Pages:** served at `…/cloud-identity/`.

Keyboard: `←` / `→` to navigate a walkthrough.

## Accuracy & honesty notes

- Technical facts trace to **Microsoft MSTIC** and **CISA ED 24-02**, fact-checked against both.
- **ATT&CK technique IDs were verified against MITRE ATT&CK APT29 (G0016).** The specific ID-to-step *mapping*
  shown is **analyst-mapped** from the reported behavior (MSTIC describes the behavior but does not enumerate
  IDs) and is labeled as such — the same discipline used in the Salt Typhoon trainer.
- **No fabricated IOCs.** MSTIC states the actor used high-churn residential proxies that make IOC-based
  detection infeasible, so the trainer teaches **behavior-based** detection and uses only synthetic tenants /
  accounts / app IDs / documentation-range IPs, clearly labeled.
- **Attribution is stated as "consistent with Midnight Blizzard / APT29," not confirmed** — the OAuth-consent
  playbook is also used by criminal BEC crews, and the trainer runs that competing hypothesis explicitly.

## Build / provenance

- **Author:** Theodore "Ted" Gransbury — [github.com/grumby42](https://github.com/grumby42) · [grumbylearns.io](https://grumbylearns.io) · [linkedin.com/in/tgransbury](https://www.linkedin.com/in/tgransbury). Independent educational reconstruction, **assembled with AI assistance (Claude)** and fact-checked line-by-line against the primary sources below.
- **Primary sources:** listed above under *Credit where it's due* and in-app under the *Reading Room* — every technical claim traces to them; all research credit belongs to those teams.
- Build ID: `HUNT-20261016-1650Z` · design-rev 2026-10-16 (fact-checked revision)
- Single file, offline, no external dependencies, no network calls, no telemetry.

## License

Code and original content in this repository are released under the **MIT License** (see `LICENSE`). The license
covers *this trainer* — it does not extend to the cited third-party research, trademarks, or report content,
which remain the property of their respective owners.
