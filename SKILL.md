---
name: cn-recruiting-workflow
description: 帮招聘 HR 整理面试反馈、初筛简历、生成候选人沟通话术和跟进记录，把今天要推进的招聘动作往前推一步。 / Help recruiting teams debrief interviews, screen resumes, draft candidate follow-ups, and update trackers.
version: 0.4.1
metadata:
  openclaw:
    homepage: https://github.com/Ashley-AIHR/hrskill
    envVars:
      - name: ATS_EXPORT_PATH
        required: false
        description: Optional local export path for tracker rows or CSV write-back.
---

# 招聘推进助手 / Recruiting Follow-up Copilot

当用户在处理招聘推进、面试反馈汇总、候选人沟通或 Offer 前置材料时使用这个 skill。它不是一个解释招聘概念的机器人，而是一个帮 HR 把下一步动作理出来、写出来、落到记录里的小助手。 / Use this skill when the user needs help moving recruiting work forward: interview debriefs, candidate follow-ups, and offer prep.

如果用户不知道怎么开始，优先引导她先选一个场景，而不是让她从零写 prompt。 / If the user is unsure how to start, route them into a concrete scenario instead of asking for a free-form prompt.

## 你可以先做这 3 件事 / Start Here

第一次使用时，优先把用户带到下面 3 个入口之一：

1. `整理面试反馈`
2. `初筛简历`
3. `生成候选人沟通稿`

默认向导语：

1. `把面试反馈、候选人简历和岗位 JD 发给我，我先帮你整理结论、分歧点、推进建议和候选人沟通稿。`
2. `把岗位 JD 和候选人简历发给我，我先帮你做初筛判断、列出风险点和建议追问。`
3. `告诉我你想推进、保留还是婉拒候选人，我直接帮你写沟通话术。`

## 你要准备什么 / What To Prepare

不要求完美 JSON。默认接受脏输入、零散输入和复制粘贴的文本。 / Do not require perfect JSON. Accept messy pasted inputs by default.

最常见输入包括：

1. 面试官在飞书、企微、邮件里的零散反馈
2. 候选人简历文本、PDF 导出、Word 导出
3. 岗位 JD 文本
4. 用人经理的一句话判断
5. 现有 ATS 或表格中的跟进备注

如果用户材料不完整，也先继续做，但要明确标出缺口。 / Continue even with incomplete inputs, but surface missing information explicitly.

## 你会拿到什么 / What The User Gets

每次运行后，优先让用户看到这 5 块结果：

1. `推进结论`
2. `关键风险`
3. `下一步动作`
4. `可直接复制的沟通话术`
5. `可回写的跟进记录`

如果用户需要文件，还要补：

1. Word 纪要
2. CSV 跟进记录
3. JSON 结构化输出

## 当前可落地场景 / Current Production-Ready Scenarios

1. `互联网招聘里的面试反馈汇总与推进`
2. `JD + 简历初筛与推进建议`

这个场景覆盖了互联网招聘里最常见的一类 HR 痛点： / That scenario covers a very common recruiting pain point:

1. a recruiter or HRBP receives messy interviewer notes from Feishu, WeCom, email, or forms
2. the hiring manager wants a short hiring recommendation fast
3. HR needs a candidate-facing follow-up message
4. HR needs a tracker update that can be pasted into ATS or a spreadsheet
5. HR often still needs a downloadable debrief memo for internal circulation

当前附带了可直接使用的文件： / This skill includes bundled files for these scenarios:

1. [references/real-user-scenario.md](references/real-user-scenario.md)
2. [assets/interview-packet-input.sample.json](assets/interview-packet-input.sample.json)
3. [assets/resume-screening-input.sample.json](assets/resume-screening-input.sample.json)
4. [scripts/generate_interview_packet.js](scripts/generate_interview_packet.js)
5. [scripts/generate_screening_packet.js](scripts/generate_screening_packet.js)

第一次想快速理解这个 skill 怎么用，先看 [references/real-user-scenario.md](references/real-user-scenario.md)。

支持的动作有： / The supported actions are:

1. `score_candidate`
2. `summarize_interview_feedback`
3. `create_offer_approval_pack`
4. `generate_candidate_message`
5. `update_candidate_tracker`

## 运行时体验 / Runtime Experience

在交互中，不要只给最终答案。先让用户看见处理中步骤，再给结果。 / Do not only return the final answer. Show the user the work stages first.

推荐按这个顺序组织过程：

1. `正在读取材料`
2. `正在抽取关键字段`
3. `正在识别风险和分歧`
4. `正在生成推进结论与下一步`
5. `正在整理可复制文案和可回写记录`

## 输出标准 / Outcome Standard

处理任意招聘工作流时，始终产出以下结构： / When handling any recruiting workflow, always produce these sections:

```text
normalized_data
decision_summary
decision_basis
missing_information
risk_summary
next_action
message_draft
record_update
human_confirmation_needed
compliance_warning_if_any
```

规则： / Rules:

1. `normalized_data` must be structured and easy to map into ATS, Feishu Bitable, DingTalk approval forms, Notion, Google Sheets, or CSV.
2. `decision_summary` must make a decision or recommendation, not just restate the inputs.
3. `decision_basis` must explain why the recommendation was made, not just what the recommendation is.
4. `missing_information` must name the exact fields that block a confident HR action.
5. `risk_summary` must highlight the top reasons this candidate should be advanced, held, or reviewed again.
6. `next_action` must be something an HR operator can actually do today.
7. `message_draft` should be directly reusable in WeCom, Feishu, email, or a candidate chat.
8. `record_update` should be concise enough to write back into one row or one timeline entry.
9. `human_confirmation_needed` should state what still needs an HR or hiring manager to confirm before action.
10. `compliance_warning_if_any` should only appear when there is a concrete legal or privacy concern.

## 工作流路由 / Workflow Routing

### 1. `score_candidate`

当输入包含 JD 和简历或候选人资料时使用。 / Use when the input includes a JD and a resume or candidate profile.

常见输入形态： / Expected input shapes:

1. JD in Markdown, DOCX export, PDF text, or pasted text
2. Resume in PDF text, DOCX export, pasted text, or profile notes
3. Optional hiring preference such as `conservative`, `aggressive`, `fast-hiring`, or `high-bar`

至少抽取以下字段： / Always extract at least:

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

结果优先按下面 5 块展示：

1. `初筛结论`
2. `匹配依据`
3. `风险点`
4. `建议追问`
5. `候选人跟进话术`

### 2. `summarize_interview_feedback`

当输入包含多位面试官意见、零散备注、聊天记录或表单片段时使用。 / Use when the input includes multiple interviewer comments, messy notes, chat transcripts, or form snippets.

这是当前版本里最完整的工作流。如果用户只要一个真正能从输入跑到产出的场景，优先使用它。 / This is the most complete workflow in the current version. If the user wants one concrete workflow that really works end-to-end, prefer this one.

将反馈归一化为： / Normalize feedback into:

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

然后产出： / Then produce:

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

结果页优先展示：

1. `是否推进`
2. `依据`
3. `分歧点`
4. `补充追问`
5. `候选人沟通稿`

如果用户需要可下载文件，还要生成： / If the user wants downloadable artifacts, also generate:

1. an internal interview debrief memo in DOCX
2. a candidate communication draft in DOCX or plain text
3. a tracker update row in CSV

如果在这个 skill 仓库本地运行，使用： / If working locally in this skill repo, use:

```text
node scripts/generate_interview_packet.js <input.json> <output-dir>
```

示例输入见 [assets/interview-packet-input.sample.json](assets/interview-packet-input.sample.json)。 / See the sample payload in [assets/interview-packet-input.sample.json](assets/interview-packet-input.sample.json).

### `score_candidate` 也已支持文件产出

如果用户希望把初筛结论直接落成文件，可以生成：

1. 初筛评估 Word
2. 候选人初筛沟通稿
3. 进展记录 CSV

本地运行命令：

```text
node scripts/generate_screening_packet.js <input.json> <output-dir>
```

示例输入见 [assets/resume-screening-input.sample.json](assets/resume-screening-input.sample.json)。

### 3. `create_offer_approval_pack`

当 HR 已经掌握足够候选人信息、准备发 Offer 前的内部审批材料时使用。 / Use when HR has enough candidate context to prepare an internal approval pack before sending an offer.

重点检查以下缺失或风险字段： / Check for missing or risky fields such as:

1. target role or grade
2. department or reporting line
3. employment entity
4. work location
5. base salary or total cash
6. bonus, allowance, or stock notes
7. budget range
8. expected onboard date
9. trial period

始终产出： / Always produce:

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

当用户已经知道下一步动作，只需要一段面向候选人的沟通话术时使用。 / Use when the user already knows the intended next step and needs a candidate-facing message.

默认语气： / Default tone:

1. clear
2. respectful
3. concise
4. realistic about timing

支持的意图： / Supported intents:

1. invite to interview
2. request missing materials
3. advance to next round
4. hold in pipeline
5. reject politely
6. start offer discussion

优先风格：

1. 像中国招聘 HR 会真的发出去的话
2. 不要 AI 腔
3. 不要过度热情
4. 时间承诺要保守

### 5. `update_candidate_tracker`

当用户需要把结果回写到 ATS 或表格时使用。 / Use when the user wants a write-back summary for ATS or a spreadsheet.

默认 tracker 行格式： / Default tracker row format:

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

如果用户提供已有 tracker schema，则优先遵循用户 schema。 / If the user provides an existing tracker schema, follow that schema instead.

## 工作方式 / Working Style

1. 优先输出小而可执行的 HR 动作，而不是大段 SOP 说明。 / Prefer small executable HR actions over large SOP explanations.
2. 默认输入可能来自 PDF、DOCX 导出、OCR、聊天记录和表格单元格，且可能不完整。 / Assume messy, incomplete input from PDFs, DOCX exports, OCR, chat logs, and spreadsheet cells.
3. 先给结论，再给依据，再给下一步。 / Lead with the recommendation, then the basis, then the next step.
4. 优先给出招聘判断、风险和下一步，不追求空泛文采。 / Prioritize hiring decisions, risks, and next steps over pretty prose.
5. 仅在问题具体且实质时提示隐私或劳动法风险。 / Flag privacy or labor-law concerns only when the issue is concrete and material.
6. 如果材料不足或自相矛盾导致把握不高，要明确说出，并收窄建议。 / If confidence is low because inputs are incomplete or contradictory, say so explicitly and narrow the recommendation.
7. 不要假设用户会写 prompt，要主动给出下一步引导。 / Do not assume the user knows how to prompt; actively guide them to the next usable step.

## Mainland China context

Keep outputs aligned with common recruiting practice in China:

1. JDs often include hard requirements, preferred requirements, reporting line, city, and salary range.
2. Resumes often omit salary expectation, notice period, or true interview motivation.
3. Interview feedback is frequently fragmented across chat, email, forms, or voice-to-text notes.
4. Internal approval flows usually require concise business value, budget fit, and risk notes before an offer can move forward.

For common field shapes and examples, see [references/recruiting-fields.md](references/recruiting-fields.md).
For the detailed production-ready workflow, see [references/real-user-scenario.md](references/real-user-scenario.md).
