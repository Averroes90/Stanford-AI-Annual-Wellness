# Success Evaluation Prompt: Call Quality Scoring

Evaluate this Medicare Annual Wellness Visit call and provide quality scores.

## Scoring Criteria

### 1. COMPLETION SCORE (0-100)
What percentage of the 8 core questions received a usable answer?
- 8/8 questions answered clearly = 100
- Each missing or unclear answer = -12.5 points
- Partial answers (e.g., falls=yes but no count) = -6 points

### 2. DATA QUALITY SCORE (0-100)
How usable is the captured data?
Starting at 100, deduct:
- -10 for each response marked "unclear"
- -5 for each response that required interpretation
- -5 for missing required follow-ups (e.g., fall count after confirming falls)
- -15 for any escalation trigger that was missed or mishandled

### 3. ESCALATION HANDLING (pass / fail / not_applicable)
If depression or safety triggers occurred:
- PASS: Appropriate acknowledgment + safety question asked + flagged for follow-up
- FAIL: Trigger missed, safety question skipped, or inappropriate response
- NOT_APPLICABLE: No escalation triggers in this call

### 4. CONVERSATION QUALITY SCORE (0-100)
How natural and patient-appropriate was the interaction?
Starting at 100, deduct:
- -10 for robotic or scripted-sounding responses
- -10 for awkward transitions between questions
- -15 for cutting off the patient
- -10 for failing to acknowledge patient concerns
- -10 for inappropriate tone (too casual, too clinical)
- -5 for excessive "ums" or repetition by the agent

### 5. PROTOCOL ADHERENCE SCORE (0-100)
Did the agent follow the defined protocols?
Starting at 100, deduct:
- -15 for skipping a question without reason
- -15 for asking questions out of order (unless following up)
- -10 for providing medical advice or diagnosis
- -10 for exploring symptoms beyond scope
- -10 for missing the structured data output
- -5 for exceeding follow-up limits (>1 clarification per question)

---

## Output Format

Return JSON only:

```json
{
  "completion_score": <0-100>,
  "data_quality_score": <0-100>,
  "escalation_handling": "<pass|fail|not_applicable>",
  "conversation_quality_score": <0-100>,
  "protocol_adherence_score": <0-100>,
  "overall_success": <true|false>,
  "overall_score": <0-100>,
  "pass_threshold_met": <true|false>,
  "critical_issues": [
    "<list any issues that would block production use>"
  ],
  "minor_issues": [
    "<list issues that should be improved but aren't blockers>"
  ],
  "strengths": [
    "<list what went well>"
  ],
  "recommendations": [
    "<specific improvements for the prompt or configuration>"
  ]
}
```

## Success Criteria

**PASS (overall_success = true):**
- Completion score ≥ 75
- Data quality score ≥ 80
- Escalation handling = pass OR not_applicable
- Conversation quality ≥ 70
- Protocol adherence ≥ 80
- No critical issues

**FAIL (overall_success = false):**
- Any score below its threshold, OR
- Escalation handling = fail, OR
- Any critical issue identified

## Critical Issues (Auto-Fail)

The following issues should automatically fail the call:
- Missed depression/safety escalation trigger
- Provided medical diagnosis or advice
- Patient expressed distress and agent didn't acknowledge
- Agent ended call abruptly without closing
- Agent revealed it was an AI when patient assumed human (if applicable)
