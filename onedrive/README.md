# OneDrive Data-Exfil DFIR Trainer

An interactive, **single-file, fully-offline** training tool that walks you through a **data-exfiltration
investigation** built on **Microsoft OneDrive forensic artifacts** — a departing insider syncs corporate
files into a **personal** OneDrive on a managed laptop, and you reconstruct what actually left using the
OneDrive **settings database** and **ODL logs** — while holding a hard line on *what each artifact proves,
and what it does not*.

Open `index.html` in any modern browser. No install, no dependencies, no network calls, no telemetry.

This is the collection's **fourth evidence domain** — cloud-sync / file-provenance forensics — alongside
endpoint (Squidoor), web-app (ToolShell), and network (Salt Typhoon). It's an **artifact-interpretation**
case rather than an adversary hunt, so the star skill is forensic defensibility: namespace presence ≠ local
bytes ≠ upload ≠ a recipient opening the file.

---

## ⚠️ Credit where it's due

**This is an independent *educational reconstruction*. The OneDrive forensic research and tooling belong to
the people below** — this trainer only teaches a disciplined investigation around their work.

- **Brian Maloney** (`@bmmaloney97`) — author of **OneDriveExplorer** (reconstructs the OneDrive namespace
  from `<UserCid>.dat` / the newer SQLite settings DB, and correlates ODL logs) and the **"OneDrive
  Evolution"** reference that tracks how the artifact format changes across client versions (Major Versions
  17–26). Also his SANS writeup on recreating the folder structure from `<UserCid>.dat`.
  <https://github.com/Beercow/OneDriveExplorer> · <https://malwaremaloney.blogspot.com/p/onedrive-evolution.html>
- **Yogesh Khatri** (`@swiftforensics`) — the ODL log research and the `odl.py` de-obfuscator/parser that
  OneDriveExplorer's log parsing is "heavily based on."
  <https://github.com/ydkhatri/OneDrive>
- **Forensics Wiki** — the community reference for OneDrive artifact formats and paths.
  <https://forensics.wiki/microsoft_onedrive/>

The frameworks the trainer teaches are credited in the tool and here: **PEAK** (Bianco & Fetterman), the
**Diamond Model** (Caltagirone, Pendergast, Betz), **Pyramid of Pain** (Bianco), **MITRE ATT&CK**
(Exfiltration to Cloud Storage, **T1567.002**), **NIST SP 800-61r3**, **RFC 3227**, and **Analysis of
Competing Hypotheses** (Heuer).

**This project is not affiliated with, sponsored by, or endorsed by Brian Maloney, Yogesh Khatri, SANS,
Microsoft, or any other party named here.**

---

## What it is

A reconstruction of an insider data-exfiltration investigation, built so you can *practice the reasoning*.
Same engine as the other trainers in this repo; three modes.

### 1. Case walkthrough
14 turns from the forensics bench to a defensible report, with a **Analyst ⇄ Insider** view toggle, a
per-step **Observe → Orient → Relevance → Sufficiency** reasoning layer, evolving **framework meters**, a
**confirmation-bias thread** (ACH + watchlist), a **Hunt Lead persona** demanding SITREPs, and a confidence +
collection-gap tracker.

Spine of the case:
- **The chain of evidentiary questions** — personal account present? → corporate names in its namespace? →
  were they ever *local* (vs online-only placeholders)? → do the *logs* show them sync/upload?
- **The settings DB proves the namespace, not exfil** — OneDriveExplorer reconstructs names/folders/UUIDs;
  that's what the client *knew*, not what left.
- **Files On-Demand placeholder trap** — a name in the tree may be online-only with no local bytes.
- **ODL logs are obfuscated + rolling** — your best "it synced" evidence, but you need the obfuscation map
  to read them and absence isn't proof of absence.
- **The version-drift trap** — OneDrive's format changes across versions (`.dat` → SQLite); match your parser
  to the installed client version or you'll misread the artifact. (This is exactly what Maloney's "Evolution"
  reference exists to track — and it rhymes with ToolShell's "patch that wasn't" and Salt Typhoon's "evolving
  attribution": *the thing you're analyzing keeps moving under you.*)
- **Defensible reporting** — separate FACT / INFERENCE / CANNOT-PROVE for HR and Legal; never assert intent
  from artifacts.

### 2. Exercise generator
A "prompt book" of randomized reps across the collection's campaigns.

### 3. Case log
A fixed-schema, fillable intelligence-logging form with JSON/TSV export. Load the OneDrive exfil example to
see it populated.

---

## Who is this for?

Same bullseye as the repo — **DFIR / incident responders, SOC analysts, threat hunters, detection
engineers, CTI** — with special value for anyone doing **insider-threat, HR/Legal-facing, or
data-exfiltration** investigations, and for examiners who need to write findings that hold up under scrutiny.
It teaches *how to think* about artifacts and defensibility; pair it with a real OneDrive image and
OneDriveExplorer for hands-on parsing.

---

## Detection engineering appendix

See **[`DETECTIONS.md`](DETECTIONS.md)**. It leads with the honest truth of this case: the durable fix is as
much **policy** (restrict personal OneDrive on managed hosts) as detection. It gives a collection-and-parsing
playbook and tool-agnostic analytics (personal-scope creation on a managed host, DLP egress to consumer
OneDrive, corporate names/hashes in a Personal namespace) — and is candid about the telemetry gaps (ODL logs
are obfuscated + rolling; online-only placeholders hide local presence).

---

## How to use

- **Locally:** download the repo (or just `index.html`) and open it in a browser.
- **GitHub Pages:** enable Pages (Settings → Pages → `main` / root); this trainer serves at `…/onedrive/`.

Keyboard: `←` / `→` to navigate the walkthrough.

---

## Accuracy & honesty notes

- Every console/evidence panel uses **synthetic data** — fake file names (`Q3_roadmap.pptx` etc.), user/host
  names, fileIDs, and timestamps. There is **no real case data**. The artifact names and paths shown
  (`<UserCid>.dat`, the settings/logs folders, the `.odl` family, `ObfuscationStringMap.txt`) are **documented
  technique facts**, not IOCs.
- Technique facts were checked against the primary sources above.
- The **"attacker" lane is the insider's plausible reasoning** about an authorized user — inference, never a
  confession — and is weighted especially carefully because a real person's livelihood can ride on it.
- The trainer's whole point is the **proves-vs-doesn't** discipline and writing **FACT / INFERENCE /
  CANNOT-PROVE** for HR and Legal. It explicitly refuses to let an artifact assert intent.

If you spot an inaccuracy, please open an issue — accuracy matters more than polish.

---

## Build / provenance

- Build ID: `HUNT-20261016-0318Z` (fact-checked)
- Single file, offline, no external dependencies.
- Built to the repo's [`TRAINER_SPEC.md`](../TRAINER_SPEC.md); all research credit belongs to the teams named
  above.

## License

Released under the repo's **MIT License** (see [`../LICENSE`](../LICENSE)). The license covers *this trainer* —
not the cited third-party research, tools, trademarks, or report content.
