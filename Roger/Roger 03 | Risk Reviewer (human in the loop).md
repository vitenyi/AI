## ③ Review gate (human in the loop)

### Technical steps

1. **Trigger:** an agent flow on *"When an item is created or modified"* in Risk Staging. Add a trigger condition: AssessmentStatus = Done and ReviewStatus = Pending.
2. **Find the reviewer:** look up the stream lead for the item's Stream in Config. For Cross-stream items, the PMO risk manager reviews.
3. **Send the card:** use *Microsoft Teams – "Post adaptive card and wait for a response"* as a chat with the reviewer.

   The card shows: title, cause/event/effect, evidence quote, source link, AI confidence, proposed P/I with rationale, and the suspected duplicate (with a link).

   The reviewer can **edit** these fields on the card (`Input.Text` / `Input.ChoiceSet`): title, P, I, owner, category.

   Buttons: **Approve**, **Merge into RSK-xxx**, **Reject** (with a reason dropdown: not a risk / duplicate / out of scope / too vague).
4. **Handle the response:**
   - **Approve (New):**
     - create the Register item and write the RiskID;
     - set Status = Open and LastReviewedOn = today;
     - store AI-proposed vs final P/I in Staging (this feeds the acceptance KPI);
     - notify the owner.
   - **Approve (Update):** update the target risk's fields and add an entry to the risk's comment history: "Updated from <source>: …".
   - **Approve (Closure):** set Status = Closed and record the ClosureReason.
   - **Merge:** attach the new evidence and source link to the existing risk. Then create an Update proposal if the reviewer changed P or I.
   - **Reject:** store the RejectReason. Also post a weekly summary of reject reasons to the PMO, which you'll use to tune the Scout.
5. **Second approval for high scores:** if the final Severity is at or above the escalation threshold, or if the proposal is a Closure of a High/Critical risk, send a second card to the PMO risk manager. Write to the Register only after both have approved.
6. **Timeouts:**
   - Set the card wait timeout to 3 working days.
   - On timeout, send one reminder.
   - After 5 days, reassign to the PMO and log it. Also keep in mind that a flow run can't wait for more than about 30 days, so don't leave items in an open-ended wait.
7. **Batching option:** if stream leads get too many cards, switch to a single daily digest card per reviewer listing all pending items. This needs a scheduled flow plus a card template that repeats per item. Start with one card per item in the pilot.

*There's no agent description here on purpose. This step contains no AI.*

---

