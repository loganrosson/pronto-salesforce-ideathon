# Pronto: Account Takeover Response on Salesforce

**Salesforce IDEAthon, David Eccles School of Business, University of Utah (Fall 2026)**
Team Cloud 9: Logan Rosson, Logan Riley, Jung Ko

Our challenge was built around Pronto, an online ordering marketplace. In one week we built a Salesforce solution for one problem: when a customer's account is hijacked, how fast does a real person on the Trust & Safety team pick it up, and what protects the customer in the meantime?

We built three connected pieces in a Salesforce Developer org:

1. **A customer website** (Experience Cloud) with a no-login form for reporting a hacked account.
2. **An Agentforce service agent** that answers order questions and hands suspected takeovers to Trust & Safety.
3. **Back-office automation** (10 flows, custom Case fields, a queue, reports and a dashboard) that routes, prioritizes, protects and times every account-takeover case.

> The Developer org this ran in is being retired, so this repo keeps a record of what we built through screenshots taken from the live org on October 2, 2026.

---

## How it works

```mermaid
flowchart LR
  A[Customer: website report form] --> C[Account Security Case]
  B[Customer: Agentforce chat] --> C
  C --> D[Priority = Critical]
  C --> E[Refunds frozen]
  C --> F[Confirmation email to customer]
  C --> Q[Trust & Safety queue]
  Q --> R[Rep accepts]
  Q -->|15 min| S[Alert + flag]
  Q -->|30 min| T[Escalate + email managers]
  R --> DB[Dashboard: time to accept, waiting cases, refunds protected]
```

---

## 1. Customer website (Experience Cloud, LWR)

| Home page | Account Security page |
|---|---|
| ![Home](01-site-home.png) | ![Account security](02-account-security-page.png) |

**No-login report form.** A customer who is locked out can still report. The form is a screen flow embedded on the site, and every submission creates a Case.

![Report form](03-no-login-report-form.png)

![What happens after you report](04-report-process-timeline.png)

---

## 2. Agentforce: Pronto Service Agent

One Agent Router greets the customer and hands off to one of six subagents: **Account Security, Order Issues & Refunds, Storefront Search, Ambiguous Question, Escalation, Off Topic**.

![Agentforce Builder](05-agentforce-builder-subagents.png)

| Order question (live chat on the site) | Hacked-account message |
|---|---|
| ![Order chat](06-agent-order-question.png) | ![Hacked chat](07-agent-hacked-account.png) |

The agent asks for the order email before sharing anything, never asks for a password, card number or code, and calls the **Agent: Escalate Account Takeover** flow to open a Critical case for Trust & Safety. Here is a case the agent created on its own (Origin = Agentforce, created by the agent user, refunds already frozen):

![Case created by Agentforce](26-case-created-by-agentforce.png)

---

## 3. Automation: 10 custom flows

![Flows list](09-custom-flows-list.png)

| Flow | Type | What it does |
|---|---|---|
| **Site: Report Account Takeover** | Screen flow (site) | Customer picks what happened and enters name, email, phone and details; creates an Account Security case (Origin = Web) and shows the case number. |
| **Case: Route Account Security to Trust and Safety** | Record-triggered (create) | Looks up the Trust & Safety queue and assigns the case to it. |
| **Case: Set Critical for Unauthorized Orders** | Before-save | Sets Priority = Critical when the reason is Unauthorized Orders. |
| **Case: Freeze Refunds on Account Security** | Record-triggered (create) | Sets *Refunds Frozen* so an attacker can't redirect refunds. |
| **Case: Trust & Safety Acceptance Alerts** | Record-triggered + scheduled paths | Alerts the queue right away; at 15 min, if nobody accepted, alerts again and flags *Not Accepted in 15 Min*; at 30 min marks the case Escalated and emails managers. |
| **Case: Stamp Accepted At** | Record-triggered (update) | Records the moment a rep takes the case, which feeds *Minutes to Accept*. |
| **Case: Sync Escalated Flag with Status** | Record-triggered | Keeps the Escalated checkbox and Status in step. |
| **Case: Secure Account** | Screen flow (record page action) | Rep locks the account down from the case; stamps *Account Locked At*. |
| **Agent: Escalate Account Takeover** | Autolaunched (Agentforce action) | Creates the ATO case from chat and returns the case number to the agent. |
| **Case: Email Report Confirmation** | Record-triggered (create) | Emails the customer "Thank you, {name}… your case number is {number}" for web reports. |

<details>
<summary><b>Flow canvases (click to expand)</b></summary>

**Site: Report Account Takeover**
![](10-flow-site-report-account-takeover.png)

**Case: Route Account Security to Trust and Safety**
![](11-flow-route-to-trust-and-safety.png)

**Case: Set Critical for Unauthorized Orders**
![](12-flow-set-critical-unauthorized-orders.png)

**Case: Trust & Safety Acceptance Alerts** (overview and detail)
![](13-flow-acceptance-alerts-overview.png)
![](14-flow-acceptance-alerts-detail.png)

**Case: Stamp Accepted At**
![](15-flow-stamp-accepted-at.png)

**Case: Sync Escalated Flag with Status**
![](16-flow-sync-escalated-flag.png)

**Case: Freeze Refunds on Account Security**
![](17-flow-freeze-refunds.png)

**Case: Secure Account**
![](18-flow-secure-account-screen.png)

**Agent: Escalate Account Takeover**
![](19-flow-agent-escalate-account-takeover.png)

**Case: Email Report Confirmation**
![](20-flow-email-report-confirmation.png)

</details>

---

## 4. Case data model and queue

Custom Case fields we added: **Refund Amount at Risk, Refunds Frozen, Account Locked At, Minutes to Lockdown** (formula), **Security Notes, Accepted At, Minutes to Accept** (formula), **Not Accepted in 15 Min**, plus the Escalated flag.

![Case record](22-case-record-custom-fields.png)
![Case status and web email](23-case-record-status-and-web-email.png)

**Trust & Safety queue**, where every account-security case lands:

![Queue](25-trust-and-safety-queue.png)

---

## 5. Reporting: Trust & Safety Acceptance dashboard

Shows average minutes to accept, cases waiting now, acceptance-time buckets (within 15, 15 to 30, over 30 minutes), cases by reason and channel, refund dollars protected and minutes to lockdown.

![Dashboard](21-dashboard-trust-and-safety.png)

![Acceptance time buckets report](24-report-acceptance-time-buckets.png)

*Note: all cases are test data created by our team. Two cases labeled "SAMPLE DATA" had their accept times edited so the 15-to-30 and over-30-minute buckets show up on the chart.*

---

## Tools used

Salesforce Service Cloud · Experience Cloud (LWR) · Agentforce (Agent Builder, subagents, flow actions) · Embedded Messaging · Flow Builder (screen, record-triggered, scheduled paths, autolaunched) · Queues · Formula fields · Reports & Dashboards · Email actions
