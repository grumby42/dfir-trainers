# ToolShell — build-a-range (hands-on next step)

The trainer builds the *how to think*. This builds the *hands-on*. ToolShell is one of the heavier platforms
to stand up at full fidelity, so this guide gives you **two tiers**: a full SharePoint lab (if you have the
compute), and — recommended for most — an **artifact-emulation** lab that reproduces the evidence you'd
actually investigate without a full exploit chain.

> **Difficulty: Medium–hard (platform weight).** **Compute: heavy for full lab; light for emulation.**
> **Cost: $0 (evals).** **Fidelity: emulation (recommended) or full-lab in isolation.**

---

## What you'll need (all free)

- **Tier B (recommended, light):** a Windows VM with **IIS** (a Windows feature), **Sysmon**, and any log
  viewer/SIEM. That's enough to reproduce the investigative artifacts.
- **Tier A (full lab, heavy):** **Windows Server** eval + **SQL Server** Express/eval + **SharePoint Server
  2019** eval (180-day) — ~16 GB+ RAM. Patch to a known state. Keep it **air-gapped**.

Pull real IOCs from the Unit 42 / Microsoft / Eye Security reports (linked in the trainer README).

---

## Step 1 — Produce the artifacts

**Tier B — emulate the evidence (recommended):**
1. **The exploit tell in IIS logs.** Hand-write (or script) IIS log entries for a `POST /_layouts/15/
   ToolPane.aspx?DisplayMode=Edit` carrying `Referer: /_layouts/SignOut.aspx`, followed by a `GET
   /_layouts/15/spinstall0.aspx`. This is the signature you hunt.
2. **The key-leak "shell."** Drop a **harmless** `spinstall0.aspx` into an IIS `LAYOUTS`-style folder that
   simply reads a **dummy** MachineKey value from config and writes it to the response — so you can see what
   "the shell leaks the key, it doesn't run commands" actually looks like. Use fake key material.
3. **The process tell.** Script `w3wp.exe`-style → `powershell -EncodedCommand <base64>` that writes the
   `.aspx`, and let Sysmon capture the lineage + the LAYOUTS file-write.
4. **The eradication step.** Set a test ASP.NET `machineKey` in a sample app's `web.config`, then **rotate
   it** and `iisreset` — so you've practiced the control that actually evicts (not just "delete the file").

**Tier A — full lab (only if you have the compute, and only air-gapped):** build the SharePoint farm and work
from public post-incident writeups to recreate the *detection* artifacts. **Do not run live public exploits
against anything connected to a real network.**

## Step 2 — Collect

IIS logs (`%SystemDrive%\inetpub\logs\LogFiles`), Sysmon EVTX, the dropped `.aspx`, and the `web.config`
before/after rotation.

## Step 3 — Hunt and verify

- Does a detection for **`POST …/ToolPane.aspx` + `SignOut.aspx` Referer** fire on your log?
- Does **`w3wp → powershell -EncodedCommand`** + a **new `.aspx` in LAYOUTS** catch the drop (keyed on the
  behavior, not the filename `spinstall0.aspx`)?
- Can you see the **key-leak GET** returning key-like material?
- After rotation + `iisreset`, confirm the old key no longer validates in your sample app.

## Step 4 — Practice the discipline

Run the walkthrough's sufficiency spine on your data: **recon ≠ compromise ≠ key theft.** Then write the
finding and the eradication order (patch → **rotate MachineKeys** → `iisreset` → reset creds → hunt).

---

## Where to stop (the emulation boundary)

- **No live exploitation outside an isolated, air-gapped lab.** Emulating the artifacts is safer and teaches
  the same investigation.
- **No ViewState-forging recipe.** The trainer keeps the stolen-key → forged-`__VIEWSTATE` re-entry at the
  *defensive* level (why you must rotate), and so should your lab — the detection **dead-zone** stays
  conceptual.
- Fake key material only; throwaway VMs; nothing pointed at production SharePoint.

---

*Range guide generated 2026-10-16 · accompanies the ToolShell trainer · artifacts emulated (no live exploit,
no weaponization), self-generated telemetry; tools credited in the trainer README.*
