---
name: cn-recruiting-workflow
description: 帮招聘 HR 整理面试反馈、初筛简历、生成候选人沟通话术和跟进记录，把今天要推进的招聘动作往前推一步。 / Help recruiting teams debrief interviews, screen resumes, draft candidate follow-ups, and update trackers.
version: 0.5.0
metadata:
  openclaw:
    homepage: https://github.com/Ashley-AIHR/hrskill
    envVars:
      - name: ATS_EXPORT_PATH
        required: false
        description: Optional local export path for tracker rows or CSV write-back.
---

# 招聘推进助手 / Recruiting Follow-up Copilot

当用户在处理招聘推进、面试反馈汇总、简历初筛、候选人沟通或 Offer 前置材料时使用这个 skill。

目标不是解释招聘概念，而是把材料整理成：

1. 结论
2. 依据
3. 风险
4. 下一步
5. 可直接发送或回写的内容

如果用户第一次使用或输入很乱，先读 [references/real-user-scenario.md](references/real-user-scenario.md)。
如果需要字段归一化，读 [references/recruiting-fields.md](references/recruiting-fields.md)。
如果需要检验工作流是否符合中国互联网招聘常见链路，读 [references/china-recruiting-workflow-benchmark-2026.md](references/china-recruiting-workflow-benchmark-2026.md)。

## 路由规则

根据输入内容路由到下面动作之一：

1. `score_candidate`
   触发条件：输入里有 `JD + 简历` 或候选人资料。
2. `summarize_interview_feedback`
   触发条件：输入里有多位面试官反馈、聊天记录、表单备注、语音转文字。
3. `create_offer_approval_pack`
   触发条件：输入里有候选人信息、面试结论、预算、拟 offer 条件。
4. `generate_candidate_message`
   触发条件：用户已经知道下一步动作，只需要候选人沟通话术。
5. `update_candidate_tracker`
   触发条件：用户要 ATS、表格或 timeline 回写内容。

如果用户不知道该选哪个动作：

1. 有面试反馈就优先走 `summarize_interview_feedback`
2. 有 JD 和简历就优先走 `score_candidate`
3. 只要消息稿就走 `generate_candidate_message`

## 输出协议

处理任意招聘场景时，始终输出：

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

要求：

1. `decision_summary` 必须直接回答“推进 / 保留 / 补面 / 拒绝 / 准备 offer”。
2. `decision_basis` 必须说明依据，不要只重复结论。
3. `missing_information` 只写真正会影响动作判断的缺口。
4. `risk_summary` 优先写硬性卡点，再写软性担忧。
5. `next_action` 必须是 HR 今天能执行的动作。
6. `message_draft` 必须像中国招聘 HR 真的会发出去的话，不要 AI 腔。
7. `record_update` 必须足够短，适合回写一行 ATS 或一条 timeline。
8. `human_confirmation_needed` 必须明确还要谁确认什么。

## 动作要求

### `score_candidate`

至少抽取：

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

结果优先顺序：

1. 初筛结论
2. 匹配依据
3. 风险点
4. 建议追问
5. 候选人跟进话术

如果需要文件产出，运行：

```text
node scripts/generate_screening_packet.js <input.json> <output-dir>
```

示例输入： [assets/resume-screening-input.sample.json](assets/resume-screening-input.sample.json)

### `summarize_interview_feedback`

至少抽取：

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

结果优先顺序：

1. 是否推进
2. 依据
3. 分歧点
4. 补充追问
5. 候选人沟通稿

如果需要文件产出，运行：

```text
node scripts/generate_interview_packet.js <input.json> <output-dir>
```

示例输入： [assets/interview-packet-input.sample.json](assets/interview-packet-input.sample.json)

### `create_offer_approval_pack`

至少检查：

1. 岗位和职级
2. 用工主体
3. 工作地
4. base salary / total cash
5. 奖金或补贴说明
6. 预算范围
7. 入职日期
8. 试用期

输出：

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

### `generate_candidate_message`

支持：

1. 邀约面试
2. 推进下一轮
3. 追回材料
4. 保留池
5. 婉拒
6. 开启 offer 沟通

默认风格：

1. 清楚
2. 尊重
3. 简短
4. 时间承诺保守

### `update_candidate_tracker`

默认回写格式：

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

如果用户已经给了 tracker schema，优先遵循用户 schema。

## 工作原则

1. 先给结论，再给依据，再给下一步。
2. 输入默认是脏的，先归一化，不要抱怨格式。
3. 面试官意见互相矛盾时，必须明确指出分歧。
4. 信息不足时，不要硬下判断，要收窄结论。
5. 只在有具体风险时提示隐私或合规问题。
