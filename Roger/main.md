Before I design the solution, a few answers will shape the architecture, mainly which Microsoft AI licences you can use and where your risks surface today.

Your answers point to a clear design: a **Microsoft-native setup built on Copilot Studio, SharePoint and Power BI**, with no pro-code Azure layer. Agents capture risks from Teams meetings and status reports and central sources, write candidates to staging, and surface them for human review.

## 1. Design principles

- **Agents propose, people decide.** No agent writes directly to the official register, rescores a high risk, or closes a risk. Everything goes through a staging step that a stream lead approves.
- **One source of truth.** A SharePoint risk register, with the PMO framework stored in configuration lists rather than in prompts.
- **Every AI suggestion cites its source**: a meeting, an email or a document link. This keeps the work auditable and catches hallucinations early.
- **Cross-stream by default.** In an HR program, most serious risks involve dependencies between streams. PA/OM feeds almost everything else.

## 2. SharePoint foundation ("Program Risk Hub" site)

| List / Library | Purpose |
|---|---|
| **Risk Register** | ID, title, *cause → event → effect* description, stream, category, owner, probability, impact, score (calculated), proximity, trend (↑/→/↓), status, response strategy, notes |
| **Risk Staging** | AI-captured candidates: proposed text, proposed score plus rationale, confidence, source, suspected duplicate ID, reviewer decision |
| **Actions** | Linked to a risk: action, owner, due date, status, effectiveness |
| **Config lists** | Your PMO scales, categories, escalation thresholds, stream ↔ owner mapping. Agents read these, so changing a rule never requires re-prompting |
| **Knowledge library** | PMO risk standard, stream charters, integration/interface map, lessons learned. Used as agent grounding |

Turn on versioning for the audit trail. Index the stream, status and owner columns so views stay fast as the list grows.

## 3. The agent setup (Copilot Studio)

I'd build **one user-facing "Program Risk Assistant"** in Teams, plus specialist sub-agents and background flows. That way the team has a single entry point instead of five bots.

**① Risk Scout (capture)**
- **Teams meetings:** after stream meetings and SteerCo, a flow pulls the transcript or recap. The agent extracts risk signals such as "we're waiting on…", "unless…" or "vendor hasn't confirmed…" and turns them into candidate risks.
- **Status reports and email:** stream leads send reports to a shared mailbox, or drop them in a library. The agent extracts candidate risks from each one.
- Each candidate is written as cause → event → effect. The agent checks it against the register for duplicates, then writes it to **Staging** with the source link.

**② Risk Assessor (rating)**
- Proposes probability, impact, category and proximity using your PMO scales from the config lists.
- Adds a short written rationale for the scores.
- Flags downstream streams. For example: "delay in position management design → affects Recruiting requisitions, Talent succession, L&D assignment rules."

**③ Review gate (human in the loop)**
- The stream lead receives a Teams Adaptive Card with four choices: **Approve / Edit / Merge with RSK-0xx / Reject**.
- Risks above your escalation threshold also go to the PMO risk manager.
- Reject reasons are logged, which helps you tune the agent's instructions later.

**④ Risk Steward (maintenance)**
- Runs weekly. It finds stale risks (e.g. not reviewed in 14 days), overdue actions and approaching proximity dates.
- It nudges owners in Teams with a card: "Still valid? Score changed? Action progress?"
- When new transcripts or reports mention an existing risk, it proposes updates such as a trend change, a rescore or a possible closure. These also go through the review gate.

**⑤ Mitigation Advisor**
- On request ("help me with RSK-042"), it proposes a response strategy (avoid, mitigate, transfer or accept).
- It drafts concrete actions with suggested owners and dates, plus contingency triggers.
- It draws on the lessons learned library and on similar closed risks.

**⑥ Reporting agent:** see the next section.

## 4. Reporting by audience

| Audience | Format | Content |
|---|---|---|
| **Stream teams** | SharePoint views + weekly Teams digest | My risks, my overdue actions, new items awaiting review |
| **Program management / PMO** | Power BI dashboard | Heat map, trend over time, ageing, action burn-down, cross-stream dependency view, capture-source mix |
| **SteerCo** | One-page pack (PowerPoint template, drafted by Copilot) | Top 5–10 risks, what changed since last time, **decisions required**, escalations |
| **Sponsor / Exec** | Short narrative (email or Teams) | Overall RAG, 3 key exposures, asks |

The key point: all four formats are generated from the **same register data**. Only the level of aggregation and the narrative change, so the numbers never contradict each other.

## 5. Program-specific risk themes to seed the taxonomy

- **Multi-vendor landscape:**
  - integration/middleware timing
  - master data consistency, with org/position as the "golden record"
  - identity provisioning (Entra ID)
  - misaligned vendor release cycles
- **PA/OM as the critical path:**
  - org structure and job architecture decisions block Recruiting, Skills, Talent and L&D
  - model this dependency explicitly in the register
- **Skills:** taxonomy ownership and governance, plus mapping skills to jobs.
- **Data protection and employee relations:**
  - GDPR for candidate and employee data
  - works council consultation on new HR processes **and on meeting transcription itself**
- **EU AI Act:** AI used in recruiting, promotion or performance decisions counts as *high-risk* under the Act. If any vendor module uses AI for screening, promotion, or performance decisions, that is a compliance risk to track.

## 6. Governance and controls

- Get works council and privacy sign-off **before** processing Teams transcripts. HR program discussions often mention named employees.
- Apply Purview sensitivity labels to the hub and DLP policies to the agent.
- Restrict the agent's knowledge sources to the Risk Hub, so it can't pull in random tenant content.
- Plan Copilot Studio capacity. Autonomous triggers (meeting-end, mailbox) consume messages or credits, so monitor usage in the first month.
- Confirm with your Microsoft licensing contact that Copilot Studio covers the flows you need. Premium connectors are generally covered when the agent calls them, but standalone scheduled flows may not be.

## 7. Phased rollout

1. **Foundation (2–3 weeks):** SharePoint lists, config lists encoding your PMO standard, migration of the existing register, Power BI dashboard.
2. **Capture + assess (3–4 weeks):** Risk Scout on status reports and the mailbox, the Assessor, and the review cards. Pilot with **one stream** (I'd pick PA/OM).
3. **Teams transcripts + Steward (3 weeks):** once privacy and works council approval is in place, roll out to all streams.
4. **Mitigation Advisor + automated SteerCo pack (3 weeks).**

**KPIs to track:**
- acceptance rate of AI-proposed risks (aim for over 60% after tuning)
- duplicate rate
- median risk age since last review
- share of overdue actions
- PMO hours spent on report preparation

---

I can make any of these concrete next: the SharePoint list schema (columns, types, formulas), the Copilot Studio agent instructions and extraction prompt, the Adaptive Card design, or this whole proposal converted into a deliverable pack.
