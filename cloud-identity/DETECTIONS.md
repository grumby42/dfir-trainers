# Detection engineering appendix — Cloud Identity (Midnight Blizzard / APT29)

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

Starters, not drop-in rules. The point is to teach **authoring**: each detection below is keyed on a step the
actor **cannot skip**, with notes on how to baseline it, how to evolve it, and what trend signal to watch. All
four map to the behavior-based detections Microsoft MSTIC recommends in its responder guidance — deliberately
**not** IP/geo rules, because the actor's residential proxies make indicator-based detection infeasible.

Data sources assumed: **Entra ID sign-in logs** (interactive + non-interactive + **service principal**),
**Entra audit logs** (consent, app-role assignment, credential changes, directory adds), and **Exchange Online
mailbox-access logs** (`MailItemsAccessed`). In Sentinel these are `SigninLogs`,
`AADNonInteractiveUserSignInLogs`, `AADServicePrincipalSignInLogs`, `AuditLogs`, and `OfficeActivity` /
`MailItemsAccessed`.

---

## D1 — An app granted Exchange `full_access_as_app` (especially via admin consent)

```
AuditLogs
| where OperationName in ("Consent to application", "Add app role assignment to service principal")
| where <permission/appRole> has_any ("full_access_as_app", "ApplicationImpersonation", "EWS.AccessAsUser.All", "Mail.Read", "Mail.ReadWrite")
| where ConsentType == "AllPrincipals"   // tenant-wide admin consent
| project TimeGenerated, InitiatedBy, TargetApp=<appDisplayName>, Permission=<value>
```
- **Why:** this is the single grant that turns a foothold into a mail-collection platform.
- **Baseline / tune:** enumerate the *legitimate* apps that already hold these roles (there should be very few,
  and they should be **scoped to specific mailboxes** via Exchange RBAC for Applications). Allowlist those app
  object IDs; alert on anything new.
- **Evolve it:** correlate the grant with **who** granted it — a brand-new or rarely-used principal granting
  tenant-wide mail access is far more suspicious than a known admin.
- **Trend signal:** count of apps holding `full_access_as_app` over time should be **flat and small**. Any
  increase is an audit lead.

## D2 — A new secret or certificate added to an existing service principal

```
AuditLogs
| where OperationName in ("Add service principal credentials", "Update application – Certificates and secrets management")
| project TimeGenerated, InitiatedBy, TargetApp=<appDisplayName>, CredType=<keyType>
| join kind=leftouter (/* known change-management / CI identities */) on InitiatedBy
```
- **Why:** quieter than registering a brand-new app — the actor adds their own credential to an app that already
  has rights, and now authenticates as it.
- **Baseline / tune:** credential rotation is normal; **who** does it is the signal. Allowlist your secret-
  rotation automation and admins; alert on credential adds by users/principals outside that set.
- **Evolve it:** chain D2 → D3 (did the app then read mail from an unusual source?).
- **Trend signal:** spikes in credential additions, or additions clustered right before anomalous app sign-ins.

## D3 — An application reading mail via **both** Graph and EWS

```
let graphMail = /* app-only Graph calls to /messages, /mailFolders */;
let ewsMail   = /* app-only EWS access in MailItemsAccessed / OfficeActivity */;
graphMail | join kind=inner (ewsMail) on AppId
| summarize MailboxesTouched=dcount(Mailbox), Protocols=make_set(Protocol) by AppId, AppDisplayName
| where MailboxesTouched > <baseline>   // bulk, unrelated mailboxes
```
- **Why:** a legitimate connector almost always uses **one** access path and a **scoped** set of mailboxes. An
  app reading across many unrelated mailboxes over app-only EWS is collection.
- **Baseline / tune:** learn each app's normal mailbox-count and protocol; alert on app-only access that spans
  many unrelated boxes (exec / legal / security are high-value tells).
- **Evolve it:** weight by **mailbox sensitivity**, not just count — four C-suite/IR mailboxes beats forty
  shared calendars.
- **Trend signal:** `dcount(mailbox)` per app, week over week.

## D4 — Consent to an unknown app requesting `offline_access` + mail scopes

```
AuditLogs
| where OperationName == "Consent to application"
| where <scopes> has "offline_access" and <scopes> has_any ("Mail.Read", "Mail.ReadWrite", "EWS.AccessAsUser.All")
| where TargetApp !in (<known app allowlist>)
```
- **Why:** `offline_access` buys a refresh token — durable access that survives the originating session.
- **Baseline / tune:** maintain an allowlist of sanctioned apps; route consent through an **admin-consent
  workflow** so new grants are reviewed, not silent.
- **Evolve it:** alert harder when the consenting user was **created recently** or has no other activity.
- **Trend signal:** rate of consents to first-seen apps.

---

## Cross-cutting

- **Detect on the TTP, not the IP.** Residential proxies defeat address-based rules by design — every detection
  above keys on an action the actor must take inside your tenant.
- **Watch service-principal sign-ins as their own stream.** They are not in the user sign-in logs most teams
  monitor; `AADServicePrincipalSignInLogs` is where app-only auth shows up.
- **Hunt the eradication, too.** After containment, alert if any removed app, credential, or consent
  **reappears** — re-consent is the actor's comeback path.
- **Validate before you trust.** Generate each behavior safely in a test tenant (see `RANGE.md`) and confirm
  your rule fires on your own telemetry before relying on it.

*Detections appendix generated 2026-10-16 · accompanies the Cloud Identity trainer · behavior-based, derived
from MSTIC responder guidance; verify against your own data.*


## Further reading (from the DFIR bookshelf)

Methodology and authoring discipline these plays draw on:

- Bollinger, Enright & Valites — *Crafting the InfoSec Playbook* (O'Reilly) — detection-as-plays.
- Don Murdoch — *Blue Team Handbook: SOC, SIEM, and Threat Hunting Use Cases* — use-case authoring + tuning.
- Scott J. Roberts & Rebekah Brown — *Intelligence-Driven Incident Response* (O'Reilly) — F3EAD + Diamond, the intel loop these trainers run on.
- Steve Anson — *Applied Incident Response* (Wiley) — evidence acquisition, host/network artifacts, lateral-movement detection.
