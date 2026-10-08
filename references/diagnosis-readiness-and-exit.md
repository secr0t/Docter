# Diagnosis Readiness & Exit

## 1. Diagnosis Ready

确诊不是“某个假设概率最高”，而是：

`SymptomClear AND CompetingHypothesesDiscriminated AND RootCauseSupported AND CriticalUnknownsResolved`

满足以下条件后才能形成 Confirmed Diagnosis：

- 用户描述的现象已经明确；
- 关键竞争假设已经被证据区分；
- 根因有直接或充分间接证据支撑；
- 剩余未知不会明显改变问题分类或根因。

如果用户补充的新证据会改变诊断，必须回到 Hypothesis / Evidence 阶段重新诊断。

## 2. GUIDANCE Ready

GUIDANCE 的就绪条件是**问题已经问对**，不是字段填满：

- Background 足够明确；
- Goal 明确，不能只有 Direction；
- Scope / Expected Output 足以决定回答范围；
- 除非初始消息已经完整，否则至少完成 2 轮有效引导。

`EffectiveGuidanceRounds >= 2 OR InitialMessageComplete`

不为了凑轮数追问。

## 3. User Confirmation

形成 Problem Definition / Diagnosis 后，先让用户确认。

用户确认：

`DOCTOR_DONE → HANDOFF_TO_GRILL_ME`

handoff 只包含已确认的问题、证据与未决事项，不包含 Solution / Implementation。

## 4. 用户否定或新证据推翻

`Confirmed Diagnosis → rejected/new evidence → Model Update → Hypothesis → Evidence → Diagnosis`

不要维护旧诊断的“面子”，直接重新诊断。

## 5. Handoff 内容

推荐最小 handoff：

```json
{
  "problem": "",
  "problem_type": "",
  "root_cause": "",
  "evidence": [],
  "confidence": 0,
  "confirmed": true
}
```

GUIDANCE 则交付：

```json
{
  "goal": "",
  "scope": [],
  "expected_output": [],
  "next_step": ""
}
```

两者均不得包含 solution / fix / implementation / code / architecture / remediation。
