# Salt Typhoon DFIR Hunt & IR Trainer
*Network-infrastructure LOTL — Salt Typhoon vs. telecom carriers: routers/switches with no EDR; evidence lives in configs, NetFlow, syslog, and AAA.*

An interactive, **single-file, fully-offline** training tool that walks you through a
**network-infrastructure** intrusion in the shape of the **Salt Typhoon** campaign against
telecommunications carriers — routers and switches with **no EDR**, where the evidence lives in
device configs, NetFlow, syslog, and AAA logs, and where the attacker **lives off the land**, turns
your own gear into a wiretap, and harvests the network's entire trust fabric.

Open `index.html` in any modern browser. No install, no dependencies, no network calls, no
telemetry.

---

## ⚠️ Credit where it's due — and a note on certainty

**This is an independent *educational reconstruction*. I did not discover, analyze, or report on
Salt Typhoon.** All of the underlying research belongs to the teams below.

- **Cisco Talos** · *"Weathering the storm: Salt Typhoon"* analysis (20 Feb 2025) — the detailed
  technical account (valid-credential access, living-off-the-land, JumbledPath, GRE/loopback
  pivoting, config/credential harvesting, anti-forensics).
  <https://blog.talosintelligence.com/salt-typhoon-analysis/>
- **CISA / NSA / FBI + international partners** · joint guidance on a PRC-affiliated actor
  compromising communications infrastructure (3 Dec 2024).
  <https://www.cisa.gov/news-events/alerts/2024/12/03/cisa-and-partners-release-joint-guidance-prc-affiliated-threat-actor-compromising-networks-global>
- **Joint cybersecurity advisory AA25-239A** (Aug 2025) · 13-nation advisory naming exploited CVEs
  and three PRC companies said to provide services to Chinese intelligence.
  <https://www.cisa.gov/news-events/cybersecurity-advisories/aa25-239a>

The frameworks the trainer teaches are credited in the tool and here: **PEAK** (Bianco & Fetterman /
Splunk SURGe), the **Diamond Model** (Caltagirone, Pendergast, Betz), **Pyramid of Pain** (Bianco),
**MITRE ATT&CK** / CTID, **NIST SP 800-61r3**, **RFC 3227**, and **Analysis of Competing Hypotheses**
(Heuer).

> **A deliberate honesty note.** This is a **network-device case with no endpoint telemetry**, so the
> public *host-level* record is thinner than an endpoint investigation. The trainer leans on
> **inference**, and labels it as such. Crucially, the **attribution evolved**: Cisco Talos (Feb 2025)
> declined to name a country and did **not** confirm the IOS XE CVEs; CISA (Dec 2024) called the actor
> **"PRC-affiliated"**; the Aug 2025 advisory named **three PRC companies**. The trainer teaches you to
> state attribution by **source and date**, and to update as the record firms up. The **ATT&CK IDs are
> taken from AA25-239A Appendix A** and verified against the advisory. (The Sept 2025 advisory also confirms,
> in Appendix B, the Cisco IOS XE CVEs — CVE-2023-20198 and CVE-2023-20273 — that Talos had not confirmed in
> Feb 2025, which is the evolving-attribution lesson borne out by the primary source.)

**This project is not affiliated with, sponsored by, or endorsed by Cisco Talos, CISA, the NSA, the
FBI, SANS, or any other party named here.**

---

## What it is

A reconstruction of a network-infrastructure hunt-to-IR workflow, built so you can *practice the
reasoning*. Same engine as the other trainers in this repo; three modes.

### 1. Case walkthrough
16 turns from mission brief to eradication, with a **Defender ⇄ Attacker** view toggle, a per-step
**Observe → Orient → Relevance → Sufficiency** reasoning layer, evolving **framework meters**, a
**confirmation-bias thread** (ACH + watchlist), a **Hunt Lead persona**, and a confidence +
collection-gap tracker.

Spine of the case:
- **No EDR on the devices** — the first question is whether you even *have* the telemetry (config
  diffs, NetFlow, syslog) to answer the question.
- **Living off the land** — Guest Shell, `tcpdump`, EPC; little on disk. You hunt behavior and config
  state, not files.
- **Your gear became a wiretap** — `JumbledPath`, a capture-and-pivot tool, means the incident is
  *collection across the core*, not a single-box compromise.
- **Rotate the whole trust fabric** — they harvested TACACS/RADIUS shared secrets and SNMP strings, so
  eradication means rotating *everything*, staged to avoid a carrier outage.
- **Attribution that evolved** — the bias thread is built around how the public assessment firmed up
  over time and differed by author.

### 2. Exercise generator
A "prompt book" of randomized reps across seven real campaigns — Salt Typhoon among them.

### 3. Case log
A fixed-schema, fillable intelligence-logging form (enumerated fields + ATT&CK IDs + Diamond
vertices) with JSON/TSV export. Load the Salt Typhoon example to see it populated.

---

## Who is this for?

Same bullseye as the repo: **threat hunters, SOC analysts (Tier 1–3), incident responders / DFIR,
detection engineers, and CTI analysts** — with secondary value for SOC/hunt leads, red/purple teams,
and MSSP/MDR service-delivery managers. Salt Typhoon is especially valuable for **network/CTI** teams
and anyone defending infrastructure where endpoint telemetry doesn't reach. It teaches *how to think*;
pair it with a lab for hands-on execution.

---

## Detection engineering appendix

See **[`DETECTIONS.md`](DETECTIONS.md)**. It leads with the lesson of a no-EDR case: the first
"detection" is **instrumentation** (off-box immutable logging, config diffing, NetFlow). Then it gives
tool-agnostic starter analytics (config-diff for GRE/AAA/account/loopback changes; loopback-sourced
device-to-device SSH; TFTP/FTP config pulls; Guest-Shell packet capture on production devices) with how
to evolve, baseline, and watch the trend for each — and is honest about the telemetry wall.

---

## Build a range (hands-on next step)

The trainer builds the reasoning; **[`RANGE.md`](RANGE.md)** builds the hands-on — a free lab on **virtual routers** (GNS3 / VyOS / FRR) that generates **real** config diffs, NetFlow, and syslog from emulated operator actions — artifact-level fidelity, no licensed gear required. Every artifact is self-generated and sterile by construction.

---

## How to use

- **Locally:** download the repo (or just `index.html`) and open it in a browser.
- **GitHub Pages:** enable Pages (Settings → Pages → `main` / root); this trainer serves at
  `…/salt-typhoon/`.

Keyboard: `←` / `→` to navigate the walkthrough.

---

## Accuracy & honesty notes

- Every console/evidence panel uses **synthetic, illustrative data** — fake device names, RFC 5737
  documentation IPs, private RFC 1918 addresses — so you never mistake training data for a real
  indicator. The report-sourced *technique artifacts* (GRE tunnels, loopback-as-SSH-source, port
  `57722`, `JumbledPath`, TFTP/FTP config exfil) are **technique facts from the reports**, not IOCs.
  **Pull real IOCs from the Talos report and the advisories.**
- Attribution is stated in **calibrated, source-dated** terms and explicitly noted as **evolving**.
- **ATT&CK IDs are taken from AA25-239A Appendix A** and verified against the advisory.
- Attacker-view reasoning is **plausible inference from observed behavior**, not a confession, and is
  weighted more cautiously here than in the endpoint trainers.

If you spot an inaccuracy, please open an issue — accuracy matters more than polish.

---

## Build / provenance

- Build ID: `HUNT-20261016-0306Z` · design-rev 2026-10-16 (fact-checked; ATT&CK verified vs AA25-239A Appendix A)
- Single file, offline, no external dependencies.
- Built to the repo's [`TRAINER_SPEC.md`](../TRAINER_SPEC.md); all research credit belongs to the
  teams named above.

## License

Released under the repo's **MIT License** (see [`../LICENSE`](../LICENSE)). The license covers *this
trainer* — not the cited third-party research, trademarks, or report content.
