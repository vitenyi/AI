## ④ Risk Steward (maintenance)

### Technical steps

1. **Weekly snapshot flow**, scheduled every Monday at 06:00 Europe/Budapest:
   - copy all non-closed risks into Risk History;
   - calculate Trend by comparing each risk's current score with its snapshot from 4 weeks ago, and write the result to the Register.
2. **Weekly hygiene flow**, Monday 07:00. It runs these queries:
   - **Stale:** open risks whose LastReviewedOn is older than the PMO review cycle (e.g. 14 days; read it from Config).
   - **Overdue or soon-due actions:** DueDate within the next 7 days, or already overdue with Status ≠ Done.
   - **Proximity passed:** ProximityDate < today and the risk is still Open. Either it happened (it's now an Issue) or the window has passed (a closure candidate).
3. **Group by owner** and send each owner **one** Adaptive Card. For each risk, the owner can choose:
   - Still valid: yes / no
   - New P/I (optional)
   - Trend comment
   - Mark as issue
   - Propose closure

   Each action on the card has these options: Done / New due date / Blocked.
4. **Handle the response:**
   - Confirming the risk, or changing P/I **within the same Severity band**, writes directly to the Register and sets LastReviewedOn.
   - A change that **moves the risk to another Severity band**, a closure, or conversion to an Issue becomes a Staging proposal and goes through ③.
5. **Evidence-based update proposals (AI part):**
   - Whenever the Scout creates a staging item with a SuspectedDuplicateID, the Steward prompt compares the new evidence with the existing risk.
   - It produces an Update proposal with a rationale, e.g. "Vendor confirmed the integration date slipped by 3 weeks → probability 3→4". This goes to ③.
6. **Escalation reminders:** two working days before each SteerCo date (from the Reporting Calendar), remind owners of risks with EscalationLevel = SteerCo to fill in DecisionRequired.

### Prompt description: "Steward – evidence update"

```
You compare NEW EVIDENCE about an existing program risk with its CURRENT REGISTER ENTRY.
Decide one of: NoChange | UpdateAssessment | ProposeClosure | ConvertToIssue.
- UpdateAssessment: propose new Probability/Impact using {ScaleDefinitions}; explain the
  change in max 2 sentences citing the new evidence.
- ProposeClosure: only if evidence shows the cause is removed or the risk window passed.
- ConvertToIssue: only if evidence shows the event has occurred.
- If evidence is ambiguous, choose NoChange and say what information is missing.
Return JSON: decision, proposedProbability, proposedImpact, rationale, missingInfo.
Never modify the register; your output is a proposal for human review.
```

---

