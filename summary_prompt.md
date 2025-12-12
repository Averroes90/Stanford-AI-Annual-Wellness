# Summary Prompt: Post-Call Data Extraction

Analyze this Medicare Annual Wellness Visit questionnaire call and extract structured data.

## Instructions

Review the transcript and extract the patient's responses to each question. Map responses to the closest valid option. If a response is ambiguous or unclear, mark as "unclear" and note the issue.

## Output Format

Return the following structured summary:

```
=== PATIENT RESPONSES ===

Q1 Overall Health: [excellent | very_good | good | fair | poor | not_asked | unclear]
   Raw response: "[what patient actually said]"

Q2 Depression (PHQ-2): [not_at_all | several_days | more_than_half | nearly_every_day | not_asked | unclear]
   Raw response: "[what patient actually said]"
   Safety follow-up asked: [yes | no]
   Safety concern identified: [yes | no | not_applicable]

Q3 Falls Past 12 Months: [yes | no | not_asked | unclear]
   Raw response: "[what patient actually said]"
   If yes - Fall count: [number | unclear | not_asked]
   If yes - Injury: [yes | no | unclear | not_asked]

Q4 Unsteady Standing/Walking: [yes | no | not_asked | unclear]
   Raw response: "[what patient actually said]"

Q5 Social Support Available: [as_much_as_needed | quite_a_bit | some | a_little | not_at_all | not_asked | unclear]
   Raw response: "[what patient actually said]"

Q6 Exercise Days Per Week: [0-7 | not_asked | unclear]
   Raw response: "[what patient actually said]"

Q7 Alcohol Days Per Week: [0-7 | not_asked | unclear]
   Raw response: "[what patient actually said]"

Q8 Advance Directive on File: [yes | no | unsure | not_asked | unclear]
   Raw response: "[what patient actually said]"


=== ESCALATION FLAGS ===

Depression Protocol Triggered: [yes | no]
Safety Concern Identified: [yes | no]
Falls with Injury: [yes | no]
Other Escalation: [describe or "none"]

Priority Follow-up Required: [yes | no]
If yes, reason: [explanation]


=== CALL QUALITY ===

Completion Status: [complete | partial | abandoned]
Questions Answered: [X] / 8
Questions Skipped: [list or "none"]
Clarifications Needed: [count]

Data Quality Issues:
- [List any responses that were unclear or required interpretation]
- [List any questions where patient seemed confused]


=== ADDITIONAL NOTES FOR CARE TEAM ===

[Any health concerns, symptoms, questions, or requests the patient mentioned that should be addressed at their visit]


=== CALL METADATA ===

Approximate Duration: [estimate based on transcript length]
Patient Engagement: [cooperative | hesitant | confused | frustrated | other]
Any Technical Issues: [describe or "none"]
```

## Mapping Guidelines

**For Likert scales:** Map colloquial responses to the closest option
- "pretty good" → good
- "not great" → fair  
- "terrible" → poor
- "I guess okay" → unclear (note: patient was vague)

**For Yes/No:** Accept variations
- "yeah", "yep", "I did" → yes
- "nope", "not really", "I don't think so" → no

**For numbers:** Extract the number
- "a couple" → 2
- "a few" → 3 (note: interpreted "few" as 3)
- "maybe 3 or 4" → 3-4 (note: patient was uncertain)

**When unclear:** Always note what the patient actually said so a human can review.
