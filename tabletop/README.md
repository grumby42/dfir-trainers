# Tabletop & IR-Readiness Facilitator
*Run a case or a readiness scenario as a facilitated tabletop, score it against NIST SP 800-61r3 / CSF 2.0, and generate a one-page after-action — the advisory-side companion to the analyst trainers.*

An interactive, **single-file, fully-offline** facilitator. Where the trainers teach an *analyst* how to think
through a case, this runs the same intrusions — plus readiness-only scenarios — as a **decision exercise for a
room**: who owns each call, what good looks like, and how mature the response capability actually is.

## ⚠️ Credit where it's due

- **NIST** · *SP 800-61r3, Incident Response Recommendations and Considerations for Cybersecurity Risk
  Management* (2025), which frames incident response around the **CSF 2.0** Functions (Govern, Identify,
  Protect, Detect, Respond, Recover) — the scoring frame used here.
  <https://csrc.nist.gov/projects/incident-response>
- **Jeff Bollinger, Brandon Enright & Matthew Valites** · *Crafting the InfoSec Playbook* (O'Reilly) — the
  detection-/response-as-plays model behind the inject + "what good looks like" structure.
- **Colby A. Clark** · *Cybersecurity Incident Management Master's Guide* (Vol. 1–2) — IR as an organizational
  capability: roles, authority, and communications, which shape the role lanes here.
- **Black Hills Information Security** · *Backdoors & Breaches* — the original IR card game, whose
  dice-and-uncertainty teaching inspired this tool's optional **uncertainty mode**. We borrowed the *mechanic*
  and wrote our own content; for full gameplay, play the original. <https://www.blackhillsinfosec.com/projects/backdoorsandbreaches/>
- Readiness scenarios also reference **CISA** (#StopRansomware, supply-chain guidance), **GDPR Art. 33**, and
  **OFAC** ransomware-payment guidance — all to be verified with counsel for your jurisdiction.

**Not affiliated with, sponsored by, or endorsed by NIST, CISA, or any other party named here.**

## What it is

1. **Pick a scenario** — six **case** scenarios replay the repo's trainers (Squidoor, ToolShell, Salt Typhoon,
   OneDrive, Scattered Spider, Cloud Identity) as decision injects; three **readiness** scenarios drill the
   calls with no clean technical answer (ransomware-extortion decision, breach-notification clock, third-party
   vendor compromise).
2. **Name the room (optional)** — assign each of six roles (SOC Lead, IR Manager, IT/Identity Admin,
   Legal/Privacy, Comms/PR, Exec Sponsor) so the after-action records who owned which call.
3. **Run the injects** — each inject names the **decision owner**, the **CSF 2.0 function** it exercises, a
   facilitator prompt, a reveal of **what good looks like**, a **maturity rating** (Ad hoc → Developing →
   Defined → Managed), and a free-text note for the decision/gap.
4. **Generate the after-action** — a one-page **readiness snapshot** by CSF function, a **top-gaps list** with
   recommendations, and the participant roster — copyable as text or JSON.
5. **Uncertainty mode (optional)** — inspired by *Backdoors & Breaches*: declare the CSF functions you're
   **Established** in, then roll a **d20** per inject to see whether your tooling/process actually caught the
   step (11+ succeeds, +4 if Established; natural 20/1 auto). A miss throws a **curveball** complication and
   feeds a **detection-coverage** read into the after-action. All original content — the mechanic is credited
   to B&B, not copied.

Everything runs in the browser; nothing leaves the machine. Keyboard `←` / `→` to step injects.

## How to use

- **As a facilitator:** project it, pick a scenario, and step the room through the injects — rate each as you
  go. Export the after-action when done.
- **Solo:** use it as a self-assessment of your own IR program against each scenario.
- **Locally / GitHub Pages:** open `index.html`, or browse to `…/tabletop/`.

## Honesty notes

- The maturity scores reflect the **facilitator's in-room judgement**, not an audit — use them to prioritize,
  not to grade.
- Readiness scenarios are **playbook-level**, not a legal opinion. Every regulatory specific (notification
  windows, sanctions exposure) must be verified with counsel for your jurisdiction.
- Case scenarios use the same **synthetic** details as the trainers they replay.

## Build / provenance

- **Author:** Theodore "Ted" Gransbury — [github.com/grumby42](https://github.com/grumby42) · [grumbylearns.io](https://grumbylearns.io) · [linkedin.com/in/tgransbury](https://www.linkedin.com/in/tgransbury). Independent educational tool, **assembled with AI assistance (Claude)**.
- Build ID: `HUNT-20261016-2024Z` · design-rev 2026-10-16
- Single file, offline, no external dependencies, no network calls, no telemetry.

## License

MIT (see repo `LICENSE`). Covers this tool, not the cited frameworks/books, which remain their owners'.
