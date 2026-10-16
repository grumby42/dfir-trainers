# Scattered Spider DFIR Hunt & IR Trainer — "Rogue VM Chronicles"
*Identity-first intrusion — Scattered Spider / UNC3944 / Octo Tempest: help-desk call → hypervisor-layer ransomware, below the endpoint sensors.*

An interactive, **single-file, fully-offline** training tool that walks you through an **identity-first
intrusion** in the shape of **Scattered Spider / UNC3944 / Octo Tempest** — from a **help-desk phone call**
to **hypervisor-layer ransomware** — where the access leaves no malware, the endgame runs *below* your
endpoint sensors, and the whole chain can take **hours, not weeks**.

Open `index.html` in any modern browser. No install, no dependencies, no network calls, no telemetry.

This is the companion to the **Squidoor** trainer — the summit's *other* headline named-actor case — and it
opens a new tradecraft domain for the collection: **social-engineering initial access + identity takeover +
the hypervisor blind spot**.

---

## ⚠️ Credit where it's due

**This is an independent *educational reconstruction*. The research belongs to the teams below.**

- **CISA & FBI** — joint advisory **AA23-320A, "Scattered Spider"** — the canonical TTP catalogue and the
  **MITRE ATT&CK mapping** used in this trainer.
  <https://www.cisa.gov/sites/default/files/2023-11/aa23-320a_scattered_spider.pdf>
- **Microsoft Threat Intelligence** — *"Protecting customers from Octo Tempest attacks across multiple
  industries"* (16 Jul 2025) — hybrid identity takeover (Entra Connect, ADFS), DragonForce ransomware,
  2025 targeting.
  <https://www.microsoft.com/en-us/security/blog/2025/07/16/protecting-customers-from-octo-tempest-attacks-across-multiple-industries>
- **Mandiant / Google Threat Intelligence Group** — *"Defending vSphere from UNC3944"* — the detailed
  **vCenter/ESXi attack chain** that is this trainer's centerpiece (help-desk → vCenter → the orphaned
  staging VM → the `NTDS.dit` disk-mount → hypervisor ransomware).
  <https://cloud.google.com/blog/topics/threat-intelligence/defending-vsphere-from-unc3944/>

The frameworks the trainer teaches are credited in the tool and here: **PEAK** (Bianco & Fetterman), the
**Diamond Model** (Caltagirone, Pendergast, Betz), **Pyramid of Pain** (Bianco), **MITRE ATT&CK**, **NIST SP
800-61r3**, **RFC 3227**, and **Analysis of Competing Hypotheses** (Heuer).

**This project is not affiliated with, sponsored by, or endorsed by CISA, the FBI, Microsoft, Mandiant/Google,
SANS, or any other party named here.**

---

## What it is

A reconstruction of an identity-first intrusion-to-ransomware investigation, built so you can *practice the
reasoning*. Same engine as the other trainers in this repo; three modes.

### 1. Case walkthrough
15 turns from the hunt desk to a fabric-level eradication, with a **Defender ⇄ Attacker** view toggle, the
**Observe → Orient → Relevance → Sufficiency** reasoning layer, evolving **framework meters**, a
**confirmation-bias thread** (ACH + watchlist), a **Hunt Lead persona**, and a confidence + collection-gap
tracker.

Spine of the case:
- **No malware = no incident? No.** Initial access is a *phone call* to the help desk; the tooling is
  *legitimate* RMM. A clean EDR is expected — **identity logs are the witness.**
- **Identity-first** — help-desk reset → new-device SSO → register own MFA → add a federated IdP (sign in as
  anyone) → Entra Connect / ADFS persistence. A password reset doesn't evict this.
- **The hypervisor blind spot** — map an identity into *"ESX Admins,"* log into vCenter (often no MFA), own
  the VCSA, enable SSH on ESXi, reuse an **orphaned VM** as staging, and steal **`NTDS.dit`** by **mounting a
  Domain Controller's virtual disk** — *below* every in-guest EDR.
- **Hours, not weeks** — exfil (double extortion) → delete the backups → encrypt the datastores from the
  hypervisor. Response speed must match.
- **Eradicate the fabric** — reset the *identity trust fabric* (passwords, MFA, federation, tokens, krbtgt
  ×2) + rebuild vCenter/ESXi + restore from **immutable** backups + phishing-resistant MFA + help-desk
  identity-proofing.

### 2. Exercise generator
A "prompt book" of randomized reps across the collection's campaigns (Scattered Spider included).

### 3. Case log
A fixed-schema, fillable intelligence-logging form with JSON/TSV export. Load the Scattered Spider example to
see it populated.

---

## Who is this for?

Same bullseye as the repo — **DFIR / incident responders, SOC analysts, threat hunters, detection engineers,
CTI** — with special value for anyone defending **identity (Okta/Entra/AD/ADFS)**, **VMware/vSphere**, or a
**help desk**, and for MSSP/MDR teams who meet this actor constantly. It teaches *how to think* against a
fast, identity-first, hypervisor-savvy adversary; pair it with a range for hands-on execution.

---

## Detection engineering appendix

See **[`DETECTIONS.md`](DETECTIONS.md)**. It's organized around the two telemetry domains this case actually
lives in — **identity** and the **hypervisor** — with tool-agnostic analytics (help-desk reset → new-device
sign-in; new MFA/federation; RMM-by-a-just-reset-account; `ESX Admins` group-add + vCenter login; ESXi
SSH/VM-power/disk-attach; backup-admin changes) and is candid about the blind spots (in-guest EDR can't see
hypervisor-layer activity → you must ship VCSA/ESXi logs off-box).

---

## Build a range (hands-on next step)

The trainer builds the reasoning; **[`RANGE.md`](RANGE.md)** builds the hands-on — a free lab where the **identity half** (a Microsoft 365 Developer tenant) reproduces real sign-in / MFA / federation logs at high fidelity, the **hypervisor half** uses free ESXi / vCenter eval, and the ransomware / NTDS.dit steps are **emulated** with stand-ins. Every artifact is self-generated and sterile by construction.

---

## How to use

- **Locally:** download the repo (or just `index.html`) and open it in a browser.
- **GitHub Pages:** enable Pages (Settings → Pages → `main` / root); this trainer serves at `…/scattered-spider/`.

Keyboard: `←` / `→` to navigate the walkthrough.

---

## Accuracy & honesty notes

- Every console/evidence panel uses **synthetic data** — fake names (`ACME-CORP`, `OLD-APPSRV-01`),
  RFC 5737 documentation IPs, and example domains. No real IOCs. The report-sourced technique artifacts
  (GRUB `init=/bin/bash`, `vim-cmd`, `net group "ESX Admins"`, `NTDS.dit`, teleport) are **technique facts**,
  not IOCs — **pull real IOCs from the CISA, Microsoft, and Mandiant reports.**
- **ATT&CK IDs are taken from CISA AA23-320A**; the ESXi-specific steps are mapped in prose to the Mandiant
  report.
- **Accuracy note on "rogue VM":** per Mandiant, the staging VM is a **reused orphaned VM**, not a
  newly-created one. (CISA separately documents cloud-instance *creation*, T1578.002, in AWS/EC2 contexts.)
  The trainer states this.
- Attacker-view reasoning is **plausible inference from observed behavior**, not a confession.

If you spot an inaccuracy, please open an issue — accuracy matters more than polish.

---

## Build / provenance

- **Author:** Theodore "Ted" Gransbury — [github.com/grumby42](https://github.com/grumby42) · [grumbylearns.io](https://grumbylearns.io) · [linkedin.com/in/tgransbury](https://www.linkedin.com/in/tgransbury). Independent educational reconstruction, **assembled with AI assistance (Claude)** and fact-checked line-by-line against the primary sources below.
- **Primary sources:** listed above under *Credit where it’s due* and in-app under the *Reading Room* — every technical claim traces to them; all research credit belongs to those teams.

- Build ID: `HUNT-20261016-0337Z` · design-rev 2026-10-16 (fact-checked)
- Single file, offline, no external dependencies.
- Built to the repo's [`TRAINER_SPEC.md`](../TRAINER_SPEC.md); all research credit belongs to the teams named
  above.

## License

Released under the repo's **MIT License** (see [`../LICENSE`](../LICENSE)). The license covers *this trainer* —
not the cited third-party research, trademarks, or report content.
