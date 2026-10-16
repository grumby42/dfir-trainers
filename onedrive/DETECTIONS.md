# OneDrive data-exfil — detection & collection appendix

**Teach yourself to investigate and detect this channel — don't copy-paste rules.** Each card gives the
behavior/artifact, a tool-agnostic starter, the real anchors to verify against, how to evolve it, how to
baseline/tune it, and the trend signals worth watching. You supply the field names, thresholds, and query
syntax for your stack.

Real parsing is done with the authors' tools — **OneDriveExplorer** (Maloney) and **odl.py** (Khatri). Pull
the artifact specifics from their work and the Forensics Wiki; everything below is method.

> **Read this first — policy beats detection here.** The durable fix for "corporate data on a personal
> OneDrive" is a **policy control**: restrict or block **personal** OneDrive sign-in on managed hosts
> (tenant allow-listing / AADJ device policy / DLP). Detecting it after the sync is strictly worse than
> removing the channel — especially because the activity logs (ODL) are **obfuscated and rolling** and
> Files On-Demand **placeholders** can hide whether bytes were ever local. Lead with prevention; treat
> detection as the safety net.

---

## Card 0 — Collection & parsing playbook (do this right first)

**Behavior.** Not a detection — the forensic method. Collect and parse the OneDrive artifacts correctly, on
a copy, with a version-matched tool, or every later conclusion is suspect.

**Starter (procedure).**
```
collect (per user profile, from an image/copy):
  AppData\Local\Microsoft\OneDrive\settings\{Personal,Business1-9}\  (incl. <UserCid>.dat + .previous, SQLite)
  AppData\Local\Microsoft\OneDrive\logs\{Personal,Business1-9,Common}\  (*.odl/.odlgz/.odlsent/.aold)
  ObfuscationStringMap.txt and/or general.keystore / vault.keystore  (to de-obfuscate logs)
  the user registry hive (mount-point resolution); $Recycle.Bin (deleted items)
check the OneDrive CLIENT VERSION, then use a matching parser:
  OneDriveExplorer -f <UserCid>.dat   (older) | -s <sqlite>   (newer)
  odl.py -o out.csv                    (ODL logs)
document tool + version in your notes.
```

**Real anchors.** OneDriveExplorer README (paths, `-f`/`-s`/`-rb`/`-r`, ODL extensions, obfuscation map);
Khatri's `odl.py`; Forensics Wiki (ODL format + paths); Maloney's "OneDrive Evolution" (version drift).

**Evolve it.** Script the whole collection (KAPE target) so it's repeatable and defensible; snapshot the
client version automatically.

**Tune & baseline.** Know which tenant (`Business1`) is sanctioned on your fleet so a `Personal` scope stands
out immediately.

**Watch the trend.** A **first-seen** `settings\Personal` on a managed host is the whole lead.

**Pivot to.** Everything below.

---

## Card 1 — A personal OneDrive on a managed host

**Behavior.** A `settings\Personal` (and `logs\Personal`) folder on a host where only `Business1` is
sanctioned = a personal account was signed in. The channel to watch — **but not, by itself, exfil**.

**Starter (pseudocode).**
```
source = EDR/file telemetry | live collection
flag host where exists:
   %LOCALAPPDATA%\Microsoft\OneDrive\settings\Personal\
   AND host in asset_group("managed") AND sanctioned_tenant == "Business1"
```

**Real anchors.** OneDriveExplorer settings paths (`settings\Personal` vs `Business1-9`); Forensics Wiki.
ATT&CK T1567.002 (the channel).

**Evolve it.** Join to identity/MDM: a personal consumer account on a corp-joined device is the signal; pair
with DLP egress to consumer OneDrive endpoints.

**Tune & baseline.** If personal OneDrive is allowed fleet-wide, this is noisy — which is itself the argument
for the policy control. Where it's disallowed, any hit is high-fidelity.

**Watch the trend.** First-seen personal scope on a managed host; a **spike** across many hosts = a policy
gap or a campaign (e.g., a layoff wave).

**Pivot to.** Parse that scope's settings DB (Card 2).

---

## Card 2 — Corporate names in the Personal namespace

**Behavior.** The Personal scope's settings DB (`<UserCid>.dat`/SQLite) reconstructs a namespace that contains
**corporate** file/folder names.

**Starter (pseudocode).**
```
parse = OneDriveExplorer (-f .dat | -s sqlite) on the Personal scope
compare namespace names/hashes against:
   known corporate file names / sensitive-doc naming conventions
   (ideally) hashes of known corporate documents
flag matches
```

**Real anchors.** OneDriveExplorer (reconstructs names, folders, UUIDs/fileIDs). Maloney SANS writeup.

**Evolve it.** Hash-match, don't just name-match, where you have corporate document hashes — a name is weak,
a hash is strong. Record the fileID to tie later log/availability evidence to the exact item.

**Tune & baseline.** Expect some corp-adjacent names benignly (templates, public docs); weight by sensitivity.

**Watch the trend.** Sensitive corporate names appearing in a **Personal** namespace is the core finding —
but it **proves namespace presence only** (not local bytes, not upload).

**Pivot to.** Local-availability (Card 3) and ODL logs (Card 4).

---

## Card 3 — Available-locally vs online-only (the placeholder trap)

**Behavior.** Files On-Demand means a namespace name may be an **online-only placeholder** with no local bytes.
"In the namespace" ≠ "was on this host."

**Starter (pseudocode).**
```
for each flagged namespace item:
   resolve availability: available-locally | online-only (placeholder)
   only 'available-locally' supports "the bytes were on this host"
```

**Real anchors.** Forensics Wiki / OneDriveExplorer (per-item attributes); Microsoft Files On-Demand behavior.

**Evolve it.** Treat availability as a required field on every item in your report; it changes what you can say
per file. Cross-check `$MFT`/USN for local presence of the actual bytes.

**Tune & baseline.** N/A — this is an interpretation discipline, not a threshold.

**Watch the trend.** A finding that lists online-only items as if they were exfiltrated is the classic
overreach — reviewers will (rightly) attack it.

**Pivot to.** The ODL logs for sync/upload evidence (Card 4).

---

## Card 4 — ODL logs: the sync/upload evidence (obfuscated, rolling)

**Behavior.** The `.odl/.odlgz/.odlsent/.aold` logs record sync/file operations you can correlate back to items
and **timestamp** — your best "it synced" evidence. But they're **obfuscated** (need the map) and **rolling**
(old activity ages out).

**Starter (pseudocode).**
```
de-obfuscate with ObfuscationStringMap.txt (+ keystores) -> parse with odl.py / OneDriveExplorer
find sync/upload operations referencing the flagged fileIDs/names, with timestamps
correlate each log event back to the namespace item (Card 2)
caveat: logs roll -> missing events are NOT exculpatory; decoded output may contain the account email/token
```

**Real anchors.** OneDriveExplorer (log pane correlates logs to the selected item; needs
`ObfuscationStringMap.txt`); Khatri `odl.py`; Forensics Wiki (ODL format).

**Evolve it.** Build the timeline: namespace item → availability → logged sync op + timestamp. That triplet is
a defensible "this item synced to the personal account at this time."

**Tune & baseline.** Know your retention reality — on a busy host ODL coverage may be days, not weeks.

**Watch the trend.** Clusters of sync ops right before a resignation/departure date; a **gap** where logs
rolled off (note it as a gap, not an absence of activity).

**Pivot to.** Deleted/previous state (Card 5) and corroboration.

---

## Card 5 — Deleted & previous state

**Behavior.** `<UserCid>.dat.previous`, `$Recycle.Bin`, and prior namespace snapshots can recover items a
subject removed **after** syncing (to hand back a tidy laptop).

**Starter (pseudocode).**
```
compare current namespace vs <UserCid>.dat.previous  -> items present-then-removed
fold in $Recycle.Bin (OneDriveExplorer -rb) + user hive (-r) for mount points
correlate removed names to post-sync deletion times in the logs
```

**Real anchors.** OneDriveExplorer (`.previous`, `-rb`, `-r`).

**Evolve it.** Deletion-after-sync is a notable behavior — but a recovered name still needs availability + log
context before you weight it. Same discipline as any other item.

**Tune & baseline.** N/A.

**Watch the trend.** Local cleanup clustered right after a sync window is a meaningful pattern — flag it,
corroborate it.

**Pivot to.** The sufficiency gate and corroboration.

---

## Card 6 — Corroborate beyond OneDrive, and prevent it next time

**Behavior.** OneDrive artifacts rarely close a case alone; and the real win is removing the channel.

**Starter (procedure).**
```
corroborate: $MFT/USN + LNK/jumplists (file access), USB history, browser up/downloads, email/webmail,
             and the file bytes where recoverable -> independent artifacts pointing the same way
prevent (policy, the durable control):
   - restrict/block personal OneDrive on managed hosts (tenant allow-list / AADJ / DLP)
detect (safety net):
   - alert on settings\Personal creation on a managed host
   - DLP egress to consumer OneDrive endpoints
   - corporate names/hashes appearing in a Personal namespace
gaps to state plainly: ODL obfuscated + rolling; online-only placeholders hide local presence
```

**Real anchors.** ATT&CK T1567.002; Microsoft tenant-restriction / Files On-Demand controls; your DLP.

**Evolve it.** Checking the other exfil channels both strengthens the OneDrive finding and **rules them out** —
document what you cleared.

**Tune & baseline.** Tie detections to the sanctioned-tenant baseline.

**Watch the trend.** After the policy control, any new `settings\Personal` on a managed host is a policy
exception to investigate.

**Pivot to.** The defensible report (FACT / INFERENCE / CANNOT-PROVE).

---

## Write your own — an 8-step recipe (artifact-forensics edition)

1. **State the artifact question**, not a verdict. ("Are corporate files in a personal OneDrive namespace on
   this host?" not "Did they steal data?")
2. **Collect on a copy** and **check the client version** before parsing (artifact drift).
3. **Parse with a version-matched tool** and record the tool + version.
4. **Separate the evidentiary layers:** namespace presence → local availability → logged sync → deletion.
5. **Say exactly what each layer proves** — and what it doesn't (presence ≠ bytes ≠ upload ≠ recipient access).
6. **Corroborate** with independent artifacts; rule out the other channels.
7. **Set "enough" by the audience** (HR vs. termination vs. litigation) and write FACT / INFERENCE /
   CANNOT-PROVE.
8. **Map to ATT&CK** (T1567.002 and friends), preserve under legal hold, and record what you couldn't see.

---

*Generated 2026-10-16T03:18Z · accompanies trainer build HUNT-20261016-0318Z · all data synthetic (no real
case data); artifact names/paths are documented technique facts, not IOCs · ATT&CK mapped to MITRE
(T1567.002 Exfiltration to Cloud Storage).*
