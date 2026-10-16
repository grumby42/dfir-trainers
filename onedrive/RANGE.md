# OneDrive exfil — build-a-range (hands-on next step)

The trainer builds the *how to think*. This builds the *hands-on*: a small, **free**, fully-local range
where you generate **real** OneDrive artifacts (that you then parse for real) — and because every file is
one you created, the whole thing is **sterile by construction**. No real case data, no malware, no IOCs.

This is the easiest of the five to reproduce at full fidelity — there's **nothing to emulate**. You're
driving the actual OneDrive client and parsing the actual settings DB + ODL logs it writes.

> **Difficulty: Trivial.** **Compute: one Windows VM (or a spare profile).** **Cost: $0.**
> **Fidelity: 100% — these are the real artifacts.**

---

## What you'll need (all free)

- A **Windows 10/11** VM — free 90-day evaluation ISO from Microsoft (Eval Center) or a dev VM image.
- A **personal Microsoft account** (free) for the "personal OneDrive on a managed host" scenario, and —
  if you want the Business side — a **Microsoft 365 Developer tenant** (free, renewable) for a `Business1`
  scope to contrast against.
- **OneDriveExplorer** (Brian Maloney) — parses the settings DB + correlates ODL logs.
- **`odl.py`** (Yogesh Khatri) — de-obfuscates and parses the ODL logs to CSV.
- Optional: **KAPE** (free for non-commercial) to collect the OneDrive folders cleanly; any **SQLite
  viewer** for ground-truth on the settings DB.

Pull the real IOCs/parsers from the authors' repos (linked in the trainer README). Nothing here needs a
paid license.

---

## Step 1 — Generate the artifacts (play the scenario)

On the Windows VM, signed in as a test user:

1. Confirm the OneDrive **client version** first (Settings → About). Write it down — the on-disk format
   changes across versions (the "version drift" lesson), so your parser has to match.
2. Sign a **personal** OneDrive account into the client (this is the "personal account on a managed host"
   signal). If you built a dev tenant, sign the **Business** account in too, so you have both `Personal`
   and `Business1` scopes to compare.
3. Create a handful of **synthetic "corporate" files** with obvious names — e.g. `Q3_roadmap.pptx`,
   `pricing_model.xlsx`, a `client_contracts\` folder — put junk content in them, and drop them into the
   **personal** OneDrive folder. Let them sync.
4. Create a second batch and leave some as **online-only** (right-click → "Free up space") so you have the
   Files-On-Demand **placeholder vs available-locally** contrast the trainer hammers on.
5. **Delete** one or two synced files locally (to mimic post-sync cleanup) so you can practice recovering
   them from `.previous` / the recycle bin.

You've now produced the exact evidence the trainer walks through — and you know ground truth, because you
did each action.

## Step 2 — Collect (work on a copy)

Collect from the user profile (image it, snapshot the VM, or use KAPE's OneDrive target):
```
%LOCALAPPDATA%\Microsoft\OneDrive\settings\Personal\    (<UserCid>.dat + .previous, or the SQLite DB)
%LOCALAPPDATA%\Microsoft\OneDrive\settings\Business1\    (if you added the dev tenant)
%LOCALAPPDATA%\Microsoft\OneDrive\logs\Personal\         (*.odl / *.odlgz / *.odlsent / *.aold)
%LOCALAPPDATA%\Microsoft\OneDrive\settings\...\ObfuscationStringMap.txt   (to de-obfuscate the logs)
```
Always parse a **copy**, never the live profile.

## Step 3 — Parse and verify against ground truth

```
OneDriveExplorer -f <UserCid>.dat          # older format
OneDriveExplorer -s <settings.sqlite>      # newer format
odl.py -o out.csv                          # (run against the Personal logs folder)
```
Now check your parsed output against what you *know* you did:
- Does the reconstructed **namespace** list your synthetic corporate names? (presence ≠ bytes)
- Does **availability** match which files you kept local vs made online-only?
- Do the **ODL logs** show sync operations for the files you synced, with timestamps that match your actions?
- Can you recover the files you **deleted** from `.previous` / the recycle bin?

## Step 4 — Practice the discipline

Now write the finding the way the trainer teaches it — **FACT / INFERENCE / CANNOT-PROVE** — and grade
yourself against ground truth. You'll feel exactly where the artifacts *support* a claim and where an
honest examiner has to stop. That calibration is the whole point.

---

## Where to stop (and why this one doesn't need emulation)

Unlike the heavier cases in this collection, OneDrive needs **no emulation** — the client writes the real
artifacts and you parse them directly. The only thing you can't reproduce in a lab is **recipient-side
activity** (whether someone on the other end opened the files) — which is exactly what the trainer says the
host artifacts *cannot* prove. The range makes that limit tangible instead of abstract.

**Keep it sterile:** every file is synthetic and self-made, so there's nothing sensitive to handle or
publish. Don't point this at a real corporate tenant or real user data — build it in a throwaway VM with
throwaway accounts.

---

*Range guide generated 2026-10-16 · accompanies the OneDrive Data-Exfil trainer · all artifacts are
self-generated and synthetic (no real case data); parsers/tools credited in the trainer README.*
