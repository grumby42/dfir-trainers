# ToolShell — detection engineering appendix

**Teach yourself to author detections for this case — don't copy-paste rules.** Each card gives the
behavior, a tool-agnostic starter in pseudocode, the real anchors to verify against, how to evolve
it, how to baseline and tune it on *your* estate, and the trend signals worth watching. You supply
the field names, thresholds, and query syntax for your stack — that's the point.

Real detections and IOCs live in the primary reports (Unit 42, Microsoft MSTIC, Eye Security, CISA).
Pull them from there; everything below is method, not a feed.

> **The honest part first:** this case has a **detection dead-zone** (Card 5). Once the ASP.NET
> MachineKey is stolen, the attacker's re-entry via forged `__VIEWSTATE` is correctly signed and
> looks like legitimate authenticated traffic — there is no clean signature for it. The mature move
> is to recognize that and reach for a **compensating control (key rotation + assume breach)**, not
> to burn a shift writing a rule that can't exist. Knowing where detection *can't* reach is a
> detection-engineering skill.

---

## Card 1 — The exploit tell: ToolPane POST with a SignOut Referer

**Behavior.** The unauthenticated chain is delivered as a `POST` to the SharePoint `ToolPane.aspx`
endpoint carrying a `Referer` of `/_layouts/SignOut.aspx` — a combination that has no legitimate
reason to occur.

**Starter (pseudocode).**
```
source = iis_logs (SharePoint web front-ends)
where  method == "POST"
  and  uri_path  matches "*/_layouts/15/ToolPane.aspx*"
  and  uri_query matches "*DisplayMode=Edit*"
  and  referer   matches "*/_layouts/SignOut.aspx*"
```

**Real anchors.** Unit 42 (recon GET vs exploit POST, the SignOut Referer); CISA (monitor POSTs to
`/_layouts/15/ToolPane.aspx?DisplayMode=Edit`).

**Evolve it.** A POST alone can be a failed probe. Promote confidence by *joining* this to Card 2/3
on the same host within a tight window — a POST **followed by** a child process or a new `.aspx` is
exploitation, not a knock on the door.

**Tune & baseline.** Learn which of your SharePoint hosts ever legitimately serve `ToolPane.aspx`
(authoring/admin), from which source ranges, and by which accounts. Everything else is signal.

**Watch the trend.** A **spike** in these POSTs = a campaign/scan wave. A **first-seen** on a host
that never got them before = new targeting. A POST that newly **succeeds** (gains a child process)
after weeks of failing = a host that fell out of patch compliance.

**Pivot to.** Source IP → other hosts it touched; the exact minute → the process tree on that host.

---

## Card 2 — w3wp.exe spawning a shell (the success signal)

**Behavior.** The IIS worker process for SharePoint, `w3wp.exe`, spawns `cmd.exe` or
`powershell.exe` — frequently `powershell -EncodedCommand <base64>` — to write the web shell. A web
server process launching a shell is almost never normal.

**Starter (pseudocode).**
```
source = edr | sysmon(ProcessCreate)
where  parent_image endswith "w3wp.exe"
  and  child_image  in ("cmd.exe","powershell.exe","pwsh.exe")
  and  (child_cmdline matches "*-enc*" or child_cmdline matches "*-EncodedCommand*"
        or child_image in ("cmd.exe","powershell.exe"))
  and  host in asset_group("SharePoint")
```

**Real anchors.** Unit 42 and Eye Security (Base64 PowerShell from `w3wp.exe` drops the shell);
Microsoft (commands run through `w3wp.exe`). ATT&CK T1059.001 / T1059.003, under T1190.

**Evolve it.** Decode and retain the `-EncodedCommand` payload — it's your best evidence and often
names the dropped file. Extend the same parent-is-a-server-worker logic to other exposed app pools.

**Tune & baseline.** Some monitoring/deployment agents legitimately spawn children from service
accounts — enumerate them per host and exclude by full path + signer, not by name.

**Watch the trend.** Any `w3wp → shell` on a SharePoint host is a **first-seen-worthy** event; alert
on the first occurrence rather than a threshold. A **cluster** across many hosts in minutes = mass
exploitation.

**Pivot to.** The decoded command → the file it wrote (Card 3) → what that file does (Card 4).

---

## Card 3 — A new .aspx appears in the LAYOUTS directory

**Behavior.** The dropped web shell (`spinstall0.aspx` and variants) is written under the SharePoint
`TEMPLATE\LAYOUTS` path. New or modified server-side code in that directory is a strong tell.

**Starter (pseudocode).**
```
source = edr(FileCreate/FileModify) | file_integrity_monitoring
where  target_path matches "*\\TEMPLATE\\LAYOUTS\\*.aspx"
  and  actor_process endswith "w3wp.exe"   # or a shell it spawned
  and  action in ("create","modify")
```

**Real anchors.** Unit 42 (paths under `...\16\TEMPLATE\LAYOUTS\spinstall0.aspx` and the `15\`
path); Microsoft (look for `spinstall0.aspx` and related names in LAYOUTS). Variants reported:
`spinstall.aspx`, `spinstall1/2.aspx`, `xxx.aspx`, `3plx.aspx`, plus `debug_dev.js`, `info.js`.

**Evolve it.** Don't hard-code `spinstall0.aspx`. Key on *any* new/modified executable content in
LAYOUTS authored by the web process — that survives filename changes (climb the Pyramid). Hash and
retain every match for comparison across hosts.

**Tune & baseline.** Patch cycles and farm solution deployments legitimately touch LAYOUTS — scope
by actor process (a running `w3wp`/shell vs. an installer) and by maintenance windows.

**Watch the trend.** A **first-seen** `.aspx` outside a known deployment window = investigate now. A
**drop-off** to zero after eradication that then **reappears** = incomplete eradication or a stolen
key being reused.

**Pivot to.** The file's content (Card 4); the writing process (Card 2); the originating POST (Card 1).

---

## Card 4 — The "web shell" that leaks a key, not a prompt

**Behavior.** `spinstall0.aspx` is not an interactive command shell. Its response returns ASP.NET
**MachineKey** material (ValidationKey, DecryptionKey, CompatibilityMode). Detect the *behavior of
the response*, not just the file's presence.

**Starter (pseudocode).**
```
source = iis_logs + response_inspection (WAF/proxy/NDR where available)
where  method == "GET"
  and  uri_path matches "*/_layouts/15/spinstall*.aspx"
  and  response_body matches "*ValidationKey*"    # or DecryptionKey / CompatibilityMode
# if response bodies aren't logged: treat ANY GET to spinstall*.aspx as high-sev pending review
```

**Real anchors.** Unit 42 and Eye Security (the shell returns ValidationKey/DecryptionKey/
CompatibilityMode in the GET response; "most probably" built with SharPyShell).

**Evolve it.** Classify recovered web shells by **what they do** — command-exec vs. credential/key-
leak — and route them differently. A key-leak page escalates the whole incident to "assume the key
is gone," which drives Card 5 and the eradication plan.

**Tune & baseline.** Few shops log response bodies; if you can't, make "GET to `spinstall*.aspx`"
alone a high-severity lead and confirm by pulling and reading the file.

**Watch the trend.** Even a **single** successful GET returning key-like material is a
rotate-now trigger — this is not a volume signal.

**Pivot to.** Card 5 (what the stolen key enables) and the eradication runbook (rotate + iisreset).

---

## Card 5 — The dead-zone: forged ViewState re-entry (and what to do instead)

**Behavior.** With the stolen ValidationKey, the actor forges a correctly-signed `__VIEWSTATE` the
server trusts and deserializes — unauthenticated RCE that **survives patching and shell removal**,
until the key is rotated. Because it's correctly signed, it looks like a legitimate authenticated
request.

**Starter (there isn't a clean one — and that's the lesson).**
```
# You cannot reliably signature a correctly-signed, forged ViewState: it is
# indistinguishable from legitimate authenticated traffic.
# DO NOT spend a sprint writing this rule. Instead:
compensating_control:
  - assume the MachineKey is compromised once Card 4 fired (or once breach is suspected)
  - rotate ASP.NET MachineKeys  (Set-SPMachineKey / Machine Key Rotation timer job)
  - iisreset  across all SharePoint servers in the farm
  - then the forged ViewState stops validating
weak_secondary_signals (low fidelity, context only):
  - unusual __VIEWSTATE sizes / deserialization errors in app logs
  - post-rotation: any request still presenting an old-key-signed ViewState
```

**Real anchors.** Eye Security (ValidationKey → forge `__VIEWSTATE` → "full persistence and zero
authentication, even after patching"); Unit 42 / Microsoft / CISA (rotate machine keys + iisreset).

**Evolve it.** Treat "stolen signing key" as a *class* of problem (it recurs beyond SharePoint). The
reflex to build when detection can't reach is: **assume-breach + rotate the secret + restart** — and
document the dead-zone so the next analyst doesn't chase a ghost signature.

**Tune & baseline.** Nothing to tune — this is a control, not a rule. Verify instead that your
rotation runbook works and that `iisreset` actually ran farm-wide.

**Watch the trend.** After rotation, treat **any** sign of old-key activity as "rotation didn't take
everywhere" — a process check, not a detection.

**Pivot to.** The eradication runbook; a farm-wide patch-timing + rotation audit.

---

## Card 6 — GPO modification: the impact tripwire

**Behavior.** The loud cluster (Storm-2603) escalates to SYSTEM, then modifies a **Group Policy
Object** to distribute **Warlock** ransomware domain-wide. A sudden GPO edit from an unusual account
is a late but loud signal.

**Starter (pseudocode).**
```
source = domain_controller(security 5136/5137) | gpo_change_audit
where  object_class == "groupPolicyContainer"
  and  (actor not in gpo_admin_allowlist
        or actor_logon_origin in asset_group("SharePoint"))
  and  change_time not in approved_change_window
```

**Real anchors.** Microsoft (GPO modification distributes Warlock; services.exe disables Defender,
PsExec `-s` → SYSTEM, Mimikatz LSASS, Impacket/WMI lateral). ATT&CK T1484.001, T1562.001, T1003.001,
T1486.

**Evolve it.** Chain it backward: a GPO change whose actor's logon traces to a compromised SharePoint
host is a full kill-chain story. Add the Defender-tamper (services.exe + registry) and Mimikatz/LSASS
reads as corroborating mid-chain signals.

**Tune & baseline.** Enumerate who *legitimately* edits GPOs and when; alert on the delta, especially
edits originating from server workloads rather than admin workstations.

**Watch the trend.** A GPO edit outside change windows is **first-seen-worthy**. Keep **Warlock**
(GPO-pushed) and **4L4MD4R** (a separate failed-attempt loader) as distinct threads — conflating them
corrupts your timeline and your attribution.

**Pivot to.** The actor account → its source host → the SharePoint foothold → the stolen key.

---

## Write your own — an 8-step recipe

1. **State the behavior**, not the IOC. ("A SharePoint worker process launched a shell," not "block
   this IP.")
2. **Pick the data source** that proves it (IIS logs, EDR/Sysmon process + file events, DC audit).
3. **Write the minimal logic** on the most evasion-resistant field (path/lineage/behavior, not a
   filename or hash).
4. **Separate "tried" from "succeeded."** Join the exploit attempt to a follow-on effect before you
   call it an incident.
5. **Baseline it** on your own estate; list the legitimate causes and scope them out by path +
   signer + window, not by name.
6. **Decide the trend you care about** — spike, drop-off, first-seen, reappearance — and alert on the
   one that matches the threat.
7. **Name the coverage gap.** If part of the behavior is a dead-zone, say so and attach a
   compensating control instead of a doomed rule.
8. **Map it to ATT&CK**, log it in the case schema, and write down what it *can't* catch — so the
   next analyst starts where you left off.

---

*Generated 2026-10-16T02:07Z · accompanies trainer build HUNT-20261016-0207Z · ATT&CK IDs verified
against Microsoft MSTIC · no fabricated IOCs (behavioral signatures are technique facts from the
cited reports; pull real IOCs from those reports).*
