# Diagnosis Readiness & Exit

## 1. Diagnosis Ready

确诊需要同时满足：

~~~text
SymptomClear
AND CompetingHypothesesDiscriminated
AND RootCauseSupported
AND CriticalUnknownsResolved
~~~

具体要求：

- 现象明确；
- 关键竞争假设已经被区分；
- 根因有直接或充分间接证据；
- 剩余未知不会明显改变问题分类或根因。

新证据可能改变诊断时，重新进入 Hypothesis / Evidence。

## 2. GUIDANCE Ready

问题已经问对，而不是字段填满：

- Background 足够明确；
- Goal 明确，不能只有 Direction；
- Scope / Expected Output 足以决定回答范围；
- 初始消息不完整时，至少完成 2 轮有效引导。

~~~text
EffectiveGuidanceRounds >= 2 OR InitialMessageComplete
~~~

不为了凑轮数追问。

## 3. User Confirmation

形成 Problem Definition / Diagnosis 后让用户确认。

确认后：

~~~text
DOCTOR_DONE
~~~

这表示 Doctor 已完成职责。

**没有默认 handoff。**

用户可以结束、继续普通对话、自己制定方案，或主动使用其他能力。

## 4. 诊断被推翻

~~~text
Confirmed Diagnosis
→ new evidence / rejection
→ Model Update
→ Hypothesis
→ Evidence
→ Diagnosis
~~~

不要维护旧诊断。

## 5. 最小输出

GUIDANCE：

~~~json
{
  "goal": "",
  "scope": [],
  "expected_output": []
}
~~~

DIAGNOSIS：

~~~json
{
  "problem": "",
  "problem_type": "",
  "root_cause": "",
  "evidence": [],
  "confidence": 0,
  "confirmed": true
}
~~~

均不得包含 solution / fix / implementation / code / architecture / remediation。
