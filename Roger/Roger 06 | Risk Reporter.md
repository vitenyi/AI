## ⑥ Reporting agent

### Technical steps – data and dashboard layer

1. **Build a Power BI semantic model** on the SharePoint lists, using the *SharePoint Online list* connector with the 2.0 implementation. Load: Register, Actions, Risk History, Staging, Config, Reporting Calendar.
   - Key measures: Open risks by Severity; New / Closed in period; Score change versus the last SteerCo; Average age; % stale; Overdue actions; Top-N by score; AI acceptance rate (Approved ÷ all reviewed staging items); Rejection reasons.
2. **Schedule refresh** for after the Monday snapshot, plus once a day. Pro allows up to 8 refreshes a day; Premium allows more.
3. **Report pages:**
   - **Heat map**: 5×5 matrix visual of P × I with risk counts, click through to the list.
   - **Trend**: severity distribution over time, from Risk History.
   - **Ageing and hygiene**: stale risks, overdue actions, owners with the most open items.
   - **Cross-stream dependencies**: matrix of Stream × AffectedStreams.
   - **Stream drill-down**: one page per stream, with a slicer.
   - **AI pipeline**: captured → approved / merged / rejected, by source type. This tells you whether the Scout is earning its keep.
4. **Publish to a program workspace.** Embed the report as a tab in the program Teams channel and on the SharePoint hub.

### Technical steps – audience outputs

5. **Stream digest** (team level): a weekly flow per stream lead posts a Teams message or HTML email with:
   - "your open risks",
   - "actions due or overdue",
   - "items waiting for your review",
   - a link to the filtered Power BI page.

   No AI is needed; it's a flow with an HTML table.
6. **Add the child agent "Reporting Assistant"** to the Program Risk Assistant. Its tools:
   - **Get top risks** (Register, sorted by score, filtered to open, top N),
   - **Get changes since date** (History compared with the current state: new, closed, severity changed),
   - **Get escalations** (EscalationLevel = SteerCo, including DecisionRequired),
   - **Get KPIs** (counts and overdue actions from the lists),
   - **Create SteerCo document** (an agent flow using *Word Online (Business) – "Populate a Microsoft Word template"*; this is a premium connector, so confirm it's covered by Copilot Studio capacity),
   - **Create Outlook draft** (for the exec summary; it creates a **draft only** and never sends).
7. **Word template for SteerCo**, in the corporate design. Content controls:
   - DataAsOf, OverallRAG
   - Top risks table: ID / title / severity / trend / owner / key action
   - Changes since last SteerCo
   - Decisions required
   - Narrative
8. **SteerCo slides:** the PMO opens the generated Word file in **Copilot in PowerPoint** ("Create presentation from file") with the corporate template, then adds the Power BI heat map.
   - To automate the image, use Power BI *Export to File* (needs Premium/Fabric capacity) to export the heat map page as PNG and place it in the Reports library.
   - Full end-to-end PPT generation is possible, but a 10-minute human finishing step is the better trade-off for a SteerCo audience.
9. **Consistency rule:** every output carries the same **"Data as of <snapshot date>"** stamp and is built from the Monday snapshot. That way the dashboard, the SteerCo pack and the exec email never show different numbers.
10. **Store and approve:** save generated documents in a "Risk Reports" library with Status = Draft. The PMO sets them to Final before distribution.

### Agent description

**Name:** Reporting Assistant
**Description:** Produces audience-specific risk reports for the HR Transformation Program: stream summaries, PMO analyses, SteerCo packs and executive summaries. It uses data from the Risk Register and the weekly snapshot, and creates drafts only. Distribution remains a human decision.

```
ROLE
You prepare risk reporting for the HR Transformation Program. Numbers ALWAYS come from
your tools (Get top risks, Get changes since date, Get escalations, Get KPIs); never
estimate, round differently or invent figures. State "Data as of <snapshot date>" in
every output.

AUDIENCES AND FORMATS
- Stream: list of the stream's open risks, overdue actions, pending reviews. Factual,
  table-oriented.
- PMO: full analysis — severity distribution, changes, ageing, cross-stream
  dependencies, hygiene issues, AI pipeline KPIs. Point to the Power BI report for detail.
- SteerCo: fill the SteerCo Word template. Top 5–10 risks by score; what changed since
  the last SteerCo date (from Reporting Calendar); escalations with the specific
  DECISION REQUIRED, options and recommendation; max 150-word narrative. Business
  language, no technical jargon.
- Executive: max 150 words — overall RAG, the 3 most material exposures in one line
  each, and explicit asks. Create as an Outlook DRAFT addressed per Reporting Calendar.

RULES
- Lead with what changed and what decision is needed, not with process detail.
- Do not include personal data about employees discussed in HR process context;
  refer to roles.
- Every output is a draft; tell the user where it was saved and that PMO must approve it.
- Answer in the user's language; documents follow the template language.
```

---

## Build order and testing

1. **Week 1–2:** the SharePoint extensions, the "Recalculate severity" and RiskID flows, and the Review gate (③). You can test the review gate with manually created staging items before any AI is involved.
2. **Week 3:** the Risk Assessor (②) and its calibration against historical risks.
3. **Week 4–5:** the Risk Steward (④): snapshot, hygiene cards, evidence updates.
4. **Week 5–6:** the Power BI model and the stream digests (first part of ⑥).
5. **Week 7–8:** the Program Risk Assistant with the Mitigation Advisor (⑤) and the Reporting Assistant (⑥). Do a SteerCo dry run with the PMO in parallel to the manual pack.

**Testing per component:**
- **Assessor:** score agreement with the PMO.
- **Review gate:** every branch (approve, merge, reject, timeout, second approval).
- **Steward:** run it against a copy of the register with dates seeded to trigger each rule.
- **Advisor:** 10 real risks reviewed by risk owners for usefulness.
- **Reporting:** reconcile the numbers against Power BI to the exact figure before the first live SteerCo.

---

Next, I can produce the complete Adaptive Card JSON for the review gate and the Steward card. I can also turn this and the Risk Scout spec into one implementation document for your team.



