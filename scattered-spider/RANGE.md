# Scattered Spider — build-a-range (hands-on next step)

The trainer builds the *how to think*. This builds the *hands-on*. Scattered Spider is the most involved case,
so split it: the **identity half is free and reproducible at high fidelity** (a Microsoft 365 Developer
tenant gives you real sign-in / MFA / federation logs), and the **hypervisor half** is reproducible with free
licenses + some compute (and the ransomware step is always a harmless stand-in). The initial-access phone call
is a **tabletop**, not a technical build.

> **Difficulty: Most involved.** **Compute: light for identity; heavier for the vSphere half.**
> **Cost: $0 (dev tenant + free/eval licenses).** **Fidelity: identity = high (real logs); hypervisor =
> artifact-level; ransomware = emulated.**

---

## What you'll need (all free)

- **Microsoft 365 Developer tenant** (free, 90-day renewable) — real **Entra** sign-in, MFA, conditional-access,
  and federation audit logs. This is the high-value, fully-free half.
- Hypervisor half (optional, heavier): **ESXi free license** (single host) + **vCenter** 60-day eval, on
  hardware that can handle nested virt; a small **AD** (Windows Server eval + a DC VM).
- Optional: **Veeam Community Edition** (free) to model backup-admin changes/deletion.
- Log aggregation: **Security Onion** / **Elastic** / Sentinel-free for the identity logs.

Pull real IOCs from the CISA AA23-320A / Microsoft / Mandiant reporting (linked in the trainer README).

---

## Step 1 — Play the scenario (identity first, then hypervisor)

**Identity half (do this one — it's free and the most reproducible):**
1. In the dev tenant, create a test user. **Reset** their password/MFA (the "help-desk reset"), then sign in
   from a **different device/location** (a second VM / browser profile / VPN) to produce the "new-device
   login after a reset" sequence.
2. **Register a new MFA method** as that user, and **add a (test) federated domain** / enable auto-linking in
   Entra — the identity-fabric persistence the trainer flags. Pull the audit log for each.

**Hypervisor half (optional, if you built vSphere):**
3. Add an AD identity to an **"ESX Admins"-style** group; log into **vCenter**; **enable SSH** on the ESXi
   host.
4. Power off a throwaway VM, **detach its `.vmdk`, attach it to a second (staging) VM, and copy a DUMMY file**
   (a stand-in for `NTDS.dit`) — so you can see that the theft happens at the hypervisor, invisible to any
   in-guest agent. Reattach, power back on.
5. (Optional) In Veeam Community, model a **backup-admin group change** + job/repo **deletion**.
6. **"Ransomware" step = a harmless script** that renames/touches files in a *test* datastore. **Never a real
   encryptor.**

**Initial access = tabletop:** script the help-desk call and the agent's decision; discuss identity-proofing.
Don't actually social-engineer anyone.

## Step 2 — Collect

Entra **sign-in + audit logs** (export), AD security logs, **vCenter/ESXi** task + syslog (ship off-box), and
(optional) Veeam audit.

## Step 3 — Hunt and verify against ground truth

- Does the **reset → new-device sign-in → new-MFA/federation** chain reconstruct cleanly from the identity logs?
- (vSphere) Do the **ESX-Admins add + vCenter login + ESXi SSH enable + VM power + disk attach** events show in
  VCSA/ESXi logs — and note that your in-guest AD agent saw **nothing** of the disk-mount?
- Confirm the "clean EDR ≠ clean" and "hypervisor blind spot" lessons against your own data.

## Step 4 — Practice the discipline

Run the sufficiency gate under a **fast clock**: with the identity logs in front of you, write the 3-line SITREP
and the **coordinated cut** (disable identity, revoke sessions, lock vCenter, sever egress). Then the fabric-level
eradication (reset identity fabric incl. krbtgt ×2, rebuild vCenter/ESXi, immutable-backup restore).

---

## Where to stop (the emulation boundary)

- **No real ransomware / encryptor** — a harmless file-touch script only.
- **NTDS.dit theft uses a dummy file** on a throwaway DC VM.
- **Help-desk social engineering is a tabletop**, never performed against real people or a real help desk.
- Use an **isolated dev tenant and lab host**; never a production tenant or real identities.

---

*Range guide generated 2026-10-16 · accompanies the Scattered Spider trainer · identity logs real (dev tenant),
hypervisor artifacts self-generated, ransomware/NTDS.dit emulated with stand-ins; tools credited in the trainer
README.*
