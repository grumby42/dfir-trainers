# Scattered Spider — detection engineering appendix

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

**Teach yourself to detect an identity-first, hypervisor-savvy actor — don't copy-paste rules.** Each card
gives the behavior, a tool-agnostic starter, the real anchors to verify against, how to evolve it, how to
baseline/tune it, and the trend signals worth watching. You supply the field names, thresholds, and query
syntax for your stack.

Real IOCs live in the primary reports (CISA/FBI AA23-320A, Microsoft Octo Tempest, Mandiant "Defending
vSphere from UNC3944"). Pull them there; everything below is method.

> **Read this first — two telemetry domains, and a blind spot.** This actor lives in **identity logs**
> (Okta/Entra/AD/ADFS) and at the **hypervisor** (vCenter/ESXi) — **not** in endpoint malware. In-guest EDR
> is structurally blind to the endgame (disk-mount `NTDS.dit` theft, ESXi ransomware), so you **must ship
> VCSA/ESXi logs off-box**. And the clock is **hours** — detections have to fire fast and route to a
> response that can cut identity + hypervisor + egress together.

---

## Card 1 — Help-desk reset → sign-in from a new device

**Behavior.** A help-desk password/MFA reset immediately followed by a sign-in from an **unmanaged device** /
new geo (impossible travel). The core account-takeover signal.

**Starter (pseudocode).**
```
source = identity sign-in logs (Okta/Entra) + help-desk/AD reset events
alert when: a password/MFA reset for user U
   is followed within N minutes by a successful sign-in for U where
   device is unmanaged OR geo/ASN is new OR impossible-travel vs last login
```

**Real anchors.** CISA AA23-320A (help-desk social engineering, T1598/T1199/T1656); Mandiant (impersonate
employee → reset, then privileged user → second reset).

**Evolve it.** Add the *second* reset (privileged user) and correlate both to the same caller/number. Enrich
with the proxy hit on a lookalike domain (`victim-okta[.]com`).

**Tune & baseline.** Help desks reset constantly — the signal is the *pairing* (reset + new-device), not the
reset. Baseline each user's normal devices/geos.

**Watch the trend.** A **spike** in resets-then-new-device across an org (or a department) = an active
campaign; a **first-seen** unmanaged device on a privileged account = investigate now.

**Pivot to.** MFA/federation changes (Card 2).

---

## Card 2 — New MFA method / federated IdP / ADFS activity

**Behavior.** The account registers a **new MFA authenticator**, or a **federated IdP is added** with
auto-account-linking, or ADFS shows **DKM key reads / DCSync / a persistent backdoor** — identity-fabric
persistence that survives a password reset.

**Starter (pseudocode).**
```
source = Entra/ADFS/AD audit
alert on: new MFA method registered shortly after a reset
       | new federated domain added OR auto-account-linking enabled
       | ADFS DKM container read | DCSync from a non-DC principal
       | anomalous activity on the Entra Connect server
```

**Real anchors.** CISA (T1556.006 register MFA; T1484.002 Domain Trust Modification); Microsoft (Entra
Connect pivot, ADFS DKM reads, DCSync, ADFS backdoor).

**Evolve it.** Treat *any* federation change as high-severity; these are rare, powerful, and auditable. Alert
on the Entra Connect / ADFS servers as Tier-0 assets.

**Tune & baseline.** Federation changes should be a tiny, known set of admin actions — near-zero baseline.

**Watch the trend.** A **first-seen** federated domain or a DKM read is a standalone high-severity event.

**Pivot to.** RMM/tooling (Card 3) and the vCenter path (Card 4).

---

## Card 3 — Legit RMM installed by a just-reset account

**Behavior.** A legitimate remote tool (**teleport, ngrok, Chisel**, ScreenConnect, TeamViewer, Splashtop,
Tailscale) installed/launched by a recently-compromised account, tunneling outbound. "Using the tool is not
itself malicious" — so hunt the **context**.

**Starter (pseudocode).**
```
source = EDR/process + install telemetry + egress flow
alert when: an RMM/tunneling binary (allow-list by name) is newly installed/run
   by an account that had a recent reset / new-device login
   AND it establishes outbound tunneling (ngrok/teleport/Chisel)
```

**Real anchors.** CISA (Table 1 RMM list, T1219); Microsoft (ngrok, Chisel, AADInternals, Mimikatz,
ADExplorer).

**Evolve it.** Maintain an allow-list of *sanctioned* RMM + who may run it; anything off-list, or on-list in
the wrong hands, is signal. Pair with AD-recon tooling (ADExplorer) and recon of Vault/vCenter/backups.

**Tune & baseline.** IT legitimately uses RMM — baseline the sanctioned tools, hosts, and operators.

**Watch the trend.** RMM install by a just-reset account is first-seen-worthy; a tunnel from a server that
never tunnels is the anomaly.

**Pivot to.** The vCenter path (Card 4).

---

## Card 4 — "ESX Admins" group-add + vCenter login + VCSA reboot

**Behavior.** An AD identity is mapped into **"ESX Admins" / "vSphere Admins"** (e.g. `net group "ESX Admins"
<acct> /add` via WinRM), then logs into **vCenter** (often **no MFA**), then the **VCSA is rebooted** into a
root shell (GRUB `init=/bin/bash`) and SSH is enabled.

**Starter (pseudocode).**
```
source = AD security logs + vCenter/VCSA audit (shipped off-box)
alert on: group change adding a member to "ESX Admins"/"vSphere Admins"
       | vCenter login by a non-standard admin principal (esp. without MFA)
       | VCSA reboot / single-user / root-shell / SSH enable
```

**Real anchors.** Mandiant (ESX Admins group-add via WinRM; vCenter login no MFA; VCSA GRUB `init=/bin/bash`;
teleport persistence). CISA T1018 (vCenter discovery).

**Evolve it.** Treat vSphere admin-group membership as Tier-0 and alert on *any* change. Require MFA on
vCenter; alert when a login bypasses it. Enable vSphere **lockdown mode**.

**Tune & baseline.** vSphere admin changes are rare and planned — near-zero baseline; tie to change tickets.

**Watch the trend.** A first-seen member of ESX Admins, or a VCSA reboot outside maintenance, is a
hypervisor-takeover alert.

**Pivot to.** ESXi host activity + disk-layer credential theft (Card 5).

---

## Card 5 — ESXi SSH + VM power-off + disk attach (the NTDS.dit theft)

**Behavior.** SSH enabled on ESXi hosts; a **Domain Controller powered off**, its **`.vmdk` detached and
attached to a staging VM**, `NTDS.dit` copied, then reattached. **Invisible to the DC's in-guest EDR** —
because it happens at the hypervisor.

**Starter (pseudocode).**
```
source = ESXi/VCSA logs (MUST be off-box) + VM task events
alert on: ESXi SSH enabled / root password changed
       | unexpected VM power-off of a Domain Controller
       | .vmdk detach from one VM + attach to another (esp. an orphaned VM)
# in-guest EDR will show NOTHING for this — do not wait for it
```

**Real anchors.** Mandiant (enable ESXi SSH + reset root; reuse orphaned VM as staging; power off DC, detach
`.vmdk`, copy `NTDS.dit`, reattach — "invisible to any EDR inside the DC's OS").

**Evolve it.** This is a **detection-coverage** lesson: the only witnesses are hypervisor logs. Enforce ESXi
**`execInstalledOnly`** and, on ESXi 8.0+, `esxcli system account set -i vpxuser -s false` so vCenter users
can't reset ESXi root. Encrypt Tier-0 VMs.

**Tune & baseline.** DC power-cycles and disk re-attaches outside maintenance are near-zero baseline.

**Watch the trend.** A DC power-off + a disk attach to an orphaned/decommissioned VM = assume domain
compromise; alert immediately.

**Pivot to.** Exfil + backup sabotage (Card 6) — and a domain-wide credential reset.

---

## Card 6 — Backup sabotage + hypervisor ransomware (the imminent-impact triggers)

**Behavior.** An account is added to **"Veeam Administrators"** (or DA via RDP) and **backup jobs/snapshots/
repositories are deleted**; then a ransomware binary is SCP'd to **`/tmp`**, `vim-cmd` force-powers-off all
VMs, and the encryptor runs with a delay + `-skip_vms` list against the datastores.

**Starter (pseudocode).**
```
source = backup-server audit + ESXi logs
HIGH-SEV alert on: membership change to a backup-admin group ("Veeam Administrators")
                | mass deletion of backup jobs/snapshots/repositories
                | SCP/upload of an unknown binary to ESXi /tmp
                | vim-cmd mass VM power-off
```

**Real anchors.** Mandiant (Veeam Admins add → delete backups; SCP to /tmp; `vim-cmd` power-off;
`nohup ... sleep 14400 && encrypt -skip_vms`). CISA/Microsoft (BlackCat/ALPHV, DragonForce; T1486, T1657).

**Evolve it.** The decisive control is **immutable, isolated backups** — they defeat the whole recovery-denial
step. Treat a backup-admin group change as an **imminent-ransomware** page.

**Tune & baseline.** Backup-admin changes and mass repo deletion are never routine — near-zero baseline.

**Watch the trend.** Backup destruction is the **last warning** before encryption; respond in minutes, with a
coordinated cut (identity + vCenter + egress).

**Pivot to.** Coordinated containment + fabric-level eradication.

---

## Write your own — an 8-step recipe (identity + hypervisor edition)

1. **Pick the right telemetry.** For this actor that's identity (Okta/Entra/AD/ADFS) and hypervisor
   (VCSA/ESXi) — ship both off-box. Endpoint malware is the wrong place to look.
2. **State the behavior as a pairing**, not an IOC. (A reset *plus* a new-device login; a group-add *plus* a
   vCenter login.)
3. **Make federation/vSphere-admin changes Tier-0 alerts** — they're rare, powerful, and auditable.
4. **Name the blind spot.** In-guest EDR can't see hypervisor-layer activity — say so and compensate with
   VCSA/ESXi logging + hardening (lockdown mode, `execInstalledOnly`, `vpxuser` restriction).
5. **Baseline the near-zero events** (federation changes, ESX-Admins membership, DC power-cycles, backup-admin
   changes) so any hit is high-fidelity.
6. **Build for speed.** The chain is hours — route detections to a response that cuts identity + vCenter +
   egress *simultaneously*.
7. **Prefer prevention controls** that remove steps: phishing-resistant MFA, help-desk identity-proofing,
   immutable backups, MFA on vCenter.
8. **Map to ATT&CK** (CISA AA23-320A), log it in the case schema, and record what you couldn't see.

---

*Generated 2026-10-16T03:37Z · accompanies trainer build HUNT-20261016-0337Z · ATT&CK IDs from CISA
AA23-320A (ESXi steps mapped in prose to Mandiant) · all data synthetic; report-sourced technique artifacts
are technique facts, not IOCs.*


## Further reading (from the DFIR bookshelf)

Methodology and authoring discipline these plays draw on:

- Bollinger, Enright & Valites — *Crafting the InfoSec Playbook* (O'Reilly) — detection-as-plays.
- Don Murdoch — *Blue Team Handbook: SOC, SIEM, and Threat Hunting Use Cases* — use-case authoring + tuning.
- Scott J. Roberts & Rebekah Brown — *Intelligence-Driven Incident Response* (O'Reilly) — F3EAD + Diamond, the intel loop these trainers run on.
- Steve Anson — *Applied Incident Response* (Wiley) — evidence acquisition, host/network artifacts, lateral-movement detection.
