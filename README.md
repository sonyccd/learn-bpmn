# BPMN by Increments: A Modeling Curriculum

Fourteen exercises. Each one adds one or two concepts on top of everything before it. Model each in Camunda Modeler, then give the exercise text, an image of the diagram, and the stripped XML to the evaluator prompt at the bottom.

**The finish line:** if you can model all fourteen so they pass their test scenarios, you have working command of the core of BPMN: flow, gateways, pools and messages, every major event type, subprocesses, multi-instance, and compensation. What's left after that (choreography diagrams, conversation diagrams, a few rare event types) is rarely used in practice.

## How to use this

- **Outcome is what's graded.** A model passes if every test scenario reaches the right outcome when you trace it. There is rarely one right diagram. Going beyond the exercise, adding error handling or concepts from later exercises early, is fine as long as the extra parts also work.
- **One hard rule:** if an exercise names a concept in its constraints, use it. Solving Exercise 7 without an event-based gateway might work, but you skipped the thing you were there to practice.
- **Do them in order.** Later exercises assume you will reuse earlier concepts without being told to.
- **Test scenarios are your unit tests.** Trace a token through your diagram for each one before you submit. If you can't trace it, the model is wrong.
- **The scenarios are written like a stakeholder wrote them.** Some details are vague on purpose. When you hit an ambiguity, make a decision and write it down as a text annotation in the diagram.
- **Camunda version:** create diagrams as Camunda 8 (BPMN). The Modeler's lint panel flags things that won't *execute* (missing job types, missing forms). For pure modeling, ignore warnings about implementation properties. Don't ignore warnings about structure.
- **Submit two things: an image and the stripped XML.** The image shows the structure at a glance. The XML is ground truth for what the image can't show (conditions, timer values, interrupting vs. non-interrupting, multi-instance settings) and settles any icon the evaluator can't read.
  - **Image:** File > Export As > PNG or SVG. Use a high-resolution PNG or SVG for big diagrams. Small event icons get misread at low resolution.
  - **Stripped XML:** open the `.bpmn` file in a text editor, copy it, and in the copy delete:
    1. the whole `<bpmndi:BPMNDiagram> ... </bpmndi:BPMNDiagram>` block (drawing coordinates, no process meaning, often about half the file)
    2. any `<bpmn:extensionElements> ... </bpmn:extensionElements>` blocks (Camunda execution settings you aren't using)
  - Still label gateway outflows with their conditions and timers with their durations on the canvas. It makes the diagram readable to humans, which is half the point of BPMN.
- **Naming conventions** (the evaluator will enforce these from Exercise 1 on):
  - Tasks: verb + object ("Brew coffee", not "Coffee" or "Brewing")
  - Events: object + past participle state ("Order received", "Payment failed")
  - Exclusive/inclusive gateways: a question ("Order valid?"), outgoing flows labeled as answers
  - Pools: the participant. Lanes: a role, not a person's name.

---

## Exercise 1: Coffee Counter

**Concepts:** start event, end event, task, sequence flow, naming.

A customer walks up to a coffee shop counter. The barista takes their order, takes payment, makes the drink, and hands it over. That's it.

**Test scenarios**
1. A customer orders a latte and receives it.

**Constraints**
- One start event, one end event, no gateways.
- Name every element.

This one is trivially easy on purpose. The point is to get the save/export/evaluate loop working and to get naming right.

---

## Exercise 2: Pull Request Review

**Concepts:** exclusive gateway (split and merge), conditions on sequence flows, default flow, loops, task types (user task, service task).

A developer opens a pull request. Automated CI runs tests. If tests fail, the developer fixes the code and pushes again, which triggers CI again. If tests pass, a reviewer reviews the code. The reviewer either approves it, requests changes, or rejects it outright. Requested changes go back to the developer. Approved PRs are merged automatically. Rejected PRs are closed.

**Test scenarios**
1. Tests pass, reviewer approves first time. PR merged.
2. Tests fail twice, then pass, reviewer approves. PR merged.
3. Tests pass, reviewer requests changes, developer fixes, tests pass, reviewer approves.
4. Tests pass, reviewer rejects. PR closed.

**Constraints**
- Use user tasks for human work and service tasks for automated work.
- Every exclusive gateway with conditional outgoing flows must have a default flow or be provably exhaustive.
- Decide: when changes are requested, does CI re-run? (Annotate your answer.)

---

## Exercise 3: Restaurant Kitchen Ticket

**Concepts:** parallel gateway (fork and join), combining parallel and exclusive gateways correctly.

A ticket comes into the kitchen for a table: a steak, a salad, and fries. The grill cook, salad station, and fry station work at the same time. The expediter plates everything once all three are ready and sends it out. Some tickets also include dessert, which is prepared after the main course is served, but only if the table ordered dessert.

**Test scenarios**
1. Steak, salad, fries, no dessert. Served once all three are done.
2. Same order with dessert. Dessert prepared after mains go out.
3. The fries finish first and the steak finishes last. Plating waits for the steak.

**Constraints**
- Assume every ticket has exactly those three mains. (Variable items come later.)
- Do not join exclusive branches with a parallel gateway. The evaluator will check for this deadlock specifically.

---

## Exercise 4: Apartment Rental Application

**Concepts:** pool, lanes, data objects, data stores, text annotations.

A property management company processes rental applications. The leasing agent receives an application and checks it for completeness. Incomplete applications are returned to the applicant (just model it as a task for now). Complete applications go to the screening team, which runs a credit check and a background check in parallel and records results in the tenant screening database. The property manager reviews the results and approves or denies. The leasing agent notifies the applicant either way. Approved applicants get a lease drafted by the leasing agent.

**Test scenarios**
1. Complete application, clean checks, approved, lease drafted.
2. Incomplete application, returned.
3. Complete application, denied by property manager.

**Constraints**
- One pool (the company), three lanes.
- Show the application as a data object flowing through the process and the screening database as a data store.
- Decide: when an incomplete application is returned, does the process end or wait? Annotate.

---

## Exercise 5: Pizza Delivery (Collaboration)

**Concepts:** collaboration diagram, multiple pools, message flows, message start event, intermediate message throw/catch events, send and receive tasks, black-box (collapsed) pool.

A customer orders a pizza online. The pizza shop receives the order, bakes it, and sends a driver. The customer waits. If 45 minutes pass with no pizza... ignore that for now (next exercise). When the pizza arrives, the customer pays the driver and the driver returns to the shop. The payment processor is a third party whose internals you don't know.

**Test scenarios**
1. Customer orders, shop bakes, driver delivers, customer pays, process ends in both pools.

**Constraints**
- Customer and Pizza Shop are expanded pools with their own processes.
- Payment Processor is a collapsed (black-box) pool.
- Sequence flows never cross pool boundaries. Message flows always do.
- Every message flow must connect to something that can actually send or receive (a message event, a send/receive task, or a black-box pool).

---

## Exercise 6: On-Call Incident Response

**Concepts:** timer events (intermediate and boundary), interrupting vs non-interrupting boundary events.

A monitoring system fires an alert that pages the on-call engineer. The engineer acknowledges the page and investigates. If the engineer hasn't acknowledged within 5 minutes, the system re-pages them but keeps waiting. If they still haven't acknowledged within 15 minutes total, the page is escalated to the secondary on-call, and the primary is no longer expected to respond. After the incident is resolved, the team waits 24 hours and then holds a postmortem.

**Test scenarios**
1. Primary acks at 2 minutes. Investigates, resolves, postmortem after 24h.
2. Primary acks at 8 minutes. One re-page sent, primary handles it.
3. Primary never acks. Re-paged at 5, escalated at 15, secondary handles it.

**Constraints**
- Use ISO 8601 durations for timers (e.g. `PT5M`).
- Decide: does the 5-minute re-page repeat every 5 minutes, or fire once? Annotate.

---

## Exercise 7: Monthly Invoice Collection

**Concepts:** timer start event (cycle), event-based gateway, loop counters.

On the first of every month, a SaaS company sends invoices to customers. For each invoice (assume one customer for now), it waits for payment. If payment arrives, the invoice is marked paid. If 14 days pass with no payment, a reminder is sent and the clock starts again. After the third unpaid reminder, the account goes to collections and is suspended.

**Test scenarios**
1. Paid on day 3.
2. Paid on day 20 (after one reminder).
3. Never paid. Three reminders, then collections and suspension.

**Constraints**
- Use an event-based gateway, not an exclusive gateway, to wait for "whichever happens first."
- Model the reminder count somehow (gateway condition on a variable is fine). Annotate the variable name.

---

## Exercise 8: E-Commerce Checkout

**Concepts:** error boundary events, error end events, escalation vs error (introduce the error half here).

A customer checks out. The system reserves inventory, then charges the card. If the card is declined, the customer is asked to enter a different payment method and the charge is retried. If the payment gateway itself is unreachable (technical failure, not a decline), the order is put on hold and support is notified. If inventory reservation fails because the item went out of stock, the order is cancelled and the customer is told.

**Test scenarios**
1. Happy path: reserved, charged, confirmed.
2. Card declined once, second card works.
3. Payment gateway down. Order on hold, support notified.
4. Out of stock. Order cancelled.

**Constraints**
- Distinguish business outcomes (declined card) from technical errors (gateway down). Think about whether "declined" should be an error event or a gateway decision. Either can be defended; annotate your reasoning.
- If you release inventory on a failure path, show it.

---

## Exercise 9: Employee Onboarding

**Concepts:** embedded subprocess, call activity, boundary events on subprocesses, inclusive gateway.

When HR marks a new hire as signed, onboarding begins. IT provisioning is a self-contained chunk of work: create accounts, then provision equipment. Equipment depends on role: everyone gets a laptop; field staff also get a phone; on-site staff also get a building badge; engineers also get VPN access. Someone can be several of these. Provisioning must finish before the start date. If it isn't done 2 days before the start date, the IT manager is alerted (provisioning continues). Separately, payroll setup is a standard company process used by other workflows too. After both IT and payroll are done, the manager schedules a first-day orientation.

**Test scenarios**
1. Remote engineer: laptop + VPN.
2. On-site field engineer: laptop + phone + badge + VPN.
3. Office staff, provisioning runs late: alert fires, provisioning finishes, orientation scheduled.

**Constraints**
- IT provisioning is an embedded subprocess. Payroll setup is a call activity.
- Use an inclusive gateway for equipment. Its join must merge only the branches that actually fired.
- Decide: are IT provisioning and payroll setup parallel or sequential? Annotate.

---

## Exercise 10: Vendor Request for Quote

**Concepts:** multi-instance activities (parallel and sequential), completion conditions.

Procurement needs quotes for a large purchase. They send an RFQ to every vendor on the approved list (the list length varies). Each vendor's response is collected and scored. Procurement stops waiting once 3 quotes are in or 10 days pass, whichever first. The best quote then goes through an approval chain: department head, then finance, then (if over $100k) the CFO. Each approver must approve before the next sees it; any rejection ends the process.

**Test scenarios**
1. 5 vendors, all respond in 4 days. Proceeds after 3rd response. $40k, approved by two approvers.
2. 4 vendors, only 2 respond. Proceeds at day 10 with 2 quotes.
3. $250k quote, CFO rejects.

**Constraints**
- Use a parallel multi-instance for the vendor RFQs with a completion condition.
- Decide whether the approval chain is a sequential multi-instance over a list of approvers or three explicit tasks. Annotate the tradeoff you chose.
- Handle zero vendors responding.

---

## Exercise 11: Theme Park Ride Operations

**Concepts:** signal events, escalation events.

A roller coaster runs through dispatch cycles: load riders, check restraints, dispatch, unload. If a rider attendant sees a problem they can't handle (a rider who won't follow safety rules), they escalate to the ride supervisor, but the ride keeps operating while the supervisor deals with it. If park operations detects lightning within 10 miles, a park-wide weather closure is broadcast and every outdoor ride must stop, unload safely, and close. Rides reopen when an all-clear is broadcast.

**Test scenarios**
1. Normal dispatch cycles, no incidents.
2. Attendant escalates; supervisor handles; ride never stops.
3. Lightning detected mid-cycle. Ride finishes unloading, closes, waits for all-clear, reopens.

**Constraints**
- Model the park operations broadcaster as a separate process (or pool) that throws signals.
- Escalation must be non-interrupting.
- Explain in an annotation why a signal fits better than a message here.

---

## Exercise 12: Food Order Tracking

**Concepts:** event subprocesses (interrupting and non-interrupting).

A customer places a food order in an app. The restaurant accepts it, prepares it, and hands it to a courier, who delivers it. At any point before the courier picks up, the customer can cancel, which stops everything and triggers a refund. At any point during the whole order, the customer can ask "where's my order?" and gets a status update without affecting the order.

**Test scenarios**
1. Normal order, no inquiries.
2. Customer asks for status twice during prep. Order continues.
3. Customer cancels during prep. Order stops, refund issued.
4. Customer tries to cancel after pickup. Cancellation not allowed.

**Constraints**
- Use event subprocesses, not boundary events on every task.
- Scenario 4 is the hard part. Decide how you scope the cancel window and annotate it.

---

## Exercise 13: Trip Booking

**Concepts:** compensation (boundary compensation events, compensation handlers, compensation throw events). Check that your Camunda version supports compensation; older Camunda 8 versions did not.

A travel site books a trip: flight, then hotel, then rental car. Each booking is a separate call to a third party and each succeeds or fails independently. If any booking fails, everything already booked must be cancelled, in reverse order, and the customer is notified that the trip couldn't be booked. If all succeed, the customer gets a confirmation.

**Test scenarios**
1. All three succeed.
2. Flight and hotel succeed, car fails. Hotel cancelled, then flight cancelled.
3. Flight fails. Nothing to undo.

**Constraints**
- Use compensation, not hand-wired "cancel" branches.
- Compensation handlers attach via association, not sequence flow.

---

## Exercise 14: Capstone, Food Delivery Platform

**Concepts:** everything.

Model a delivery platform with four participants: Customer, Platform, Restaurant, Courier.

- Customer places an order. Platform charges the card (declines handled as in Exercise 8).
- Platform sends the order to the restaurant. If the restaurant doesn't accept in 5 minutes, the order goes to the next-closest restaurant offering the same item, up to 2 alternates; then the order fails and the customer is refunded.
- Once accepted, the platform offers the delivery to couriers. Offers go to nearby couriers in parallel; first to accept wins. If none accept in 10 minutes, widen the radius once and retry.
- Customer can cancel until the courier picks up (refund, compensation for the courier offer).
- Customer can request status any time.
- If a severe weather signal is raised for the region, new courier offers are paused until it clears.
- After delivery, the customer is asked to rate the order. If no rating in 48 hours, the request expires.

**Test scenarios:** write at least six of your own before modeling. Include the evaluator in reviewing them.

**Constraints**
- Restaurant and Courier may be expanded or black-box. Justify the choice.
- Keep the Platform process readable. Use subprocesses to keep any single level under ~20 flow nodes.

---

# Evaluator Prompt

Paste the block below into any AI chatbot that accepts images. Replace both placeholders and attach the exported diagram image.

````text
You are a BPMN 2.0 reviewer evaluating a learner's Camunda Modeler diagram. You have two inputs: an attached image of the diagram, and the diagram's BPMN XML with the drawing (BPMNDI) and Camunda extension sections removed. Your main job is to decide whether the model achieves the outcomes the exercise requires. Be accurate and direct. Do not praise work unless it is specifically correct, and do not soften real errors. If you are unsure about something, say so instead of guessing.

## The exercise

<exercise>
[PASTE THE FULL EXERCISE TEXT HERE: title, concepts, scenario, test scenarios, constraints]
</exercise>

## The model XML (drawing and extension sections removed)

<bpmn_xml>
[PASTE THE STRIPPED XML HERE]
</bpmn_xml>

## How to use the two inputs

- Use the image to understand the structure and layout as a human would read it.
- Use the XML as ground truth for element types, connections, conditions, timer definitions, interrupting vs non-interrupting (`cancelActivity`), multi-instance settings, and called processes.
- If the image and XML disagree, trust the XML and report the mismatch.
- The removed sections are expected. Do not report missing BPMNDI or missing Camunda extension elements.

## Grading philosophy

- Outcome over form. There is usually more than one valid model. If the learner's approach is valid BPMN and every test scenario reaches the correct outcome, it passes, even if you would have modeled it differently.
- Extra scope is fine. If the learner models things the exercise did not ask for, including concepts from later in a curriculum, do not penalize it. Only flag extras if they are themselves broken or break a required scenario.
- Required concepts are required. If the exercise's constraints name a concept (for example "use an event-based gateway"), the learner must use it correctly. A model that reaches the outcome without the required concept fails that constraint.
- Do not invent requirements that are not in the exercise. If the exercise is ambiguous in a way that affects your judgment, say so instead of penalizing the learner.

## How to evaluate

Work through these steps in order.

1. **Inventory.** From the XML, list the pools, lanes, and every flow node with its ID, label, and type (including event trigger, catch/throw, interrupting vs non-interrupting, and markers such as multi-instance). Note unnamed elements. If the XML is truncated or malformed, stop and say so. Cross-check against the image and note any mismatch.

2. **Trace each test scenario.** For each scenario in the exercise, walk a token through the model element by element, citing each element's label and ID. State whether it reaches the correct outcome. If it fails, name the exact element and explain why. This step decides the verdict.

3. **Structural checks.** Look for problems that break scenarios or would break untested ones:
   - Deadlocks (a parallel join waiting on branches from an exclusive split; an inclusive join without a matching inclusive split).
   - Unintended duplicate tokens (a parallel split merged by an exclusive gateway, then continuing).
   - Dead ends: elements with no outgoing flow that are not end events, or unreachable elements.
   - Sequence flows crossing pool boundaries (invalid) or message flows within one pool (invalid).
   - Message flows connected to elements that cannot send or receive messages.
   - Gateways with conditional outflows and no default or obviously exhaustive labels.

4. **Required concepts.** For each concept the exercise lists, state whether it appears and whether it is used correctly per BPMN 2.0. If a different construct would be clearly more correct (for example an exclusive gateway where only an event-based gateway can wait on competing events), flag it.

5. **Decisions and annotations.** Where the exercise says "decide" or "annotate," confirm the learner made a choice and wrote it in a text annotation. Judge whether the choice is reasonable. A reasonable choice you disagree with is not an error.

6. **Naming.** Briefly note naming that hurts readability: tasks should be verb + object, events object + state, gateways a question with answer-labeled outflows, lanes as roles. Keep this short; it never changes the verdict.

## Output format

- **Verdict:** `Pass`, `Pass with issues`, or `Not yet`. `Pass` means every scenario reaches the right outcome and every required concept is used correctly. One sentence of justification.
- **Scenario results:** one line per scenario: pass or fail, and the failure point.
- **Findings table:** columns `Severity` (Error / Warning / Style), `Element`, `Issue`, `Why it matters`, `Suggested fix`. Identify each element by label and XML ID. Errors are things that break a scenario or misuse a required concept. Warnings are things that work but would cause trouble in a real model. Style is naming and readability.
- **Image/XML mismatches:** any place the picture and the XML disagree. Write "None" if none.
- **Valid alternatives:** places where the learner modeled something differently than you would have, but validly. These are not errors.
- **Ready to move on?** Yes or no. If no, the single concept to fix first.

Do not redraw or rewrite the full model unless asked.
````

## Caveats on AI evaluation

Chatbots are good at scenario tracing on small and medium models and get less reliable as models grow. Exercises 12 through 14 are where you'll feel it. Two habits help:

- **Check the inventory first.** If the evaluator's element list is wrong, everything after it is suspect. Correct it in a follow-up message before reading the findings.
- **Split big models.** For the capstone, evaluate one pool or subprocess at a time (its image crop plus its slice of the XML), then do a final pass on just the message flows between pools.

If two chatbots disagree, the BPMN 2.0 spec and Camunda's BPMN docs are the tiebreakers.
