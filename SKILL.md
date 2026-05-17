---
name: cn-recruiting-workflow
description: Micro-workflows for mainland China recruiting HR: score resumes against JD requirements, normalize interview feedback, draft offer approval packs, generate candidate messages, and produce tracker-ready record updates.
version: 0.1.0
metadata:
  openclaw:
    homepage: https://github.com/yongthelaoma-cyber/hrskill
    envVars:
      - name: ATS_EXPORT_PATH
        required: false
        description: Optional local export path for tracker rows or CSV write-back.
---

# CN Recruiting Workflow Skill

Use this skill when the user is doing recruiting operations for mainland China and needs actionable workflow output instead of a generic HR explainer.

This skill is optimized for 5 recruiting actions:

1. `score_candidate`
2. `summarize_interview_feedback`
3. `create_offer_approval_pack`
4. `generate_candidate_message`
5. `update_candidate_tracker`

## Outcome standard

When handling any recruiting workflow, always produce these sections:

```text
normalized_data
decision_summary
missing_information
next_action
message_draft
record_update
compliance_warning_if_any
```

Rules:

1. `normalized_data` must be structured and easy to map into ATS, Feishu Bitable, DingTalk approval forms, Notion, Google Sheets, or CSV.
2. `decision_summary` must make a decision or recommendation, not just restate the inputs.
3. `missing_information` must name the exact fields that block a confident HR action.
4. `next_action` must be something an HR operator can actually do today.
5. `message_draft` should be directly reusable in WeCom, Feishu, email, or a candidate chat.
6. `record_update` should be concise enough to write back into one row or one timeline entry.
7. `compliance_warning_if_any` should only appear when there is a concrete legal or privacy concern.

## Workflow routing

### 1. `score_candidate`

Use when the input includes a JD and a resume or candidate profile.

Expected input shapes:

1. JD in Markdown, DOCX export, PDF text, or pasted text
2. Resume in PDF text, DOCX export, pasted text, or profile notes
3. Optional hiring preference such as `conservative`, `aggressive`, `fast-hiring`, or `high-bar`

Always extract at least:

```text
candidate_name
target_role
match_score
must_have_match
preferred_match
risk_flags
interview_focus
next_action
message_to_candidate
record_summary
```

### 2. `summarize_interview_feedback`

Use when the input includes multiple interviewer comments, messy notes, chat transcripts, or form snippets.

Normalize feedback into:

```text
interviewer_name
interview_round
capability_feedback
experience_feedback
motivation_feedback
communication_feedback
culture_fit_feedback
salary_risk
stability_risk
hire_recommendation
follow_up_questions
confidence_level
```

Then produce:

```text
candidate_summary
interviewer_opinions
key_strengths
key_concerns
conflict_between_feedback
recommended_decision
follow_up_questions
candidate_reply_draft
record_summary
```

### 3. `create_offer_approval_pack`

Use when HR has enough candidate context to prepare an internal approval pack before sending an offer.

Check for missing or risky fields such as:

1. target role or grade
2. department or reporting line
3. employment entity
4. work location
5. base salary or total cash
6. bonus, allowance, or stock notes
7. budget range
8. expected onboard date
9. trial period

Always produce:

```text
offer_approval_summary
candidate_value_summary
salary_budget_check
missing_fields
approval_recommendation
risk_notes
offer_message_draft
record_summary
```

### 4. `generate_candidate_message`

Use when the user already knows the intended next step and needs a candidate-facing message.

Default tone:

1. clear
2. respectful
3. concise
4. realistic about timing

Supported intents:

1. invite to interview
2. request missing materials
3. advance to next round
4. hold in pipeline
5. reject politely
6. start offer discussion

### 5. `update_candidate_tracker`

Use when the user wants a write-back summary for ATS or a spreadsheet.

Default tracker row format:

```text
candidate_name
target_role
stage
decision
risk_flags
owner
next_action
last_update_summary
```

If the user provides an existing tracker schema, follow that schema instead.

## Working style

1. Prefer small executable HR actions over large SOP explanations.
2. Assume messy, incomplete input from PDFs, DOCX exports, OCR, chat logs, and spreadsheet cells.
3. Prioritize hiring decisions, risks, and next steps over pretty prose.
4. Flag privacy or labor-law concerns only when the issue is concrete and material.
5. If confidence is low because inputs are incomplete or contradictory, say so explicitly and narrow the recommendation.

## Mainland China context

Keep outputs aligned with common mainland China recruiting practice:

1. JDs often include hard requirements, preferred requirements, reporting line, city, and salary range.
2. Resumes often omit salary expectation, notice period, or true interview motivation.
3. Interview feedback is frequently fragmented across chat, email, forms, or voice-to-text notes.
4. Internal approval flows usually require concise business value, budget fit, and risk notes before an offer can move forward.

For common field shapes and examples, see [references/recruiting-fields.md](references/recruiting-fields.md).

