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

GUIDANCE 的结束标准是**问题定义已经足够清楚**，而不是某个输出格式已经确定。

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

不为了凑轮数追问。

## 3. Problem Definition ≠ Answer Method

GUIDANCE 确认的是：

~~~text
用户为什么提出这个问题
用户真正要解决什么
问题涉及什么范围
哪些背景/约束会影响问题
什么结果对用户才算有用
~~~

GUIDANCE 不要求用户决定：

~~~text
研究方法
分析框架
回答章节结构
图文 / 纯文字 / 表格 / JSON 等呈现形式
~~~

这些属于后续执行层。

如果用户已经明确给出了某种任务约束，可以保留为 Constraint；不要因为 Doctor 自己需要内部表示而再次询问。

## 4. User Confirmation

形成 Problem Definition / Diagnosis 后让用户确认。

确认内容：

~~~text
“我们的理解是否准确？”
~~~

而不是：

~~~text
“你希望我用什么方式研究？”
“你希望最终以什么格式输出？”
~~~

确认后：

~~~text
DOCTOR_DONE
~~~

这表示 Doctor 已完成职责。

没有默认 handoff。

用户可以结束、继续普通对话、自己制定方案，或主动使用其他能力。

## 5. 诊断被推翻

~~~text
Confirmed Diagnosis
→ new evidence / rejection
→ Model Update
→ Hypothesis
→ Evidence
→ Diagnosis
~~~

不要维护旧诊断。

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

二者都不要求包含研究方法、分析框架或呈现格式。
