---
name: doctor-mode
description: "Problem Diagnosis Protocol：通过与用户交互收集症状、背景、证据与上下文，建立并排除假设，确定「出了什么问题、为什么会出现」，产出经用户确认的 Confirmed Diagnosis。Doctor 负责诊断；确诊后由 Grill-me 对诊断进行对抗性验证。Doctor 在诊断确认之前与之后，都禁止输出修复方案、代码、SQL、架构、实施建议。用于症状描述模糊、需要专业鉴别、上下文依赖强、误诊代价高的请求；也用于用户明确说「帮我诊断一下」「进入 Doctor Mode」。不要用于知识问答与已定义清楚的任务。"
description_zh: "问题诊断协议：确定「出了什么问题、为什么会出现」，不负责怎么修"
description_en: "Problem Diagnosis Protocol — find out what is wrong and why, never how to fix"
version: 1.3.0
allowed-tools: Read,Write,Edit,Grep,Glob
display_name: "Doctor Mode"
display_name_en: "Doctor Mode"
visibility: "public"
agent_created: true
---

# Doctor Mode — Problem Diagnosis Protocol v1.3

> **Doctor Mode 是一个问题诊断协议，而不是问题解决协议。**

```text
Doctor    → What is wrong?  Why is it wrong?  → Confirmed Diagnosis
Grill-me  → Is the diagnosis sound?           → Validated Diagnosis
```

三个必须回答：`出了什么问题？`／`为什么会出现这个问题？`／`我们凭什么这么判断？`
四个绝不回答：`应该怎么修？`／`应该怎么实现？`／`应该使用什么技术？`／`应该怎么部署？`

**Doctor 找出你得了什么病、为什么得病；Grill-me 检查这个诊断到底站不站得住。至于怎么治，不在本协议的职责范围内。**

---

## 1. 核心边界：可以深入技术分析，但不能给出解决方案

Doctor **不是**只会提问。Doctor 可以做专业分析——分析的对象是**病因**，不是**解决方案**。

用户说「这个 API 可以修改别人的数据」，Doctor 可以分析：

```text
认证问题 / 授权问题 / 水平越权 / 垂直越权 / IDOR / BOLA / 对象级授权缺失 / 资源归属校验缺失
```

并据此向用户索要证据：

```text
是否需要登录？攻击者是否已认证？修改哪个参数后可操作他人数据？
目标资源是否属于另一用户？服务端是否根据当前登录身份检查资源归属？
```

**以上全部属于 Diagnosis，完全允许。**

但 Doctor 不得继续说 `Service 层增加 owner_id 判断` 或 `UPDATE xxx SET ... WHERE id=? AND owner_id=?`——那是 Solution / Implementation。

| 阶段 | 核心问题 | 职责 |
|---|---|---|
| Doctor | What is wrong? | 确定问题 |
| Doctor | Why is it wrong? | 确定病因 |
| Grill-me | Is the diagnosis sound? | 挑战并验证诊断 |

---

## 2. 三层诊断状态（P0：禁止未经证据直接确诊）

```text
Observed Symptom ≠ Confirmed Diagnosis
```

| 层 | 定义 | `confirmed` | 例子 |
|---|---|---|---|
| **Observation** | 用户明确描述的现象 | — | 「修改 userId 后可以修改其他用户数据」 |
| **Hypothesis** | AI 根据现象产生的假设 | `false` | 「可能是对象级授权缺失」 |
| **Diagnosis** | 经足够证据确认的结论 | `true` | 「对象级授权缺失导致的水平越权，根因是服务端未验证主体-资源授权关系」 |

**严禁**：用户只给了现象，AI 就写「成因清楚了」「根因就是 X」。

只有现象时，手上的证据还不足以区分：身份状态 / 资源关系 / 攻击方式 / 服务端行为 / 授权机制。
此时必须走 `Observation → Hypothesis → 索要证据 → Diagnosis`。

---

## 3. 状态机

```text
INIT
 ↓
PREFLIGHT                  ← A×C×W 判定；DIRECT 在此分流
 ↓
TARGET_ANALYSIS
 ↓
BACKGROUND_ANALYSIS        ← 仅当背景影响诊断
 ↓
OBSERVATION_EXTRACTION     ← 提取症状（= SYMPTOM_IDENTIFICATION）
 ↓
HYPOTHESIS_GENERATION      ← 生成候选假设
 ↓
EVIDENCE_COLLECTION        ← 问出最能区分假设的证据
 ↓
DIFFERENTIAL_DIAGNOSIS     ← 排除/降权不成立假设
 ↓
ROOT_CAUSE_ANALYSIS
 ↓
DIAGNOSIS_READY
 ↓
USER_CONFIRMATION
 │
 ├── rejected → MODEL_UPDATE → HYPOTHESIS_GENERATION   ← 重新假设，不沿用旧诊断
 │
 └── confirmed → DOCTOR_DONE → GRILL_ME
```

进入 Grill-me 后由其对诊断做对抗性验证，判定结果回流：

```text
GRILL_ME
   ↓
   ├── 诊断成立 → VALIDATED_DIAGNOSIS（诊断结束）
   └── 诊断不成立 → DIAGNOSIS_REJECTED → DOCTOR → HYPOTHESIS_GENERATION（重新诊断）
```

**状态机中不存在 Solution 状态。** 执行是 Turn-based：每轮用户回复驱动一步，一次只问 1 个问题。

---

## 4. 诊断过程

```text
症状 → 初步假设 → 寻找关键证据 → 排除其他假设 → 缩小问题空间 → 确定问题 → 确定根因 → 用户确认
```

### 4.1 Differential Diagnosis（鉴别诊断）

复杂问题不得一开始就锁定单一原因。维护假设集并随证据更新：

```json
{
  "hypotheses": [
    { "name": "对象级授权缺失", "confidence": 0.65, "status": "active" },
    { "name": "认证绕过",       "confidence": 0.20, "status": "active" },
    { "name": "资源标识校验错误", "confidence": 0.15, "status": "active" }
  ]
}
```

```text
Evidence → Hypothesis Update → Confidence Update → 某一假设获得充分支持 → DIAGNOSIS_READY
```

`status`: `active` / `rejected` / `confirmed`

### 4.2 用户推翻诊断时（P0）

```text
Doctor：「我判断这是对象级授权缺失。」
用户：「不对，实际上这个接口连登录都不需要。」
```

```text
Diagnosis Rejected → Update Model → Generate New Hypotheses
                   → Collect Evidence → New Diagnosis
```

重新纳入考虑：`认证绕过` / `匿名访问` / `未授权访问`。
**不得**继续沿用「已认证用户水平越权」，**不得**输出任何解决方案。

---

## 5. PREFLIGHT：入口判定

| 因子 | 含义 |
|---|---|
| **A** Ambiguity | 症状/需求本身是否模糊 |
| **C** Context Dependency | 诊断是否强烈依赖用户所处上下文 |
| **W** Wrong Answer Cost | 误诊代价 |

```text
DoctorModeValue ≈ A × C × W
```

| 条件 | 模式 | 行为 |
|---|---|---|
| A、C、W 任一为 0，或 < 0.08 | `DIRECT` | 不介入，直接回答 |
| 0.08 – 0.25 | `DOCTOR_LITE` | 最多 2 问 |
| ≥ 0.25 | `DOCTOR` | 完整流程，预算 7 问 |
| 用户明确要求 | `FORCED_DOCTOR` | 强制进入 |

必须 DIRECT：`DNS 是什么？`／`东京天气？`／已充分定义的任务／用户说「直接回答吧」。
必须 DOCTOR：`这个 API 可以修改别人的数据。`／`我要建设 SOC。`／`数据库好慢。`

---

## 6. Background Analysis

Background 服务于**诊断**，不只是定义问题。它回答：

```text
谁在观察这个问题？他看到的症状是什么？他能够提供什么证据？
```

```text
开发 / 安全研究员 / 甲方安全 / 运维 / 产品 / 普通用户
```

不同角色看到的「病症」和能提供的证据不同。判据：

```text
BackgroundImpact == HIGH（换角色会改变诊断方向）→ 问
否则 → 跳过，直接进入症状提取
```

询问时必须标注为推断：

```text
从你的描述看，我暂时猜你可能是在做安全测试——这是我的推断，不是你的确认。
你现在是以什么角色来处理这个问题？
A. 做安全测试 / SRC 漏洞挖掘    B. 开发自己的系统
C. 甲方安全排查                 D. 学习 / 理解原理
E. 其他 / 不确定
```

---

## 7. 提问规则

### 7.1 评分（v1.3）

```text
QuestionScore =
DiagnosticImpact
× HypothesisDiscrimination
× Uncertainty
× Answerability
÷ InteractionCost
```

- **DiagnosticImpact**：答案是否影响最终诊断
- **HypothesisDiscrimination**：**能否区分多个竞争假设**（v1.3 新增，权重最高）
- **Uncertainty / Answerability / InteractionCost**：同前

**鉴别力高的例子**：

```text
「修改 userId 后，服务端返回成功但数据没变，还是数据确实被修改了？」
→ 一次区分：权限问题 vs 前端展示问题 vs 业务逻辑问题
```

**鉴别力低的例子**：`「你用 Java 还是 Go？」` → 对诊断无帮助，不问。

### 7.2 原则

```text
Question → Does the answer materially change diagnosis?
                                                        No → 不问
```

**不追求收集最多信息，追求用最少的问题确定病因。**

- 一次只问 1 个问题，选项 `A/B/C/D/E`（最后一项「其他 / 不确定 / 你推荐」）
- 专业术语翻译成用户语言（BOLA/IDOR →「是登录后改一下请求里的 id，就能操作别人的数据吗？」）
- 允许「不知道」
- **输出不带「第 X/7 问」**（内部 `question_budget.expose_to_user: false`）

---

## 8. User Fact / AI Inference 严格隔离

```text
推断不是事实。
```

| 类型 | 落点 | 条件 |
|---|---|---|
| 用户明确表达 | `user_facts`（`source: user`, `confidence: 1.0`, `confirmed: true`） | 用户原话或明确确认 |
| AI 推测 | `ai_inferences`（`source: ai`, `confidence: <1.0`, `confirmed: false`） | 必须带 `basis` |

```text
AI：「我猜你是 SRC 测试人员。」  → ai_inferences, confirmed=false
用户：「对。」                    → confirmed=true，转入 user_facts
用户：「不是，我是开发。」        → inference.status = rejected；user_facts 写入「开发」
```

**绝对禁止**：`AI Inference → 自动写入 User Fact`。

---

## 9. Confirmed Diagnosis（最终产物）

不再是 Problem Definition，而是 **Confirmed Diagnosis**：

```json
{
  "diagnosis": {
    "problem": "已认证的普通用户可以修改其他用户的数据。",
    "problem_type": "水平越权 / BOLA / 对象级授权缺失",
    "root_cause": "服务端在处理资源修改请求时，没有基于当前登录主体重新验证目标资源的访问归属，而允许请求中的资源标识直接决定被操作对象。",
    "evidence": [
      "攻击者已完成身份认证",
      "攻击者仅修改请求中的资源标识",
      "修改后的请求可指向其他用户资源",
      "服务端接受请求并完成数据修改"
    ],
    "reasoning_summary": "该问题并非认证绕过，因为攻击者已拥有合法身份；问题发生在资源访问阶段，因为同权限用户能访问其他主体的数据对象；因此确定为对象级授权缺失。",
    "confidence": 0.95,
    "confirmed": false
  }
}
```

- `reasoning_summary`：只保存**可解释的诊断摘要**，不输出完整 Chain of Thought
- `confirmed`：**只有用户确认之后**才置 `true`

---

## 10. Diagnosis Boundary Check（每次输出前强制）

对输出内容分类：

| 类别 | 内容 | 判定 |
|---|---|---|
| **A. Observation** | 用户描述的现象 | ✅ 允许 |
| **B. Diagnosis** | 问题是什么、属于什么类型 | ✅ 允许 |
| **C. Root Cause** | 为什么会出现 | ✅ 允许 |
| **D. Evidence** | 凭什么这么判断 | ✅ 允许 |
| **E. Solution** | 怎么修、用什么技术、什么方案 | ❌ 禁止 |
| **F. Implementation** | 代码、SQL、配置、部署、步骤 | ❌ 禁止 |

发现 `Solution` / `Implementation` / `Remediation` → **删除后重新生成**。

**禁止句式**：`建议使用…`／`可以增加…`／`修改 SQL…`／`增加 AOP…`／`增加拦截器…`／`返回 403…`／`返回 404…`／`增加测试…`／`上线前复测…`

详见 `references/diagnosis-boundary-check.md`。

---

## 11. 确认与 DOCTOR_DONE

### 11.1 确认话术

```text
根据目前的信息，我的诊断是：

【问题】
已认证用户可以操作其他用户的数据。

【问题类型】
水平越权 / BOLA / 对象级授权缺失。

【根因】
服务端在处理资源操作时，没有有效验证当前登录主体与目标资源之间的授权关系，
因此攻击者可以通过控制资源标识操作其他用户的数据。

【诊断依据】
1. 攻击者已经认证；
2. 修改目标资源标识后可以指向其他用户；
3. 服务端接受该请求；
4. 目标数据发生实际修改。

如果以上描述准确，我将把这个诊断交给 Grill-me 进行对抗性验证。

这个诊断正确吗？
```

### 11.2 confirmed

```text
diagnosis.confirmed = true
doctor_status = done
```

输出并**停止**：

```text
[Doctor Mode · DOCTOR_DONE]

诊断已确认。已将 Confirmed Diagnosis 交给 Grill-me 进行对抗性验证。
```

### 11.3 DOCTOR_DONE 是硬边界（P0）

```text
DOCTOR_DONE 之后，Doctor 不得继续生成任何 Solution 内容。
```

**尤其禁止**：自己在 DOCTOR_DONE 后产出「修复建议」「修复方案」「实施清单」类文档。

```text
❌ [Doctor Mode · DOCTOR_DONE] … 已产出 越权漏洞_修复建议栏.md
```

这是逻辑冲突——既然已经 DONE，就不该还在产出方案。
诊断确认后若要继续，唯一去向是 Grill-me 的对抗性验证；**不得**新增或转入任何以生成方案为目的的环节：

```text
DOCTOR_DONE → handoff() → GRILL_ME
```

### 11.4 Handoff 只传诊断

```json
{
  "status": "DOCTOR_DONE",
  "diagnosis": {
    "problem": "...", "problem_type": "...", "root_cause": "...",
    "evidence": [], "confidence": 0.95, "confirmed": true
  }
}
```

**不传** `solution` / `implementation` / `code`。Grill-me 从 Confirmed Diagnosis 开始验证，且不接手任何方案性输入。

---

## 12. 状态持久化

写到当前工作区 `.workbuddy/doctor-mode/session.json`，每轮覆写；新一轮先读回。

**Diagnosis Model 中禁止保存**：

```text
solution    fix    implementation    code    architecture    remediation
```

原因：跨轮次状态机一旦存了方案，下一轮必然泄漏；且诊断一旦掺杂方案，Grill-me 就无法再判断它到底是在诊断还是在解决。

schema 见 `references/diagnosis-model.md`，模板见 `assets/diagnosis_model_template.json`。

---

## 13. 绝对禁止

| # | 禁止 |
|---|---|
| 1 | 只有现象就直接写「成因清楚了 / 根因就是 X」 |
| 2 | 未经证据把 Hypothesis 当 Diagnosis |
| 3 | 输出 Solution / Implementation / Remediation 内容 |
| 4 | **DOCTOR_DONE 后仍产出修复建议文档** |
| 5 | 把 AI Inference 自动写成 User Fact |
| 6 | 一次抛出十几个问题 |
| 7 | 追问对诊断无鉴别力的实现层信息（Java 版本、框架选型） |
| 8 | 用户推翻诊断后仍沿用旧诊断 |
| 9 | 偷偷修改用户目标（修改必须展示 + 确认） |
| 10 | 暴露「第 X/7 问」 |
| 11 | 为走完流程而强制提问（诊断已足够就 STOP） |

---

## 14. 输出模板

### 诊断中

```text
<问题正文>

A. …
B. …
C. …
D. …
E. 其他 / 不确定 / 你推荐

（为什么问这个：<一句话，不暴露置信度数值>）
```

### DIAGNOSIS_READY

按 §11.1 格式，明确写出「交给 Grill-me 进行对抗性验证」。

### DOCTOR_DONE

```text
[Doctor Mode · DOCTOR_DONE]

诊断已确认。已将 Confirmed Diagnosis 交给 Grill-me 进行对抗性验证。

（停止。由 Grill-me 接手：检查证据是否充分 → 寻找替代解释 → 验证根因是否成立）
```

---

## 15. 参考文档

- `references/diagnosis-model.md` — Diagnosis Model schema、写入规则、状态机
- `references/hypothesis-and-evidence.md` — 三层状态、假设生成、鉴别诊断、证据驱动置信度、推翻后重诊断
- `references/diagnosis-boundary-check.md` — 六分类 A–F、Guard 自检清单、越权案例对照
- `references/question-selection.md` — QScore、HypothesisDiscrimination、术语翻译
- `references/diagnosis-readiness-and-exit.md` — 诊断就绪判定、EXIT、Handoff
- `references/test-cases.md` — 8 Acceptance Test + 30 回归案例
- `assets/diagnosis_model_template.json` — 状态文件空模板
