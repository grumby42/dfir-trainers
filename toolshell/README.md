# ToolShell DFIR Hunt & IR Trainer
*Web-app RCE — ToolShell SharePoint (CVE-2025-49704/49706, bypasses -53770/53771): the “web shell” is a MachineKey thief, so a patch alone doesn’t evict.*

An interactive, **single-file, fully-offline** training tool that walks you through the
investigation of the **ToolShell** SharePoint exploitation (**CVE-2025-49704 / 49706** and the
patch-bypass variants **CVE-2025-53770 / 53771**) — from the first IIS-log anomaly, through the
realization that the "web shell" is really a **key thief**, to an eradication that a patch alone
can't achieve.

Open `index.html` in any modern browser. No install, no dependencies, no network calls, no
telemetry. Everything runs locally.

---

## ⚠️ Credit where it's due

**This is an independent *educational reconstruction*. I did not discover, analyze, or report
on ToolShell.** All of the underlying research and every technical fact belongs to the teams
below, and this trainer exists only to help defenders learn from their work.

- **The exploit chain's origin** — demonstrated at **Pwn2Own Berlin 2025** by **Dinh Ho Anh Khoa
  of Viettel Cyber Security** (auth bypass + insecure deserialization), as documented by the
  **Zero Day Initiative**. <https://thezdi.com/blog/2025/5/16/pwn2own-berlin-2025-day-two-results>
- **Palo Alto Networks — Unit 42** · *"Active Exploitation of Microsoft SharePoint
  Vulnerabilities"* threat brief (the exploit chain, timeline, IOCs, CL-CRI-1040).
  <https://unit42.paloaltonetworks.com/microsoft-sharepoint-cve-2025-49704-cve-2025-49706-cve-2025-53770/>
- **Microsoft Threat Intelligence (MSTIC / MSRC)** · *"Disrupting active exploitation of
  on-premises SharePoint vulnerabilities"* (22 Jul 2025) — actor attribution (Linen Typhoon,
  Violet Typhoon, Storm-2603) and the verified ATT&CK mapping.
  <https://www.microsoft.com/en-us/security/blog/2025/07/22/disrupting-active-exploitation-of-on-premises-sharepoint-vulnerabilities/>
- **Eye Security** · *"SharePoint under siege"* — first to identify large-scale in-the-wild
  exploitation, and the MachineKey / ViewState-persistence insight.
  <https://research.eye.security/sharepoint-under-siege/>
- **CISA** · guidance and KEV listing for CVE-2025-53770.
  <https://www.cisa.gov/news-events/alerts/2025/07/20/microsoft-releases-guidance-exploitation-sharepoint-vulnerability-cve-2025-53770>

The frameworks the trainer teaches are likewise other people's work, credited in the tool and
here: the **Pyramid of Pain** and the **PEAK** framework (David Bianco; PEAK with Ryan Fetterman /
Splunk SURGe), the **Diamond Model** (Caltagirone, Pendergast, Betz), **MITRE ATT&CK** and the
**Center for Threat-Informed Defense** (Summiting the Pyramid), **NIST SP 800-61r3**, and
**Analysis of Competing Hypotheses** (Richards J. Heuer Jr.).

**This project is not affiliated with, sponsored by, or endorsed by Unit 42, Microsoft, Eye
Security, CISA, ZDI, SANS, or any other party named here.**

---

## What it is

A reconstruction of a real exploitation-to-IR workflow, built so you can *practice the reasoning*.
Three modes (the same engine as the Squidoor trainer in this repo):

### 1. Case walkthrough
16 turns from mission brief to eradication, with a **Defender ⇄ Attacker** view toggle, a per-step
**Observe → Orient → Relevance → Sufficiency** reasoning layer, evolving **framework meters**
(Kill Chain, NIST IR, PEAK, Pyramid of Pain, Diamond), a **confirmation-bias thread** (bias checks +
ACH + watchlist), a **Hunt Lead persona** with SITREPs, and a live confidence + collection-gap tracker.

The spine of the case is three hard lessons:
- **The patch that wasn't** — July 8 fixes were bypassed within days by the 53770/53771 variants;
  "we patched" is where the investigation *starts*.
- **The shell that isn't** — `spinstall0.aspx` runs no commands; its job is to steal the ASP.NET
  **MachineKey**. Deleting the file doesn't remediate a stolen key.
- **The invisible door** — with the stolen key, forged `__VIEWSTATE` RCE looks like legitimate
  authenticated traffic: a **detection dead-zone** whose only answer is **rotate the keys + assume
  breach**.

### 2. Exercise generator
A "prompt book" of randomized reps (distillation, relevance, sufficiency, behavioral inference,
bias-spotting, SITREPs, scoping, pyramid-climbing, structured logging) grounded in seven real,
current campaigns — ToolShell among them.

### 3. Case log
A fixed-schema, fillable intelligence-logging form (enumerated fields + ATT&CK IDs + Diamond
vertices) with JSON/TSV export, so different analysts' logs stay comparable. Load the ToolShell
example to see it populated.

---

## Who is this for?

Same bullseye as the rest of this repo: **threat hunters, SOC analysts (Tier 1–3), incident
responders / DFIR, detection engineers, and CTI analysts** — with secondary value for SOC/hunt
leads, red/purple teams, and MSSP/MDR service-delivery managers and consultants. ToolShell is
especially strong for **IR** (the eradication twist) and **detection engineering** (the honest
dead-zone). It teaches *how to think*; pair it with a hands-on range for execution.

---

## Detection engineering appendix

See **[`DETECTIONS.md`](DETECTIONS.md)**. In keeping with the "how to think" goal, it gives
**starter detections as tool-agnostic pseudocode** (the ToolPane/SignOut exploit tell; `w3wp.exe`
spawning encoded PowerShell; new `.aspx` in `LAYOUTS`; the `spinstall0.aspx` key-leak response; the
GPO impact tripwire) — then, for each, **how to evolve it, how to baseline and tune it, and the
trend signals to watch**. It is deliberately honest about the **forged-ViewState dead-zone**, where
the right answer is a compensating control (key rotation), not a rule.

---

## Build a range (hands-on next step)

The trainer builds the reasoning; **[`RANGE.md`](RANGE.md)** builds the hands-on — a free lab that reproduces the investigative artifacts (the ToolPane + SignOut IIS tell, the key-leak `.aspx`, `w3wp`→PowerShell) via **emulation**, with an optional full-SharePoint tier — no live exploit, no weaponization. Every artifact is self-generated and sterile by construction.

---

## How to use

- **Locally:** download the repo (or just `index.html`) and open it in a browser.
- **GitHub Pages:** enable Pages (Settings → Pages → `main` / root); this trainer serves at
  `…/toolshell/`.

Keyboard: `←` / `→` to navigate the walkthrough.

---

## Accuracy & honesty notes

- Every console/evidence panel uses **synthetic, illustrative data** — fake hostnames, RFC 5737
  documentation IPs, and placeholder values — so you never mistake training data for a real
  indicator. The *behavioral signatures* shown (`ToolPane.aspx`, the `SignOut.aspx` Referer,
  `spinstall0.aspx`, `w3wp.exe` lineage, the `LAYOUTS` path) are **technique facts from the
  reports**, not IOCs. **Pull real IOCs from the Unit 42, Microsoft, and Eye Security reports.**
- Technical facts were **fact-checked against the primary sources above**; ATT&CK IDs are verified
  against the Microsoft report.
- The forged-ViewState re-entry is described **defensively** (mechanism, so you can justify
  rotation) — **not** a weaponization recipe.
- Attacker-view reasoning is **plausible inference from observed behavior**, not a confession.
- A deliberate teaching point: even the primary reporters had to **correct the record** (Eye
  Security revised its CVE attribution on 29 Jul 2025). Label confidence; update when facts change.

If you spot an inaccuracy, please open an issue — accuracy matters more than polish.

---

## Build / provenance

- **Author:** Theodore "Ted" Gransbury — [github.com/grumby42](https://github.com/grumby42) · [grumbylearns.io](https://grumbylearns.io). Independent educational reconstruction, **assembled with AI assistance (Claude)** and fact-checked line-by-line against the primary sources below.
- **Primary sources:** listed above under *Credit where it’s due* and in-app under the *Reading Room* — every technical claim traces to them; all research credit belongs to those teams.

- Build ID: `HUNT-20261016-0207Z` · design-rev 2026-10-16 (fact-checked)
- Single file, offline, no external dependencies.
- Built to the repo's [`TRAINER_SPEC.md`](../TRAINER_SPEC.md); all research credit belongs to the
  teams named above.

## License

Released under the repo's **MIT License** (see [`../LICENSE`](../LICENSE)). The license covers *this
trainer* — not the cited third-party research, trademarks, or report content.
