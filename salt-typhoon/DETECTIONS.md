# Salt Typhoon — detection engineering appendix

**Teach yourself to author detections for a network-device case — don't copy-paste rules.** Each card
gives the behavior, a tool-agnostic starter in pseudocode, the real anchors to verify against, how to
evolve it, how to baseline and tune it on *your* estate, and the trend signals worth watching. You
supply the field names, thresholds, and query syntax for your platform.

Real detections and IOCs live in the primary reports (Cisco Talos; CISA/NSA/FBI; AA25-239A). Pull them
from there; everything below is method, not a feed.

> **Read this first — the control before the rule.** This is a **no-EDR** case. The single most
> important "detection" is **instrumentation**: ship device logs **off-box** to immutable storage,
> back up and **diff running-configs**, and collect **NetFlow**. The actor routinely clears local logs
> (`.bash_history`, `auth.log`, `wtmp`, `btmp`, `lastlog`) and runs `guestshell disable` to restore a
> normal look — so anything that lives only on the device may be gone. Every card below assumes you did
> Card 0.

---

## Card 0 — Instrumentation (the prerequisite)

**Behavior.** Not an attacker behavior — your posture. On network gear there is usually no endpoint
sensor and only local logging. Without off-box telemetry, none of the analytics below can fire, and
cleared local logs read as "clean."

**Starter (control, not a rule).**
```
ensure:
  - syslog + AAA (TACACS+/RADIUS) logs shipped to immutable, off-box storage
  - running-config backups on a schedule, with automated DIFF + alert on change
  - NetFlow/IPFIX export from core + edge devices to a SIEM
verify:
  - retention covers your plausible dwell window (this campaign ran 3+ years)
```

**Real anchors.** Talos (logs routinely cleared; `guestshell disable` restores normal); CISA/NSA/FBI
hardening guidance (visibility + logging for communications infrastructure).

**Evolve it.** Treat config history as a first-class data source (RANCID/Oxidized or vendor
equivalent). Alert on *any* diff, then triage by change-ticket.

**Tune & baseline.** Map your normal change cadence and maintenance windows so diffs are scoped to
"out-of-ticket."

**Watch the trend.** A **drop-off** in device log volume, or a device that suddenly stops shipping
syslog, is itself a signal — it can mean logging was tampered with.

**Pivot to.** Everything else here.

---

## Card 1 — Config diff: GRE tunnel / rogue account / AAA-server change

**Behavior.** Running-config gains an un-ticketed **GRE tunnel**, an **unexpected local account**, a
**loopback edit**, or a **changed AAA/TACACS+ server IP** (so the actor's own server can authorize
logins).

**Starter (pseudocode).**
```
source = config_diff (off-box backups)
alert when a diff adds any of:
   "interface Tunnel" with "tunnel mode gre"
   a new local "username ... privilege 15"
   a changed "aaa ... server" / "tacacs server" host IP
   a modified "interface Loopback" address
  AND no matching approved change ticket in the window
```

**Real anchors.** Talos (GRE tunnel creation; AAA/TACACS+ server IP changes; unexpected local
accounts; loopback changes). ATT&CK (AA25-239A Appendix A) T1572, T1556, T1136.001.

**Evolve it.** Join each diff to your change-management system automatically; the signal is the
*un-ticketed* change with campaign shape, not any change.

**Tune & baseline.** Enumerate who legitimately edits these stanzas and when; exclude by author +
window, not by device.

**Watch the trend.** A **first-seen** GRE tunnel on a device that never had one, or a **new** AAA
server IP, is high-severity. A **cluster** of similar diffs across devices in a short window = campaign
activity.

**Pivot to.** Who made the change (AAA logs, Card 4); where the tunnel/loopback points (Card 2/3).

---

## Card 2 — Loopback-sourced, device-to-device SSH (ACL bypass)

**Behavior.** The actor changes a switch's **loopback** and uses it as the **SSH source** to reach
other devices, bypassing ACLs that trust internal loopbacks. Device-to-device SSH is abnormal.

**Starter (pseudocode).**
```
source = netflow/ipfix + device auth logs
flag SSH (tcp/22, or alt port per Card 5) where:
   src is a device loopback address
   dst is another network device (not an admin jump box)
   OR src device has no business initiating admin SSH to dst
```

**Real anchors.** Talos (repeatedly changed a switch loopback and used it as the SSH source to bypass
ACLs; machine-to-machine pivoting across companies/vendors). ATT&CK (AA25-239A Appendix A) T1090.003,
T1021.004.

**Evolve it.** Baseline which devices *ever* legitimately SSH to which — a small, stable set. Anything
outside it, especially loopback-sourced, is signal. Extend across trust boundaries: check inbound SSH
from peer/partner networks.

**Tune & baseline.** Build the device-to-device management matrix once; most pairs never talk.

**Watch the trend.** A **new** device-to-device SSH pair = investigate. Pivoting that **crosses an org
boundary** = widen scope and notify the peer.

**Pivot to.** The dst devices (new in-scope); the GRE tunnel carrying exfil (Card 1).

---

## Card 3 — Running-config pulled over TFTP/FTP

**Behavior.** A device's **running-config** is pulled over **TFTP/FTP** to a non-admin destination. The
config is a credential store (SNMP strings, weakly-encrypted local accounts).

**Starter (pseudocode).**
```
source = netflow + device logs
flag when:
   protocol in (tftp/69, ftp/21) with a device as source
   dst not in (approved config-backup servers)
   OR a device log shows "copy running-config tftp:" to an unknown host
```

**Real anchors.** Talos (configs exfiltrated over TFTP/FTP; contained SNMP community strings and
weak-crypto local accounts). ATT&CK (AA25-239A Appendix A) T1602.002.

**Evolve it.** Allowlist your real backup servers; everything else pulling configs is suspect. Treat a
config pull as a **credential-exposure** event, not just data movement.

**Tune & baseline.** You already know your backup infrastructure — small allowlist, high fidelity.

**Watch the trend.** Any config pull to a **non-allowlisted** destination is a first-seen-worthy alert;
a **burst** across devices = mass harvest.

**Pivot to.** What's in the pulled config → rotate those secrets (Card 6 / eradication).

---

## Card 4 — Guest Shell / packet capture on a production device

**Behavior.** **Guest Shell** is enabled and **packet capture** (`tcpdump`, EPC, or a dropped Go/ELF
utility like JumbledPath) runs on a production core device. Legitimate tools, illegitimate context.

**Starter (pseudocode).**
```
source = device logs / AAA command accounting
flag:
   "guestshell enable" on a production (non-lab) device
   bash/tcpdump/EPC capture started on a core device
   an ELF written under bootflash/guestshell outside maintenance
```

**Real anchors.** Talos (LOTL hallmark; Guest Shell + Bash; tcpdump/Tpacap/EPC; JumbledPath Go/ELF run
from Guest Shell on Nexus, captures via jump hosts, clears logs). ATT&CK (AA25-239A Appendix A) T1040.

**Evolve it.** A capture tool that pivots via jump hosts means **network-wide collection** — scope by
the jump-host chain, not the one device. Capture command-accounting (AAA) so you see shell activity
even when on-box history is cleared.

**Tune & baseline.** Production core devices should essentially never run packet captures; make it a
near-zero-baseline alert.

**Watch the trend.** Any Guest-Shell-enable or capture on a production device is first-seen-worthy;
repeated enable/disable cycles = anti-forensic resets.

**Pivot to.** The capture targets (AAA/SNMP traffic → Card 6); the jump-host chain (scope).

---

## Card 5 — Shell-level persistence on the device

**Behavior.** An **alternate SSH daemon on a high port** (Talos reports e.g. `57722`), new Linux users
via `/etc/passwd` + `/etc/shadow`, and **SSH authorized_keys** entries — several independent doors.

**Starter (pseudocode).**
```
source = device shell / config integrity checks
flag:
   a listening sshd on a non-standard port (compare to approved mgmt ports)
   new entries in /etc/passwd, /etc/shadow, or authorized_keys on network OS shells
   accounts/keys with no provisioning record
```

**Real anchors.** Talos (alternate SSH daemon on high ports e.g. 57722; new users via /etc/passwd +
/etc/shadow; authorized_keys). ATT&CK (AA25-239A Appendix A) T1098.004, T1136.001, T1021.004.

**Evolve it.** Enumerate ALL persistence layers before acting — a rogue **AAA server** (Card 1) lets
the actor back in even after you delete local accounts. Partial removal invites return.

**Tune & baseline.** You know your approved mgmt ports and provisioned keys; the delta is signal.

**Watch the trend.** A listener on a new high port, or an authorized_keys entry you can't tie to
provisioning, is first-seen-worthy.

**Pivot to.** The full persistence inventory → staged, complete eviction.

---

## Card 6 — The trust-fabric problem (why eradication is big)

**Behavior.** Not a single detection — the consequence. The actor captured **TACACS/RADIUS shared
secrets** and **SNMP strings** and pulled configs with weak-crypto accounts. Those secrets authenticate
across the whole estate.

**Starter (there's no rule — it's an eradication control).**
```
once harvest is confirmed (Card 3/4), eviction REQUIRES:
  - rotate ALL device credentials
  - rotate TACACS+/RADIUS shared secrets
  - rotate SNMP community strings / move to SNMPv3
  - remove rogue accounts, keys, tunnels, and AAA-server entries
  - reimage devices where integrity is in doubt (don't trust the config)
  - stage per-device to preserve carrier service (no outage)
weak_signal_only:
  - post-rotation logins still using an old secret = rotation didn't take everywhere
```

**Real anchors.** Talos (captured SNMP/TACACS/RADIUS incl. shared secrets; config exfil with weak-crypto
accounts); CISA/NSA/FBI hardening guidance.

**Evolve it.** Treat "captured shared secret" as a class: once the trust fabric is exposed, rotation is
the only true eviction. Build a rotation runbook you can execute staged, farm-wide.

**Tune & baseline.** Nothing to tune — verify the rotation actually propagated to every device and AAA
server.

**Watch the trend.** Any sign of old-secret use after rotation = incomplete rotation, not a new intrusion.

**Pivot to.** Peer/partner notification (cross-org pivoting means your exposure may be theirs too).

---

## Write your own — an 8-step recipe (network-device edition)

1. **Instrument first.** If the telemetry isn't collected off-box, no rule can fire — fix that before
   anything else.
2. **State the behavior** in config/flow terms, not an IOC. ("A device gained an un-ticketed GRE
   tunnel," not "block this IP.")
3. **Pick the data source** that proves it (config diff, NetFlow, AAA logs, syslog).
4. **Join to change management.** The signal is the *un-ticketed* change with campaign shape.
5. **Baseline the device-to-device matrix** and the approved mgmt ports/servers — the deltas are your
   detections.
6. **Scope by the pivot chain**, not the first device — capture-and-pivot tooling implies multi-device.
7. **Name the telemetry gap.** Where local-only logs were cleared, say so; the control is instrumentation,
   not a doomed rule.
8. **Map to ATT&CK** (AA25-239A Appendix A), log it in the case schema, and record what you *can't*
   see — so the next analyst starts where you left off.

---

*Generated 2026-10-16T03:06Z · accompanies trainer build HUNT-20261016-0306Z · ATT&CK IDs verified
against AA25-239A Appendix A · no fabricated IOCs (report-sourced
technique artifacts are technique facts; pull real IOCs from the cited reports).*
