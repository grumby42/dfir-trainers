# Cloud Identity — build-a-range (hands-on next step)

The trainer builds the *how to think*. This builds the *hands-on* — and this is the **most reproducible case in
the repo**. A free **Microsoft 365 Developer tenant** gives you a real Entra + Exchange Online environment, so
you generate **genuine** sign-in, consent, app-role, and mailbox-access logs by emulating each step safely. You
reproduce the *investigative artifacts*, never a live attack: no real victim mailbox is stolen, no attacker
infrastructure is stood up.

> **Difficulty: Easy–medium.** **Compute: none (cloud tenant + a browser).** **Cost: $0.**
> **Fidelity: high — the logs you hunt are real, produced by your own actions.**

---

## What you'll need (all free)

- A **Microsoft 365 Developer tenant** (free, renewable) — real **Entra ID** (sign-in / audit logs), **Exchange
  Online**, and the **app registration / consent** machinery. This is the whole lab.
- A second browser profile / VM (or a phone on cellular) to act as the "different source."
- Optional aggregation: **Microsoft Sentinel free tier** / **Log Analytics**, or just export the logs to CSV.

Pull real IOCs from the MSTIC / CISA reporting (linked in the trainer README) — though note MSTIC says IOCs are
low-value here, which is exactly the point.

---

## Step 1 — Play the scenario (safe emulation, in order)

Do each step as an **admin in your own test tenant**, against **test accounts and dummy mailboxes you created**:

1. **Password spray → weak account.** Create a legacy-style test user, **exclude it from MFA / conditional
   access**, give it a weak password. From a second source, sign in after a few deliberate failures to produce
   the **`50126` failures → success, `singleFactor`, CA `notApplied`** pattern in the sign-in logs.
2. **Register a malicious-stand-in OAuth app.** Create an app registration ("Mail Reporting Connector"),
   generate a **client secret** (this is your D2 artifact), and create a throwaway **consent user**.
3. **Grant the high-privilege app role.** Grant the app **`full_access_as_app`** (or, to stay lighter,
   `Mail.Read` application permission) via **admin consent** — this is your D1 artifact. Capture the
   *"Consent to application" / "Add app role assignment"* audit event.
4. **Authenticate app-only and read a DUMMY mailbox.** Using the app's own token, call Graph and/or EWS to read
   a **test mailbox you own** — producing app-only **`MailItemsAccessed`** and **service-principal sign-in**
   records (D3). Read only your own dummy mailboxes; never anyone's real mail.
5. (Optional) **Add `offline_access`** to an app's consented scopes to see the D4 signal.

## Step 2 — Collect

Export **Entra sign-in logs** (interactive + non-interactive + **service principal**), **Entra audit logs**
(consent, app-role assignment, credential add, user create), and **Exchange `MailItemsAccessed`**. Ship to
Sentinel/Log Analytics or CSV.

## Step 3 — Hunt and verify against ground truth

- Does the **spray pattern** (`50126` low-and-slow → one success on the no-MFA/CA-excluded account) reconstruct
  from the sign-in logs?
- Do the **four DETECTIONS.md analytics** (D1 full_access_as_app admin consent; D2 new SP credential; D3 app
  reading mail; D4 offline_access/mail consent to an unknown app) fire on the events **you** generated?
- Can you see the **service-principal sign-ins** as a separate stream from user sign-ins — and note that no
  endpoint/EDR saw any of it?

## Step 4 — Practice the discipline

Run the walkthrough's sufficiency spine on your own data: spray-success ≠ app compromise ≠ mailbox collection.
Then write the **coordinated identity cut** (disable account + revoke sessions, remove app + revoke tokens,
block consent user — together) and the **fabric eradication** (strip app roles, rotate secrets, enforce MFA/CA
everywhere, kill legacy auth). Finally, state attribution as **"consistent with"** and record what you ruled
out — exactly the exec-legible call an advisory engagement produces.

---

## Where to stop (the emulation boundary)

- **Your own test tenant only** — never a production tenant, never real identities, never anyone else's mailbox.
- **Read only dummy mailboxes you created.** The exercise is to generate the *telemetry*, not to exfiltrate mail.
- **No real attacker infrastructure / residential proxies.** "Different source" = a second browser profile or
  your phone; the lesson (IOC detection is futile, hunt the TTP) stands without it.
- Use app permissions scoped as narrowly as you can while still producing the signal; **remove the app and
  revoke the secret** when done.

---

*Range guide generated 2026-10-16 · accompanies the Cloud Identity trainer · logs real (your own M365 Dev
tenant), all actions self-generated against dummy accounts/mailboxes; no live attack infrastructure. Tools and
reports credited in the trainer README.*
