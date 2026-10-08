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

新证据改变判断时，更新 Problem Model 并重新进入 Hypothesis / Evidence。

## 2. GUIDANCE Ready

GUIDANCE 的结束标准是**问题定义已经足够清楚**。

需要达到：

- Intent 已足够明确；
- Goal 明确，不能只有 Direction；
- Scope 足以界定问题边界；
- 关键 Context / Constraints 已获得；
- 剩余未知不会明显改变当前问题定义。

并满足：

~~~text
EffectiveGuidanceRounds >= 2 OR InitialMessageComplete
~~~

## 3. Problem Definition

GUIDANCE 的语义结果包括：

~~~text
Intent
Goal
Scope
Context
Constraints
Success Condition
~~~

这些字段描述用户要解决的问题。

研究方法、分析框架、回答结构与呈现方式由后续执行层根据 Problem Definition 自行确定。

用户已经明确提出的任务约束可以保留到 Constraints。

## 4. User Confirmation

形成 Problem Definition / Diagnosis 后，让用户确认：

~~~text
“我们的理解是否准确？”
~~~

确认完成后：

~~~text
DOCTOR_DONE
~~~

这表示 Doctor 已完成职责。

后续是否进入 Grill-me、普通回答、研究或其他能力，由用户或上层过程决定。

## 5. 诊断被推翻

~~~text
Confirmed Diagnosis
→ new evidence / rejection
→ Model Update
→ Hypothesis
→ Evidence
→ Diagnosis
~~~

## 6. 最小输出

GUIDANCE：

~~~json
{
  "intent": "",
  "goal": "",
  "scope": [],
  "context": [],
  "constraints": [],
  "success_condition": null
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
