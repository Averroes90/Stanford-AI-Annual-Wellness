# Medicare Annual Wellness Visit Voice Agent

## Hippocratic AI - TPM Take-Home Project

**Candidate:** Rami Ibrahimi  
**Date:** December 2024  
**Status:** 🟡 In Progress

---

## Executive Summary

A voice AI agent that conducts Medicare Annual Wellness Visit (AWV) health risk assessments via phone, built on the Stanford Medicine Partners questionnaire. The agent captures structured patient responses, handles sensitive topics with appropriate escalation protocols, and demonstrates production-readiness considerations for healthcare deployment.

---

## Quick Demo Guide

> **For interview presentation - start here**

1. **Live Call** (3 min): [Call the agent, demonstrate happy path + one escalation]
2. **Show Output**: [Structured summary generated post-call]
3. **Walkthrough**: [Key design decisions below]
4. **Eval Discussion**: [Part 2 of this document]

---

# Part 1: The Agent

## What's Working Now

### Core Conversation Flow

- [x] Greeting and identity confirmation
- [x] 8 representative questions from Stanford AWV form
- [x] Brief acknowledgments between questions
- [x] Closing with open-ended "anything else" prompt
- [ ] Structured data extraction at end of call

### Questions Implemented

| # | Question | Type | Escalation Trigger |
|---|----------|------|-------------------|
| 1 | Overall health rating | 5-point Likert | No |
| 2 | Feeling depressed/hopeless (PHQ-2) | 4-point Likert | **Yes** |
| 3 | Falls in past 12 months | Yes/No + follow-up | **Yes (if injured)** |
| 4 | Unsteady when standing/walking | Yes/No | No |
| 5 | Social support availability | 5-point Likert | No |
| 6 | Exercise days per week | Numeric (0-7) | No |
| 7 | Alcohol days per week | Numeric (0-7) | No |
| 8 | Advance directive on file | Yes/No/Unsure | No |

### Escalation Protocols

- [x] Depression screening (PHQ-2 positive → safety follow-up question)
- [x] Falls with injury → flagged for care team
- [ ] Home safety concerns → flagged for care team

### Conversation Handling

- [x] Vague answers → one clarifying question, then accept
- [x] "I don't know" → acknowledge and move on
- [x] Off-topic responses → acknowledge, note for provider, redirect
- [ ] Interruption handling → confirm answer matches question
- [ ] Follow-up limits → max 1 per topic, no symptom exploration

---

## Technical Implementation

### Platform

- **Voice AI:** Vapi
- **LLM:** Claude Sonnet / GPT-4o (configurable)
- **Speech-to-Text:** Vapi default (Deepgram)
- **Text-to-Speech:** Vapi default

### Configuration

| Setting | Value | Rationale |
|---------|-------|-----------|
| Temperature | 0.4 | Consistency over creativity |
| Max Tokens | 200 | Responses are short |
| End of Turn Timeout | 1.5s | Balance: don't cut off, don't lag |

### System Prompt
>
> Located in: `prompts/system_prompt.md`

Key sections:

- Opening script
- Question sequence
- Escalation protocols
- Scope boundaries
- Interruption handling
- State tracking guidance

---

## Production Architecture Considerations

### Current Prototype Approach

The system prompt (~2000 tokens) is sent with every conversational turn, along with growing conversation history. By question 8, each turn sends ~3500+ tokens. For a prototype, this is acceptable.

### Why This Doesn't Scale

| Metric | 8-Question Call | At 1M Calls/Month |
|--------|-----------------|-------------------|
| Tokens per call | ~25,000 | 25 billion |
| Estimated cost (GPT-4o) | ~$0.15 | ~$150,000 |
| Estimated cost (Haiku) | ~$0.02 | ~$20,000 |

Latency also increases as context grows - directly impacting the emotional connection Hippocratic emphasizes.

### Production Alternative: State Machine + Minimal Prompt

```
┌─────────────────────────────────────────────────┐
│  Backend State Manager                          │
│  - Tracks current question                      │
│  - Manages branching logic (falls → count)      │
│  - Validates responses                          │
│  - Triggers escalations                         │
└─────────────────────────────────────────────────┘
                      │
          Injects: {{currentQuestion}}
          Injects: {{validResponses}}
                      │
                      ▼
┌─────────────────────────────────────────────────┐
│  Minimal LLM Prompt (~500 tokens)               │
│  - Role and tone                                │
│  - Current question only                        │
│  - Escalation rules                             │
│  - Parse response + acknowledge + ask next      │
└─────────────────────────────────────────────────┘
```

**Benefits:**

- 60-70% reduction in tokens per call
- Consistent latency (context doesn't grow)
- Deterministic state management
- Easier to test and debug branching logic

### Minimal Prompt Template (Example)

```
You are a healthcare assistant for Stanford Medicine Partners.
Tone: Warm, patient, professional.

CURRENT QUESTION: {{currentQuestion}}
VALID RESPONSES: {{validResponses}}
FOLLOW-UP IF NEEDED: {{followUpRule}}

RULES:
- Acknowledge briefly, then ask the question naturally
- One clarifying question max, then accept the response
- Never give medical advice
- Safety concerns → "I'll make sure your care team follows up"

Parse the patient's response and proceed.
```

### Implementation Options

| Approach | Complexity | Token Savings | Notes |
|----------|------------|---------------|-------|
| Vapi dynamic variables | Low | 40-50% | Use `{{variables}}` in prompt |
| Vapi function calling | Medium | 60-70% | Backend returns next question |
| Custom voice pipeline | High | 70-80% | Full control, more engineering |

For this project, I implemented the full-prompt approach for simplicity. The above architecture would be my recommendation for production deployment.

## Planned Enhancements

### High Priority (Before Submission)

| Enhancement | Status | Effort | Impact |
|-------------|--------|--------|--------|
| Add interruption handling to prompt | 🔲 Todo | 15 min | Handles edge case from testing |
| Add follow-up limits to prompt | 🔲 Todo | 10 min | Prevents infinite loops |
| Add scope boundaries to prompt | 🔲 Todo | 10 min | Aligns with Hippocratic's non-diagnostic model |
| Configure Summary Prompt for structured output | 🔲 Todo | 15 min | Concrete artifact to show |
| Configure Success Evaluation | 🔲 Todo | 10 min | Auto-generates eval metrics |
| Test depression escalation path | 🔲 Todo | 15 min | Critical safety feature |
| Test falls with injury path | 🔲 Todo | 10 min | Secondary escalation |

### Medium Priority (Time Permitting)

| Enhancement | Status | Effort | Impact |
|-------------|--------|--------|--------|
| Simple API wrapper script | 🔲 Todo | 30 min | Shows programmatic understanding |
| Add state tracking to prompt | 🔲 Todo | 20 min | More deterministic flow |
| Tune turn-taking settings | 🔲 Todo | 15 min | Better interruption handling |
| Test 3+ challenging scenarios | 🔲 Todo | 45 min | Strengthens eval discussion |
| Document test results with metrics | 🔲 Todo | 30 min | Concrete data for Part 2 |

### Future / Production Considerations

| Enhancement | Notes |
|-------------|-------|
| Hybrid NER/classifier architecture | 80% cost reduction, 5-10x latency improvement |
| Fine-tuned small model for response extraction | Requires training data |
| Webhook integration for real-time escalation | Alert care team immediately, not post-call |
| EHR integration | Push structured data to patient record |
| Multi-language support | Spanish priority (Hippocratic highlights this) |
| Deterministic state machine | LLM handles conversation, backend tracks state |
| Function calling for response capture | Real-time structured data, not post-call parsing |

---

# Part 2: Evaluation Framework

## Challenging Conversational Scenarios

### Tested

| Scenario | Description | Result | Notes |
|----------|-------------|--------|-------|
| Happy path | Straightforward answers to all questions | 🔲 | |
| Depression escalation | "Nearly every day" to PHQ-2 | 🔲 | |
| Falls with injury | Multiple falls, confirmed injury | 🔲 | |
| Vague answers | "I guess... maybe fair?" | 🔲 | |

### Planned to Test

| Scenario | Description | Why It Matters |
|----------|-------------|----------------|
| Patient confusion | Misunderstands question, gives irrelevant answer | Common with elderly population |
| Interruption | Answers before question complete | Voice UI challenge |
| Emotional distress | Patient becomes upset during call | Safety + empathy |
| Off-topic tangent | Patient tells long unrelated story | Time management |
| Hearing difficulty | Patient asks to repeat multiple times | Accessibility |
| Non-native English | Accented speech, different phrasing | Diverse patient population |
| Decline to answer | "I'd rather not say" on sensitive question | Respect autonomy |

---

## Test Plan for Production Readiness

### Test Volume Recommendation

| Phase | Test Type | Volume | Purpose |
|-------|-----------|--------|---------|
| **Alpha** | Internal team testing | 50 calls | Find obvious bugs, prompt issues |
| **Beta** | Simulated patients (diverse personas) | 200 calls | Edge cases, accent handling, escalation accuracy |
| **Pilot** | Real patients (supervised) | 500 calls | Real-world validation with human backup |
| **Production** | Ongoing monitoring | Continuous | Drift detection, new edge cases |

### Test Variety Matrix

| Dimension | Variations to Test |
|-----------|-------------------|
| **Patient personas** | Elderly, cognitively impaired, anxious, impatient, verbose, non-native speaker |
| **Response styles** | Terse, verbose, vague, precise, off-topic |
| **Escalation triggers** | Depression positive, falls with injury, safety concerns, none |
| **Technical conditions** | Good audio, background noise, poor connection |
| **Completion paths** | Full completion, partial (patient ends early), abandoned |

### Signoff and Rollout Process

```
┌─────────────────────────────────────────────────────────────────┐
│  1. DEVELOPMENT                                                 │
│     - Prompt engineering complete                               │
│     - Internal testing passes                                   │
│     - Escalation protocols verified                             │
│                        ↓                                        │
│  2. CLINICAL REVIEW                                             │
│     - Physician reviews conversation design                     │
│     - Safety protocols approved                                 │
│     - Escalation language approved                              │
│                        ↓                                        │
│  3. SIMULATION TESTING                                          │
│     - 200+ calls with simulated patients                        │
│     - All persona types covered                                 │
│     - Accuracy thresholds met (see below)                       │
│                        ↓                                        │
│  4. PILOT                                                       │
│     - Limited real patient deployment                           │
│     - Human review of 100% of calls                             │
│     - Escalations trigger immediate human follow-up             │
│                        ↓                                        │
│  5. PRODUCTION                                                  │
│     - Gradual rollout (10% → 50% → 100%)                        │
│     - Automated monitoring for drift                            │
│     - Sampling-based human QA (5-10% of calls)                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Accuracy Thresholds

### Recommended Thresholds

| Metric | Target | Rationale |
|--------|--------|-----------|
| **Form field accuracy** | ≥95% | Data must be reliable for clinical use |
| **Escalation recall** | ≥99% | Missing a safety signal is unacceptable |
| **Escalation precision** | ≥80% | Some false positives OK (human reviews anyway) |
| **Completion rate** | ≥85% | Most calls should finish successfully |
| **Patient satisfaction** | ≥8/10 | Must not harm patient experience |

### Why These Numbers

**Form field accuracy (95%):**  

- The receiving clinician needs to trust the data
- 5% error rate = ~1 wrong answer per 20-question form
- Higher threshold ideal but may require human review for edge cases

**Escalation recall (99%):**  

- Patient safety is non-negotiable
- Missing a depression or safety flag could lead to harm
- This is Hippocratic's core value proposition: "safety-focused"

**Escalation precision (80%):**  

- False positives = unnecessary human review (cost, not harm)
- Acceptable tradeoff: better to over-flag than under-flag
- 20% false positive means 1 in 5 escalations are unnecessary

### Current Performance

| Metric | Measured | Target | Status |
|--------|----------|--------|--------|
| Form field accuracy | TBD | ≥95% | 🔲 |
| Escalation recall | TBD | ≥99% | 🔲 |
| Escalation precision | TBD | ≥80% | 🔲 |
| Completion rate | TBD | ≥85% | 🔲 |

---

## Evaluation Metrics

### Per-Call Metrics

| Metric | Description | How to Measure |
|--------|-------------|----------------|
| Completion rate | Questions answered / total | Post-call analysis |
| Call duration | Total time | Vapi metadata |
| Turns per question | Clarifications needed | Transcript analysis |
| Parse success | Responses mappable to fields | Structured output validation |
| Escalation triggered | Yes/No + type | Post-call analysis |

### Aggregate Metrics (Over Test Set)

| Metric | Description | Target |
|--------|-------------|--------|
| Mean completion rate | Avg % of questions answered | >90% |
| Mean call duration | Avg time per call | 4-7 min |
| Escalation recall | True positives / all positives | >99% |
| Escalation precision | True positives / predicted positives | >80% |
| Abandonment rate | Calls ended early by patient | <10% |

---

# Appendix

## A. Design Decisions

| Decision | Choice | Alternatives Considered | Rationale |
|----------|--------|------------------------|-----------|
| Number of questions | 8 | Full 24, minimal 5 | Representative coverage without excessive demo length |
| Escalation approach | Flag + continue | Stop call, transfer immediately | Non-diagnostic scope; human follows up post-call |
| Response extraction | Post-call LLM summary | Per-turn function calling, NER | Simplicity for prototype; noted production alternative |
| State management | Prompt-based | External state machine | Sufficient for demo; noted production alternative |

## B. Stanford AWV Form Mapping

| Form Question | Agent Question | Included |
|---------------|----------------|----------|
| Q1: Overall health | ✓ Implemented | Yes |
| Q2: Quality of life | Skipped | No |
| Q3: Mental health | Covered by PHQ-2 | Partial |
| Q4: Pain interference | Skipped | No |
| Q5: PHQ-2 (depression) | ✓ Implemented | Yes |
| Q6: ADL difficulties | Skipped | No |
| Q7: Incontinence | Skipped | No |
| Q8: Falls | ✓ Implemented | Yes |
| Q9: Walking status | Covered by unsteady Q | Partial |
| ... | ... | ... |

## C. Files

| File | Description |
|------|-------------|
| `PROJECT.md` | This document |
| `prompts/system_prompt.md` | Main agent system prompt |
| `prompts/summary_prompt.md` | Post-call summary extraction |
| `prompts/eval_prompt.md` | Success evaluation criteria |
| `scripts/api_demo.py` | Simple Vapi API wrapper (if built) |
| `test_results/` | Test call transcripts and analysis |

## D. References

- [Stanford Medicine Partners AWV Form](uploaded)
- [Hippocratic AI Website](https://hippocraticai.com/)
- [Vapi Documentation](https://docs.vapi.ai/)
- [Medicare AWV Requirements](https://www.medicare.gov/coverage/yearly-wellness-visits)

---

## Change Log

| Date | Change | Status |
|------|--------|--------|
| Dec 12, 2024 | Initial skeleton created | Current |
| | Added interruption handling to prompt | 🔲 |
| | Added follow-up limits to prompt | 🔲 |
| | Configured Summary Prompt | 🔲 |
| | Tested depression escalation | 🔲 |
| | ... | |

---

*Last updated: December 12, 2024*
