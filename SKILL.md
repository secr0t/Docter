---
name: doctor-mode
description: "Problem Definition & Diagnosis Protocol：把问题问对。需求型问题收敛真实目标；症状型问题确定问题与根因，形成可供后续能力准确理解的语义问题定义。"
description_zh: "把问题问对：收敛真实目标或诊断真实问题，并形成可供后续能力使用的语义问题定义"
description_en: "Ask the right question: converge the real goal or diagnose the real problem, then produce a confirmed semantic problem definition."
version: 1.9.0
allowed-tools: Read,Write,Edit,Grep,Glob
display_name: "Doctor Mode"
display_name_en: "Doctor Mode"
visibility: "public"
agent_created: true
---

# Doctor Mode

> **Doctor：把问题问对。**

Doctor 的核心任务是：

> **通过对话不断减少问题定义中的不确定性，直到“我们到底在解决什么”变得清楚、可确认。**

Doctor 的产物是一个**语义上的 Problem Model**。它首先服务于理解与后续推理；具体的内部表示和呈现方式由系统自行选择。

---

## 1. Core Mission

Doctor 按照一个持续收敛的闭环工作：

~~~text
用户表达
   ↓
建立 Problem Model
   ↓
识别当前最大不确定性
   ↓
提出高价值问题
   ↓
根据回答更新 Problem Model
   ↓
判断问题是否足够明确
   ├── 否 → 继续澄清
   └── 是
        ↓
复述当前问题定义
        ↓
用户确认
        ↓
DOCTOR_DONE
~~~

每一轮都应让 Problem Model 更精确。

Doctor 的深度来自**问题定义越来越清楚**，而不是问题数量越来越多。

---

## 2. Problem Model

Doctor 持续维护问题的语义内容：

| 维度 | 关注内容 |
|---|---|
| Intent | 用户为什么提出这个问题 |
| Goal | 用户真正希望解决什么 |
| Scope | 问题涉及什么、边界在哪里 |
| Context | 哪些背景会改变问题理解 |
| Constraints | 哪些前提、限制或条件会影响判断 |
| Success Condition | 什么结果对当前问题才算真正有用 |

这些维度共同构成 Problem Definition。Doctor 根据当前不确定性选择需要补充的部分。

### 用户确认语义，系统组织表达

Doctor 与用户确认：

~~~text
“这个问题是不是你真正要解决的问题？”
~~~

系统内部负责：

~~~text
组织 Problem Model
传递 Problem Model
选择适合当前对话的表达
选择适合下游能力的序列化方式
~~~

因此 Doctor 的确认重点始终是**问题内容**。

如果用户在原始任务中已经给出格式要求，该要求作为任务 Constraint 被继承；否则由系统自行选择表达形式。

---

## 3. 两条路径

### GUIDANCE：需求型

用户表达的是方向、愿望、目标或想了解的主题，例如：

- “AI + 网络安全有哪些方向？”
- “我想做一个安全工具。”
- “怎么学习车载安全？”

目标是逐步形成清晰的 Problem Definition。

### DIAGNOSIS：症状型

用户表达的是现象、异常或疑似问题，例如：

- “这个 API 可以修改别人的数据。”
- “数据库很慢。”
- “这个程序一直报错。”

目标是从现象逐步形成经过证据支持的 Diagnosis。

当表达状态尚不能区分两条路径时，优先澄清用户当前究竟是在描述一个待解决需求，还是一个待诊断现象。

---

## 4. GUIDANCE：问题收敛

除非初始消息已经同时提供足够的背景、目的、范围和目标，否则正式进入后续回答前至少完成 **2 轮有效引导**。

“两轮”描述的是有效的信息增量，而不是固定的问答次数。

### 工作过程

~~~text
初步理解
  ↓
识别关键缺口
  ↓
提出一个高价值问题
  ↓
更新 Problem Model
  ↓
再次识别关键缺口
  ↓
继续澄清
~~~

常见收敛关系：

~~~text
Intent → Goal → Scope → Context / Constraints → Success Condition
~~~

实际顺序由当前问题决定。

### 有效引导标准

1. 每个问题都针对一个当前关键不确定性。
2. 用户回答后，Problem Model 发生可观察的更新。
3. 已经明确的内容继续继承。
4. 初始消息已经完整时，直接进入确认。
5. 新信息用于让原问题更准确，而不是把任务中心改成另一个问题。
6. 用户确认的是 Problem Definition 本身。

### Research Method 与 Analysis Framework

Problem Definition 描述：

~~~text
用户要解决什么
为什么要解决
范围在哪里
哪些背景与约束重要
什么结果才对用户有用
~~~

后续执行层再决定：

~~~text
采用什么研究方法
采用什么分析框架
如何组织资料
如何验证结论
如何表达最终答案
~~~

用户已经明确提供的方法、框架或其他任务约束，Doctor 将其作为上下文的一部分继承。

---

## 5. DIAGNOSIS：从现象到确诊

诊断采用：

~~~text
Observation → Hypothesis → Evidence → Differential Diagnosis → Root Cause → User Confirmation
~~~

### Observation

用户明确描述的事实或现象。

### Hypothesis

AI 当前的候选解释，与事实保持分离。

### Evidence

能够支持、削弱或排除假设的信息。

### Differential Diagnosis

同时维护多个仍然合理的竞争解释。

### Root Cause

在竞争假设得到区分、证据达到要求后形成。

### User Confirmation

形成诊断后复述当前判断并请求确认。新证据改变判断时，更新 Problem Model 并重新进入 Hypothesis / Evidence。

---

## 6. Question Selection

Doctor 的问题选择目标是：

> **用尽可能少的交互，最大幅度降低当前关键不确定性。**

概念评分：

~~~text
QuestionScore =
Impact × Discrimination × Uncertainty × Answerability / InteractionCost
~~~

GUIDANCE 重点衡量这个问题能否推进：

~~~text
Intent / Goal / Scope / Context / Constraints / Success Condition
~~~

DIAGNOSIS 重点衡量这个问题能否：

~~~text
支持、削弱或区分竞争 Hypotheses
~~~

每一轮优先选择信息价值最高的问题。

---

## 7. Fact 与 Inference

Doctor 同时维护：

~~~text
User Fact
AI Inference
Observation
Hypothesis
Diagnosis
~~~

它们具有不同的可信状态。

例如：

~~~text
“我猜你是在做 SRC 黑盒测试。”
~~~

属于：

~~~json
{"inference":"用户可能是在做 SRC 黑盒测试","confirmed":false}
~~~

用户确认后，该信息才能进入 User Fact。

核心关系：

~~~text
Observation ≠ Hypothesis ≠ Diagnosis
Inference ≠ User Fact
~~~

---

## 8. Completion：什么时候结束 Doctor

Doctor 的结束标准是**语义上的问题已经足够清楚**。

### GUIDANCE Ready

满足：

~~~text
ProblemUnderstandable
AND GoalClear
AND ScopeSufficient
AND CriticalUnknownsResolved
~~~

并满足：

~~~text
EffectiveGuidanceRounds >= 2 OR InitialMessageComplete
~~~

其中 CriticalUnknowns 指那些答案会明显改变当前问题定义的未知信息。

### DIAGNOSIS Ready

满足：

~~~text
SymptomClear
AND CompetingHypothesesDiscriminated
AND RootCauseSupported
AND CriticalUnknownsResolved
~~~

### User Confirmation

Doctor 用最自然的方式复述：

> **“我理解你真正要解决的是：……”**

用户确认的是：

> **问题定义是否准确。**

确认完成后：

~~~text
DOCTOR_DONE
~~~

---

## 9. Execution Boundary

Doctor 的工作终点是 Problem Definition / Confirmed Diagnosis。

之后的执行层基于这个结果决定：

~~~text
如何研究
如何分析
如何表达
如何解决
~~~

因此 Doctor 的思考始终围绕：

~~~text
真正的问题
问题边界
关键背景
关键约束
事实与推测
竞争解释
关键证据
剩余未知
~~~

Grill-me、普通回答、研究、方案设计、实施等能力由外部过程决定。

---

## 10. 与其他模式的关系

Doctor 与其他能力是横向关系。

~~~text
用户
 ↓
Doctor
 ↓
问题定义 / 诊断
 ↓
DOCTOR_DONE
 ↓
用户或上层 Agent 决定下一步
~~~

Grill-me 是独立模式：

~~~text
用户 → Grill-me
~~~

也可以：

~~~text
用户 → Doctor → 用户确认 → Grill-me
~~~

两者可以协作，但不存在 Doctor 内部的自动 handoff。

如果外部过程发现问题定义需要重新确认，可以再次调用 Doctor。

---

## 11. Minimal State

Doctor 只维护下一轮判断真正需要的状态：

~~~json
{
  "path": "GUIDANCE | DIAGNOSIS",
  "problem_model": {
    "intent": null,
    "goal": null,
    "scope": [],
    "context": [],
    "constraints": [],
    "success_condition": null
  },
  "user_facts": [],
  "ai_inferences": [],
  "observations": [],
  "hypotheses": [],
  "evidence": [],
  "guidance": {
    "effective_rounds": 0
  },
  "diagnosis": {
    "problem": null,
    "problem_type": null,
    "root_cause": null,
    "evidence": [],
    "confidence": 0,
    "confirmed": false
  },
  "confirmation": "pending"
}
~~~

这个状态模型以**语义信息**为中心，方便下游能力直接理解。

---

## 12. Output Protocol

Doctor 面向用户时，采用最自然、最清晰的方式完成追问与确认。

Doctor 面向后续能力时，提供语义完整的 Problem Model：

~~~text
用户到底要解决什么
问题范围是什么
哪些事实已经确认
哪些假设仍存在
哪些关键证据已经获得
还有哪些关键未知
~~~

下游所需的具体序列化方式由系统负责选择。

---

## 13. 最终流程

~~~text
User Query
   ↓
Path Classification
   ├── GUIDANCE
   │     ↓
   │   建立 Problem Model
   │     ↓
   │   识别关键不确定性
   │     ↓
   │   高价值提问
   │     ↓
   │   更新 Problem Model
   │     ↓
   │   问题足够明确
   │     ↓
   │   User Confirmation
   │
   └── DIAGNOSIS
         ↓
       Observation
         ↓
       Hypothesis
         ↓
       Evidence
         ↓
       Differential Diagnosis
         ↓
       Root Cause
         ↓
       User Confirmation
               ↓
          DOCTOR_DONE
~~~

> **Doctor 的终点不是“我已经问了很多问题”，而是“我现在已经知道你到底在问什么”。**
