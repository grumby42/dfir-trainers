# Salt Typhoon — build-a-range (hands-on next step)

The trainer builds the *how to think*. This builds the *hands-on*. Salt Typhoon is the hardest **infra** to
reproduce — but the **investigative substance is config diffs, NetFlow, and syslog**, and those you can
generate at full fidelity on **free virtual routers**. You emulate the network and the operator actions; the
artifacts you analyze are real.

> **Difficulty: Hard (infra setup).** **Compute: a few light VMs / virtual routers.** **Cost: $0.**
> **Fidelity: artifact-level (real config/flow/syslog from an emulated network).**

---

## What you'll need (all free)

- **GNS3** or **EVE-NG Community** to build the topology.
- Free router/switch images: **VyOS**, **FRRouting**, or **Arista cEOS-lab** (free). (Cisco IOS/IOL images
  need legitimate access — note the licensing; the open images are enough to generate the artifacts.)
- A **free AAA server**: `tac_plus` (TACACS+) or FreeRADIUS, on a small Linux VM.
- **Oxidized** or **RANCID** for scheduled config backups + **diffing**.
- A flow/log collector: **ntopng**/**ElastiFlow** (free tiers) for NetFlow/IPFIX, and plain **syslog** to a
  Linux box or **Security Onion**.

Pull real IOCs from the Talos / CISA / AA25-239A reporting (linked in the trainer README).

---

## Step 1 — Build the network, then play the operator

Stand up a small topology: 2–3 routers, a management host, the AAA server, and the flow/syslog collector.
Capture a clean config + flow baseline. Then perform the operator actions the trainer teaches, one at a time:

1. **GRE tunnel** — add an `interface Tunnel` (GRE) with a loopback source.
2. **Rogue local account** — add a privilege-15 local user with no change ticket.
3. **AAA server change** — point AAA/TACACS at a new (attacker-stand-in) server IP.
4. **Loopback-as-SSH-source** — change a device loopback and SSH **from it** to a peer to bypass an ACL.
5. **Config exfil** — `copy running-config tftp:` to a non-approved host.
6. **Anti-forensics** — clear a device log to create the gap.

Snapshot configs + flow + syslog after each action.

## Step 2 — Collect

Oxidized/RANCID **config history** (before/after), **NetFlow/IPFIX** records, device **syslog**, and AAA logs.

## Step 3 — Hunt and verify against ground truth

- Does the **config diff** flag the GRE tunnel, the new account, the AAA-server change, and the loopback edit
  (and correlate each to "no change ticket")?
- Does **NetFlow** show **loopback-sourced device-to-device SSH** and the **TFTP config pull** to a non-approved
  destination?
- Is there a **syslog gap** where you cleared logs — and can you tell "gap" from "quiet"?
- Build each as a detection in **DETECTIONS.md** style; confirm against your own actions.

## Step 4 — Practice the discipline

Live the case's core lesson: **instrument first.** Deliberately run one action with logging centralized and one
with local-only logging, then try to reconstruct both — you'll *feel* the no-EDR telemetry wall. Then write the
eradication: rotate the whole trust fabric (device creds, TACACS/RADIUS secrets, SNMP strings).

---

## Where to stop (the emulation boundary)

- **No real Nexus Guest Shell / JumbledPath / IOS XE CVEs** — those need licensed/real gear. You reproduce the
  **config-and-flow artifacts**, which are the actual investigative substance.
- **No real exploit code**; the operator actions are ordinary (if malicious) configuration changes you make
  yourself in a lab.
- Isolated lab only; attacker-stand-in IPs are documentation-range; nothing pointed at real infrastructure.

---

*Range guide generated 2026-10-16 · accompanies the Salt Typhoon trainer · network emulated on free virtual
routers, artifacts (config/flow/syslog) self-generated; tools credited in the trainer README.*
