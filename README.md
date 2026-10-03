# orange_project
Project Resolution: Prescription Clarification Prioritiser
1. Project Title

Prescription Clarification Prioritiser for Timely Pharmacy–Prescriber Communication

2. Problem Resolution

The proposed system is a decision-support application that helps pharmacy staff identify which prescription clarification requests should be handled first.

The system receives a clarification request containing:

Medicine risk level
Patient waiting time
Clarification type
Prescription completeness/quality
Urgency-related information
Previous clarification status

It then assigns a priority level:

Priority	Meaning	Action
🔴 High	Potentially time-critical clarification	Escalate to authorised pharmacist/prescriber
🟡 Medium	Important but not immediately time-critical	Normal escalation queue
🟢 Low	Lower urgency	Process through normal workflow

Important: The system does not diagnose the patient, recommend medicines, change doses, or make treatment decisions. It only prioritises communication between authorised pharmacy and prescriber staff.

3. Proposed System Architecture
Synthetic Clarification Request
             ↓
      Input Validation
             ↓
   Data Quality / Missing Check
             ↓
   Risk + Waiting-Time Features
             ↓
     Priority Model / Rules
             ↓
   ┌─────────┼─────────┐
   ↓         ↓         ↓
 HIGH      MEDIUM      LOW
   ↓         ↓         ↓
Evidence   Evidence   Evidence
   ↓         ↓         ↓
Authorised Pharmacy Staff
             ↓
      Prescriber Contact
             ↓
 Clarification Resolved
             ↓
      Outcome Recorded
4. Dataset

Create a synthetic dataset because real patient data should not be required for this student project.

Example:

Request ID	Medicine Risk	Waiting Time	Clarification Type	Input Quality	Resolution Time	Priority Outcome
CR001	High	45 min	Missing information	Good	12 min	High
CR002	Medium	20 min	Prescription mismatch	Good	35 min	Medium
CR003	Low	10 min	Missing instruction	Good	60 min	Low
CR004	High	70 min	Unclear prescription	Poor	18 min	High
CR005	Medium	90 min	Missing information	Good	25 min	High

Generate around 1,000–5,000 synthetic clarification requests so the experiment has enough examples.

Important dataset fields
request_id
medicine_risk
patient_waiting_time
clarification_type
prescription_quality
missing_information
request_age
previous_escalation
resolution_time
resolved_before_threshold
priority_label

The target outcome should focus on:

Was a high-risk clarification resolved earlier?

5. Labels and Thresholds

Use three priority labels:

High Priority

A clarification can be labelled High when it satisfies conditions such as:

Medicine risk = High
AND
patient waiting time >= 30 minutes

or

Medicine risk = High
AND
prescription/input quality is poor

or

High-risk medicine
AND
request has remained unresolved beyond the defined threshold
Medium Priority

Examples:

Medium medicine risk
AND
waiting time >= 30 minutes
Low Priority

Lower-risk requests with short waiting times and no major missing information.

These thresholds are project-defined operational thresholds, not clinical guidelines.

6. Baseline

Your baseline should be something simple and measurable.

Baseline: FIFO

First-In-First-Out queue

The oldest clarification is handled first regardless of medicine risk.

Example:

Request A → 60 min waiting → Low risk
Request B → 20 min waiting → High risk
Request C → 30 min waiting → Medium risk

FIFO:

A → B → C

Your prioritiser:

B → A → C

This allows you to test whether the proposed system gets high-risk clarifications resolved earlier than the existing/basic queue.

7. Proposed Prioritisation Model

For a student project, I recommend starting with a transparent scoring model rather than a black-box model.

Example:

Priority Score =

Medicine Risk Score
+
Waiting Time Score
+
Input Urgency Score
+
Unresolved Duration Score

Example:

Feature	Score
High medicine risk	+50
Medium medicine risk	+25
Low medicine risk	+10
Waiting >60 min	+30
Waiting 30–60 min	+20
Waiting <30 min	+5
Missing critical information	+15
Previous escalation	+10

Then:

Score >= 70  → HIGH
40–69        → MEDIUM
<40          → LOW

The exact thresholds should be tuned using the synthetic validation dataset and then frozen for the final test.

8. Evidence for Every High-Priority Output

This is an important requirement.

Don't simply display:

HIGH PRIORITY

Instead show why.

Example application output:

REQUEST: CR004

Priority: 🔴 HIGH

Reasons:
✓ Medicine risk: HIGH
✓ Patient waiting time: 68 minutes
✓ Clarification information incomplete

Evidence:
Medicine risk contributed +50
Waiting time contributed +30
Incomplete information contributed +15

Priority Score: 95

Recommended workflow:
Escalate to authorised pharmacy/prescriber staff.

Note:
This system does not provide a diagnosis,
dose recommendation, or treatment decision.

This makes the system explainable and auditable.

9. Low-Quality / Missing Input Handling

You specifically need to demonstrate this.

Example 1 — Missing medicine risk
Medicine risk: UNKNOWN
Waiting time: 55 minutes
Clarification: Missing information

System:

Priority: UNCERTAIN

Reason:
Medicine risk information is missing.

Action:
Manual review required by authorised staff.

Do not automatically assume it is low risk.

Example 2 — Missing waiting time
Medicine risk: HIGH
Waiting time: UNKNOWN

Output:

Priority: HIGH

Confidence: LIMITED

Reason:
High-risk classification is available,
but patient waiting time is missing.

Action:
Verify waiting time before final queue ordering.
Example 3 — Poor-quality clarification
Clarification:
"Rx unclear, pls check"

Output:

Priority: UNCERTAIN

Data quality warning:
Clarification description is insufficient.

Action:
Authorised staff review required.
10. Three Required Failure / Edge Cases
Edge Case 1 — Missing medicine risk

Expected behaviour:

No automatic low-risk assumption
↓
Flag uncertainty
↓
Manual review
Edge Case 2 — Extremely long waiting time but low medicine risk

Example:

Low risk
Waiting = 180 minutes

The system should not blindly classify this as high-risk merely because of waiting time.

Instead:

Priority = Medium
Reason = prolonged waiting time

depending on your frozen threshold.

Edge Case 3 — High-risk medicine but missing other inputs
Risk = High
Waiting = Unknown
Prescription quality = Unknown

The system should still provide the available evidence but clearly state:

Incomplete information
↓
Limited confidence
↓
Authorised staff review
11. False Positive Analysis

A false positive occurs when the system marks something High Priority but the actual outcome shows it did not need early resolution according to your predefined evaluation label.

Example:

Predicted: HIGH
Actual: LOW

Inspect:

Why was it prioritised?
Was waiting time unusually high?
Was medicine risk incorrectly recorded?
Was input information incomplete?
Did the threshold produce excessive escalation?

Create a table:

Request	Prediction	Actual	Reason
CR102	High	Low	Waiting time dominated score
CR241	High	Medium	Risk information borderline
12. False Negative Analysis

This is even more important.

A false negative is:

Predicted: LOW/MEDIUM
Actual: HIGH

Example:

Medicine risk = HIGH
Waiting time = 15 min
Prediction = LOW
Actual = HIGH

This tells you your threshold may be missing high-risk requests.

For every false negative, investigate:

Input
↓
Feature values
↓
Score
↓
Threshold
↓
Why it was missed
↓
Potential threshold adjustment
13. Experiment

Run the same synthetic requests through:

Experiment A — FIFO baseline
First request → first processed
Experiment B — Proposed prioritiser
Priority score
↓
High
↓
Medium
↓
Low

Measure:

Primary metric

High-risk clarification early-resolution rate

High-risk requests resolved early
---------------------------------- × 100
Total high-risk requests

Also measure:

Precision
Recall
F1 score
False positives
False negatives
Average resolution time
Median resolution time
High-risk resolution time
Percentage of high-risk requests resolved within target time
14. Example Experimental Result

Your final result could look like this after actually running the experiment:

Metric	FIFO Baseline	Prioritiser
High-risk early resolution	61%	84%
Average high-risk resolution time	48 min	29 min
Precision	—	0.82
Recall	—	0.88
F1	—	0.85

Don't present these numbers as real results until your implementation produces them. They are example target/result formats.

Your main project conclusion should be based on the actual measured result.

15. Functional Application

Make a simple web application using something like:

Python + Streamlit + Pandas + Scikit-learn

Application pages:

Dashboard
-------------------------------------
Prescription Clarification Dashboard
-------------------------------------

🔴 High Priority       12
🟡 Medium Priority     21
🟢 Low Priority        34
⚠ Uncertain             4
-------------------------------------
New Clarification
Medicine Risk:
[ High ▼ ]

Waiting Time:
[ 45 ]

Clarification Type:
[ Missing Information ▼ ]

Input Quality:
[ Good ▼ ]

[ PRIORITISE ]
Result
🔴 HIGH PRIORITY

Score: 85

Why?
✓ High medicine risk
✓ Waiting > 30 minutes
✓ Missing information

Evidence available: YES

Action:
Escalate to authorised pharmacy/prescriber staff.
16. Uncertainty Panel

Add a dedicated section:

⚠ DATA QUALITY WARNING

Medicine risk: Missing
Waiting time: Available
Clarification text: Available

Confidence: LIMITED

The system cannot reliably determine priority
from the available information.

Required action:
Manual review by authorised staff.

This directly satisfies your uncertainty communication requirement.

17. Field Workflow Map

Your documentation should contain:

Prescription received
        ↓
Pharmacist identifies clarification
        ↓
Clarification request created
        ↓
Current process
        ↓
Request waits in queue
        ↓
Prescriber contacted
        ↓
Clarification resolved
        ↓
Medicine dispensed
Proposed workflow
Prescription received
        ↓
Clarification identified
        ↓
Request entered into system
        ↓
Input/data-quality check
        ↓
Priority scoring
        ↓
┌───────────────┐
│ High          │ → Immediate authorised escalation
├───────────────┤
│ Medium        │ → Standard escalation
├───────────────┤
│ Low           │ → Normal queue
├───────────────┤
│ Uncertain     │ → Manual review
└───────────────┘
        ↓
Prescriber communication
        ↓
Clarification resolved
        ↓
Outcome recorded
18. Stakeholder Validation

You don't need a huge clinical study.

Conduct a small user/stakeholder validation with authorised/knowledgeable participants such as pharmacy staff, faculty, or project reviewers.

Ask them to rate:

Question	1–5
Is the priority understandable?	
Are the reasons useful?	
Is the evidence clear?	
Is uncertainty communicated properly?	
Would this reduce manual queue sorting?	
Is the interface easy to understand?	

Also ask:

"What would make you hesitate to trust this priority?"

Summarise the responses rather than claiming clinical validation.

19. Project Deliverables

Your final submission can contain exactly these:

1. Field-workflow map

Current workflow + proposed workflow.

2. Data-generation script

Python script generating synthetic clarification requests.

3. Functional application

Streamlit web application.

4. Experiment notebook

Baseline vs prioritiser experiment.

5. Failure-mode analysis

At least 3 edge cases + FP/FN analysis.

6. User feedback summary

Short stakeholder/user evaluation.

7. Technical documentation

Architecture, dataset, labels, thresholds, limitations and deployment instructions.

8. Presentation

Problem → workflow → dataset → model → application → experiment → failures → validation → conclusion.
