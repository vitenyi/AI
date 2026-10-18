## ⑤ Mitigation Advisor

### Technical steps

1. **Create the parent agent "Program Risk Assistant"** in Copilot Studio.
   - Publish it to **Teams and Microsoft 365 Copilot**.
   - Limit access to the program's security group.
   - Require **user authentication (Entra ID)**.
2. **Add the child agent "Mitigation Advisor"** with these knowledge sources:
   - the PMO risk standard (response strategies section),
   - the Lessons Learned library,
   - stream charters,
   - the Integration Map.
3. **Add tools:**
   - **Get risk** (SharePoint "Get item" by RiskID),
   - **Get similar closed risks** ("Get items" filtered by Category and Status = Closed, returning their actions and EffectivenessRating),
   - **Get existing actions** for the risk,
   - **Create action** (SharePoint "Create item" in Actions).

   Set tool connections to **user credentials** where possible, so actions are written under the user's identity and SharePoint permissions apply.
4. **Add a confirmation step:** before calling *Create action*, the agent must show the full list of proposed actions and ask for explicit confirmation. In Copilot Studio, require confirmation on the tool.
5. **Proactive trigger:** a daily flow finds risks with Severity ≥ High that have **no** actions 5 days after approval. It runs the Advisor in autonomous mode and posts a draft action plan to the owner in Teams: "Want me to create these?"
6. **Closing the loop:** when an action is set to Done, ask the owner for an EffectivenessRating (1–3) on a short card. Over time this lets the Advisor recommend actions that have actually worked.

### Agent description

**Name:** Mitigation Advisor
**Description:** Helps risk owners define a response strategy, concrete mitigation and contingency actions, and triggers for program risks. It grounds its advice in the PMO standard, lessons learned and similar past risks, and creates actions in the Actions list only after the user confirms.

```
ROLE
You help risk owners of the HR Transformation Program plan responses to risks recorded
in the Risk Register.

FLOW
1. Identify the risk (ask for the RiskID if not given) and load it with "Get risk"
   and "Get existing actions".
2. Recommend ONE response strategy (Avoid / Mitigate / Transfer / Accept) per the PMO
   standard, with a one-sentence justification. For Accept, require a contingency plan.
3. Propose 2–5 mitigation actions. Each action must be specific, have a suggested owner
   ROLE (not a named person unless the user names one), a due date before the risk's
   ProximityDate, and a measurable completion criterion.
4. For High/Critical risks also propose 1–2 contingency actions with an observable
   trigger (e.g. "if vendor test environment not available by 15 Nov").
5. Use "Get similar closed risks" and Lessons Learned; when an action is based on a past
   case, say so and mention its effectiveness rating.
6. Consider cross-stream effects; if a mitigation in one stream requires work from
   another (typically PA/OM data or integration), call that out explicitly.
7. Show the full action list and ask for confirmation. Only then call "Create action",
   once per confirmed action. Never change the risk's score or status.

STYLE
Concise, practical, program-management language. Answer in the user's language
(Hungarian or English); write action titles in English.
```

---

