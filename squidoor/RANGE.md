# Squidoor — build-a-range (hands-on next step)

The trainer builds the *how to think*. This builds the *hands-on*: a small, **free** endpoint range where you
**emulate the behaviors** Squidoor used and hunt the telemetry they produce. You do **not** need the real
malware — you reproduce the ATT&CK techniques with safe, open tooling and generate your own Sysmon/EDR
evidence, which is sterile by construction.

> **Difficulty: Easy–medium.** **Compute: 1–2 small VMs.** **Cost: $0.**
> **Fidelity: behavioral (emulated TTPs, real telemetry) — no live malware.**

---

## What you'll need (all free)

- A **Windows 10/11** VM (free 90-day eval ISO) — the victim endpoint.
- **Sysmon** (with a good config, e.g. SwiftOnSecurity/Olaf's) for rich process/file/network telemetry.
- A free place to look at it: **Security Onion**, **Wazuh**, or an **Elastic** stack (any one).
- **MITRE Caldera** and/or **Atomic Red Team** to safely generate the techniques.
- **Velociraptor** (free) for live-response collection; **WinPMEM** if you want a memory capture.
- `cdb.exe` — a real, signed Microsoft debugger from the **Windows SDK** (free); the LOLBin at the heart of
  the case.

Pull real IOCs from the Unit 42 / Elastic reports (linked in the trainer README); don't use real samples.

---

## Step 1 — Emulate the behaviors (play the scenario)

Reproduce the chain as discrete, safe steps and let Sysmon record each:

1. **Web-process spawns a shell.** Stand up a throwaway web process (IIS Express or a simple listener) and
   have it spawn `cmd`/`powershell` running recon (`whoami /all`) — the "fresh IIS foothold" signal.
2. **Renamed LOLBin.** Copy `cdb.exe` to `C:\ProgramData\fontdrvhost.exe` and run it. Use a **safe public
   cdb shellcode-runner PoC that pops calc** (search "cdb.exe shellcode runner calc PoC") — never real
   shellcode. The point is the telemetry: a signed binary with PE `OriginalFileName = CDB.Exe` running from
   a non-standard path with `-c/-cf` args.
3. **GUI-app injection sleeve.** Use **Atomic Red Team T1055** tests to inject into `mspaint.exe`/
   `notepad.exe` so you can hunt "a GUI app doing network I/O / spawning threads on a server."
4. **Scheduled task persistence** (T1053.005) with a benign payload.
5. **Dead-drop C2 pattern.** Point a **non-browser** test process at a **local mock HTTP endpoint** (stand
   up a tiny Flask/`http.server` that mimics a Graph/Drafts API) and have it poll a "subject." This
   reproduces the *pattern* — a non-browser process doing API dead-drops — without touching real Microsoft
   Graph.

## Step 2 — Collect (work on a copy)

Snapshot the VM, then pull: the **Sysmon EVTX** (`Microsoft-Windows-Sysmon/Operational`), scheduled-task
registration, the dropped files, and (optional) a **WinPMEM** memory image. Velociraptor artifacts make this
repeatable.

## Step 3 — Hunt and verify against ground truth

- Does `parent=w3wp/web-proc → cmd/powershell → whoami` fire your H1?
- Can you key a detection on **PE `OriginalFileName=CDB.Exe` + non-standard path + `-c/-cf` args** (so a
  rename doesn't evade it)?
- Does `mspaint.exe` with network I/O + injected threads stand out from baseline?
- Does the non-browser → mock-Graph poll show up in network telemetry?
- Write each as a detection in **DETECTIONS.md** style, then confirm it against what you *know* you did.

## Step 4 — Practice the discipline

Run the walkthrough's Relevance/Sufficiency calls against your own data: which artifacts link by host/lineage/
time, and when do you have "enough" to say it's one incident? Then write the eradication plan (remove shells +
loader + task; rotate any stolen token in the real case).

---

## Where to stop (the emulation boundary)

- **No real Squidoor/FinalDraft samples.** Emulate behavior with Caldera/Atomic; that's safe and sufficient
  for the telemetry.
- **The Outlook/Graph dead-drop is simulated** against a local mock endpoint — never a real tenant.
- Keep everything in an **isolated VM/network**; use throwaway accounts. Nothing here should touch production.

---

*Range guide generated 2026-10-16 · accompanies the Squidoor trainer · behaviors emulated with open tooling,
telemetry self-generated (no real samples/IOCs); tools credited in the trainer README.*
