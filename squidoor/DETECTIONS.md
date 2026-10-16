# Detection Engineering Appendix — think it, don't paste it

> **How these detections are structured — the "play" schema.** Each starter below is a *detection play* in
> the sense of Bollinger, Enright & Valites, *Crafting the InfoSec Playbook* (O'Reilly) and Don Murdoch,
> *Blue Team Handbook: SOC, SIEM, and Threat Hunting Use Cases*: a reusable unit you author, tune, and hand
> off — not a one-off query. Read each starter against these seven fields, and fill all seven when you write
> your own:
>
> 1. **Objective** — the attacker behavior this catches, and why it matters in this case.
> 2. **Hypothesis** — the testable statement ("a NON-X does Y").
> 3. **Data source** — the exact log/table that proves or refutes it.
> 4. **Logic** — the query/rule (the pseudocode here is a starting point, not a drop-in).
> 5. **Known false positives** — what legitimately looks like this, and how you scope it out.
> 6. **Response / triage** — what the analyst does on a hit (the play's "then").
> 7. **Maintenance & trend** — how you baseline/tune it, and the trend signal that says it's drifting.
>
> The starters below already carry the objective, data source, logic, and baseline/tune/trend. When you
> operationalize one, add the two fields a console can't guess for you: **known false positives** and the
> **response** step.

> A companion to the Squidoor DFIR Hunt & IR Trainer. In keeping with the tool's
> "how to think" goal, **these are starting moves, not drop-in rules.** You own the
> field names, the operators, the path exclusions, and the thresholds — because those
> are specific to *your* SIEM/EDR schema, *your* environment, and *your* normal. What's
> below is the **shape of the logic**, the **real rules to anchor against**, and — most
> importantly — **how to evolve each one and read its trend lines.**

**How to read each card**

- **Behavior** — what the attacker actually does, and why it's detectable.
- **Starter logic** — tool-agnostic pseudocode. Map the fields yourself.
- **Real anchors** — published rules/artifacts to compare against (attributed).
- **Evolve it** — climb the Pyramid of Pain: make it survive the attacker swapping tools.
- **Tune & baseline** — how you'd cut false positives with *your* data, not a guess.
- **Watch the trend** — what a spike, a drop-off, a first-seen, or a beat pattern means.
- **Pivot to** — the adjacent data/patterns you check next.

A note on honesty: where a canonical public rule exists, it's cited. Where one doesn't
(some of these are hunts you build from your own baseline), that's said plainly — don't
go looking for a rule that isn't there. All malware-specific facts are from the Unit 42
and Elastic reports cited at the bottom; the illustrative artifacts in the trainer are
*not* IOCs.

---

## 1 · A renamed debugger loading code (cdb.exe → fontdrvhost.exe)

**Behavior.** The actor dropped the Microsoft Console Debugger `cdb.exe`, renamed it to a
system-process name in a user-writable folder, and used it to load shellcode in memory —
a LOLBAS technique Unit 42 calls "quite rare… reported a handful of times."

**Starter logic (pseudocode):**
```
WHEN process_start
WHERE ( pe.original_file_name == "CDB.Exe"  OR  image ENDSWITH "\cdb.exe" )
  AND image_path NOT MATCHES "C:\Program Files*\Windows Kits\*"   // its only normal home
  AND command_line CONTAINS_ANY ( " -c ", " -cf ", " -pd ", ".ini", ".wds" )
// field names, the exact path allowlist, and the arg list are YOURS to map.
```

**Real anchors (compare yours to these):**
- SigmaHQ — *"Potential Binary Proxy Execution Via Cdb.EXE"* (`proc_creation_win_cdb_arbitrary_command_execution`). Keys on `Image|endswith \cdb.exe` **or** `OriginalFileName: CDB.Exe` **and** command line containing ` -c `/` -cf `.
- Elastic protections-artifacts — *"Execution via Windows Command Line Debugging Utility."*
- LOLBAS project — the `Cdb` entry (the authoritative "why this is abusable").

**Evolve it (climb the pyramid).** The filename is the weakest signal — it changes for free.
Two stronger, more durable generalizations:
1. **Key on the PE metadata, not the on-disk name.** `OriginalFileName == "CDB.Exe"` survives
   any rename. That's the apex-of-pyramid version.
2. **Generalize the class.** Don't hunt one binary — hunt *"any Microsoft-signed debugging
   or dev LOLBin running from a user-writable path with a script/config argument."* Same
   idea reaches `ntsd.exe`, `windbg`, `dnx`, `msbuild`, `installutil`, `dnspy`, and friends.

**Tune & baseline.** Legit debugger use exists (developers, crash tooling, some EDR/AV
vendors ship `cdb`). Cut noise by: scoping to **servers first** (where interactive debugging
is rare), excluding the real `Windows Kits` paths, and excluding your known dev/build hosts.
Start **detect-only**, rank by *unsigned-or-wrong-path + non-dev host*.

**Watch the trend.**
- **First-seen** of a signed system-binary name running from `ProgramData`/`Temp`/`AppData`
  is worth more than any threshold — stealthy actors are low-volume by design.
- A **spike** in "signed-binary-name ≠ PE OriginalFileName" events across hosts can be a wave.
- A detection that **goes silent** after you deploy it isn't necessarily a win — confirm the
  log source still flows and your allowlist didn't quietly swallow it.

**Pivot to.** The parent (`services.exe` spawning a "font driver" from `ProgramData` is
nonsense), the `.ini`/`.wds`/config file on the command line, and what the process touches
next (in-memory allocation, network egress).

---

## 2 · The rename / masquerade **meta-hunt** (the durable one)

**Behavior.** Card 1 is one instance of a bigger, tool-independent tell: **a trusted system
name wearing the wrong clothes.** This is the single most reusable idea in the whole case.

**Starter logic (pseudocode):**
```
// (a) metadata mismatch — signed Microsoft binary whose stored name != its filename
WHEN process_start WHERE signer == "Microsoft*"
  AND pe.original_file_name != basename(image)

// (b) system name from the wrong house
WHEN process_start
WHERE image ENDSWITH IN ("\svchost.exe","\lsass.exe","\fontdrvhost.exe","\conhost.exe","\services.exe", ...)
  AND image_path NOT MATCHES ("*\System32\*","*\SysWOW64\*","*\WinSxS\*")

// (c) impossible lineage
WHEN process_start WHERE image ENDSWITH "\fontdrvhost.exe" AND parent NOT IN (expected parents)
```

**Real anchors.** No single rule covers all of this — it's a family. Compare against
SigmaHQ's masquerading rules and Elastic's *"Suspicious Parent-Child Relationship."* Map to
ATT&CK **T1036.003** (Rename System Utility) and **T1036.005** (Match Legitimate Name/Location).

**Evolve it.** The mismatch idea extends past process creation to **services, scheduled
tasks, and loaded modules** (a DLL whose internal name ≠ its file name). Each is a separate,
cheap, high-value hunt.

**Tune & baseline.** Some legit software ships mismatched metadata — build your allowlist of
*known* offenders once; what's left is a short, high-signal list. This is a classic
**baseline hunt** ("know normal, catch abnormal"), not a volume alert.

**Watch the trend.** The value is in **rarity**, not count — a *new* mismatch on a server is
the alert. A rising count usually means a vendor pushed sloppy binaries (tune), not an
intrusion — but verify before you assume.

**Pivot to.** Path, signer, parent, and whether the thing then allocates executable memory or
egresses.

---

## 3 · C2 hiding in sanctioned SaaS (Outlook/Graph drafts + Pastebin)

**Behavior.** Squidoor authenticates to Microsoft Graph with a hard-coded **refresh token**,
reads the attacker's **Drafts** folder (`p_<id>` beacon / `r_<id>` tasking, create→read→delete),
and GETs a Pastebin page used as an install counter. To a proxy it's just allowed,
encrypted traffic to `graph.microsoft.com`. **The maliciousness is *which process* does it
and the *pattern*, not the destination.**

**Starter logic (pseudocode) — three angles, because one isn't enough:**
```
// HOST / EDR angle — the discriminator is the ACTOR
WHEN network_connection OR dns
WHERE dest IN ("graph.microsoft.com","login.microsoftonline.com")
  AND initiating_process NOT IN <YOUR baselined allowlist of legit Graph callers>
// strengthen hard: same process ALSO reaches a paste/raw host (pastebin.com, raw gists, etc.)

// IDENTITY angle — Entra/Azure sign-in & token logs
WHEN token_redeemed WHERE grant_type == "refresh_token"
  AND NOT preceded_by interactive_sign_in(same_identity, recent_window)
// non-interactive refresh with no interactive origin = candidate

// MAIL angle — Graph / mailbox audit
WHEN graph_mail_op WHERE folder == "Drafts"
  AND sequence == create -> read -> delete  BY a non-mail-client process
```

**Real anchors — read this honestly.** There is **no single canonical public rule** for
"non-browser process → Graph + Drafts dead-drop." This is a **hunt you build** from your own
baseline. Anchor your *thinking* on ATT&CK **T1102.003** (Web Service), Microsoft's Entra
sign-in-log hunting guidance, and the general body of Graph-abuse detections — but expect to
author this one yourself. (That's the point.)

**Evolve it.** Generalize "Outlook as C2" to **any trusted SaaS used as a dead-drop**: Graph/
Outlook, Gmail/Drive APIs, Dropbox, Telegram, Discord, Notion, GitHub gists, paste sites. The
reusable hunt is: *"a process making OAuth/API calls to a major SaaS it has no business
using."* The specific service is this quarter's fashion; the shape persists.

**Tune & baseline.** The whole game is the **allowlist of legit Graph callers** — browsers,
Office, Teams, OneDrive, MDM/sync agents, and a long tail of LOB apps and security tools.
Build it from **2–4 weeks** of baseline, run **detect-only** first, and rank candidates by
*process rarity + unsigned/wrong-path + first-seen*.

**Watch the trend (this is where you asked to lean in):**
- **First-seen caller:** a *new* process name appearing as a Graph talker is the alert — more
  than any count.
- **Refresh without sign-in:** a rise in `refresh_token` redemptions **not** preceded by
  interactive auth (especially from endpoints/IPs that never did interactive auth).
- **Beacon shape:** low-jitter, periodic calls to Graph from one host (regular heartbeat ≠
  human mail use).
- **Drop-off that should worry you:** a known-good process's Graph volume suddenly *replaced*
  by a new caller, or your Drafts-churn signal going silent right after an eviction — could
  mean they rotated to a different channel (DNS/ICMP/HTTP), not that they left.
- **Tiny-response dead drops:** repeated GETs to a paste/raw host returning a near-empty body
  (the Squidoor counter returned just `1`).

**Pivot to.** Entra sign-in logs (unusual app/client IDs, non-interactive redemptions,
service-identity "impossible travel"), mailbox audit (`MailItemsAccessed`, Drafts ops), and
the process's full network picture (does it also DNS-tunnel or ICMP-tunnel as a backup?).

---

## 4 · A GUI app behaving like malware (injection sleeves)

**Behavior.** Modules are injected into fresh `mspaint.exe` (or `conhost.exe` on Win11), each
gated by a password. A paint program doing network I/O and hosting named pipes on a server is
nonsense.

**Starter logic (pseudocode):**
```
WHEN (network_connection OR remote_thread_created OR clr/powershell_engine_loaded OR named_pipe_created)
WHERE image ENDSWITH IN ("\mspaint.exe","\conhost.exe","\notepad.exe","\taskhostw.exe", ...)
  AND host_role == "server"     // these GUI apps shouldn't run on servers at all
// any ONE of those behaviors on a GUI image is already odd; on a server it's a flag
```

**Real anchors.** Elastic protections-artifacts — *"Unusual PowerShell Engine ImageLoad"*
(CLR/PowerShell loaded into `mspaint`), *"Suspicious Memory Write to a Remote Process,"* and
*"AMSI Bypass via Unbacked Memory."* Map to ATT&CK **T1055 / T1055.002**.

**Evolve it.** Generalize to *"an innocuous/GUI process exhibiting non-GUI behavior"* —
network egress, injected/remote threads, CLR load, or **named-pipe server creation**. In the
analyzed samples those pipes had **GUID-style names** (`\\.\Pipe\<GUID>`); the *durable*
version of that tell is **"a GUI process creating a GUID-named pipe,"** not the specific GUIDs
(which change per build).

**Tune & baseline.** On workstations, `mspaint`/`notepad` are normal — scope the *behavioral*
conditions tightly (network/injection/CLR), and treat their presence **on servers** as the
cheap first filter.

**Watch the trend.** GUI image names appearing on servers **at all**; **multiple short-lived**
`mspaint` instances in a burst; a rise in GUID-named pipes created by non-service processes.

**Pivot to.** The injector (parent) — that's your implant; the module behavior (PowerShell
without `powershell.exe`, pass-the-hash, enumeration); and lateral movement (WinRM) that
follows.

---

## 5 · When it's fileless — memory, registry, persistence

**Behavior.** The payload is reflectively loaded and **never written to disk**. A disk-only
("dead box") exam misses it — which is exactly why the summit panel kept hammering *capture
volatile data first.*

**Where to look (not a SIEM rule — an acquisition & triage plan):**
- **Memory image.** Hunt injected **RWX private regions** in `mspaint`/`conhost` (Volatility
  `malfind`); `yarascan` for the C2 **channel-name strings** (HTTP/ReverseTCP/DNS/ICMP/Outlook)
  that sit in the binary; and the decrypted **config / AES key / refresh token** in process
  memory.
- **Registry.** The Graph token is cached at
  `HK{CU,LM}\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\UUID\<uuid>` (per both vendors)
  — a concrete, searchable host artifact.
- **Persistence.** A scheduled task (Unit 42 observed `Microsoft\Windows\AppID\EPolicyManager`)
  that launches the loader. Hunt scheduled tasks whose action runs a binary from `ProgramData`/
  `Temp`, or that reference a `.ini`/`.log` payload file.

**Evolve it.** The durable lessons: (a) *fileless ≠ invisible* — it's in RAM and often in the
registry and a persistence mechanism; (b) treat **"signed loader + encrypted blob on disk
(`config.ini`, `wmsetup.log`) + nothing else"** as its own suspicious shape.

**Watch the trend.** A rise in scheduled tasks pointing at user-writable paths; registry
`UUID`-style value churn under `Explorer`; hosts where EDR sees execution but **no
corresponding on-disk PE** (classic reflective-load fingerprint).

**Pivot to.** Dump the suspect process, extract the config, recover the decoder
(Base64 → AES+XOR → Zlib), and read the recovered tasking — some of the highest-value
evidence in the case.

---

## 6 · The network-forensics angle (pcap / NDR)

**Behavior.** Even when the host is clean-looking, the wire tells on it.

**What to hunt (pseudocode / NDR logic):**
```
// non-browser TLS client
WHEN tls_client_hello to ("graph.microsoft.com","login.microsoftonline.com")
WHERE ja3/ja4 NOT IN <known browser/Office/sync fingerprints>

// beacon shape (any channel)
WHEN egress_sessions GROUP BY (src_host, dest) HAVING low_jitter_periodicity == true

// tunneling (the other 8 channels)
WHEN dns  WHERE subdomain_entropy high  AND qtype IN (TXT, NULL, A-with-long-labels)
WHEN icmp WHERE payload_size unusual OR payload_entropy high
```

**Real anchors.** JA3/JA4 fingerprinting (open standards), Zeek/Suricata community rules for
DNS/ICMP tunneling, and your NDR's beaconing analytics. These are *approaches*, not one rule.

**Evolve it.** The reusable idea: **profile your clients, not just your destinations.** A
TLS fingerprint that doesn't match any browser/Office client, talking to a legit cloud, is a
strong actor-level signal regardless of the domain's reputation.

**Watch the trend.** New/unknown JA3/JA4 to major clouds; regular-heartbeat egress; a host
that suddenly **shifts channels** (Graph goes quiet, DNS/ICMP volume rises) — that's often
adaptation after you touched the primary channel, not cleanup.

**Pivot to.** Correlate the wire back to the **process** (EDR) and the **identity** (sign-in
logs). One channel seen on the wire implies the mesh has others — go find the internal
peers (named pipes / bind-TCP between hosts).

---

## Detection health — reading the trend lines

A detection is a living thing. Beyond "did it fire," watch *how* it behaves:

- **A spike** = mass-exploitation wave **or** a noisy new legit app (tune) **or** active
  intrusion. Triage which before you react.
- **A drop-off / sudden silence** is not automatically good. It can mean: the attacker evaded
  or rotated channels, a **log source broke**, or an allowlist got too broad and ate the
  signal. *Investigate silence.*
- **First-seen / rare-value** analytics beat static thresholds against stealthy, low-volume
  actors — "a process/parent/caller we've never seen do this" is often the whole alert.
- **Beaconing / periodicity** (low jitter) is channel-agnostic and hard for the adversary to
  shed.
- **Drift** = your allowlist and baselines age. Re-derive them on a cadence, or your precision
  quietly rots.

---

## Write your own — the thinking recipe

When you build the *next* detection from any report, run this loop (it's the same muscle the
trainer drills):

1. **Start from the behavior, not the IOC.** "What did they *do*," not "what hash."
2. **Pick the most evasion-resistant observable** (climb the pyramid): prefer PE metadata,
   parent/child lineage, a *required* API call, or protocol structure over names/hashes/IPs.
3. **Name the data source that actually carries it** — and confirm you *have* that telemetry.
4. **Write the logic broad first**, as a hypothesis. Narrow later.
5. **Baseline normal, then set exclusions/thresholds from the data** — never from a guess.
6. **Choose detect-only vs. alert** by measured precision, not optimism.
7. **Define the pivot:** when it fires, what do you look at next? (A detection with no pivot
   is a dead end.)
8. **Define the decay plan:** which spike / drop-off / drift tells you to revisit it, and on
   what cadence.

If you can answer 1–8, you can write the detection for *any* of the seven campaigns in the
trainer's generator — which is the whole idea.

---

## Sources & attribution

All malware-specific facts above are from the primary reporting, credited in full in the
README:

- **Unit 42 (Palo Alto Networks)** — *Squidoor*, Lior Rochberger & Tom Fakterman (27 Feb 2025):
  <https://unit42.paloaltonetworks.com/advanced-backdoor-squidoor/>
- **Elastic Security Labs** — *FINALDRAFT / REF7707*, Cyril François, Jia Yu Chan, Salim Bitam,
  Daniel Stepanic (13 Feb 2025): <https://www.elastic.co/security-labs/finaldraft>

Rule/framework references: **SigmaHQ** (github.com/SigmaHQ/sigma), **Elastic protections-artifacts**
(github.com/elastic/protections-artifacts), **LOLBAS** (lolbas-project.github.io), **MITRE ATT&CK**
(attack.mitre.org). This appendix is an independent educational reconstruction and is not affiliated
with or endorsed by any of them. Pseudocode is intentionally tool-agnostic — map it to your own stack.

<!-- provenance -->
*Generated 2026-10-15T23:34Z · accompanies trainer build HUNT-20261015-2250Z · no fabricated IOCs.*


## Further reading (from the DFIR bookshelf)

Methodology and authoring discipline these plays draw on:

- Bollinger, Enright & Valites — *Crafting the InfoSec Playbook* (O'Reilly) — detection-as-plays.
- Don Murdoch — *Blue Team Handbook: SOC, SIEM, and Threat Hunting Use Cases* — use-case authoring + tuning.
- Scott J. Roberts & Rebekah Brown — *Intelligence-Driven Incident Response* (O'Reilly) — F3EAD + Diamond, the intel loop these trainers run on.
- Steve Anson — *Applied Incident Response* (Wiley) — evidence acquisition, host/network artifacts, lateral-movement detection.
