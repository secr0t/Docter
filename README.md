# Doctor Mode — Problem Definition Protocol v1.6

> **Doctor 的核心任务：把问题问对。**
> **Doctor 的深度不是"问得越来越深"，而是"需求越来越清楚"。**
>
> **这是一套「定义问题」的协议，不是「解决问题」的协议。Doctor 交付一份定义清楚、经用户确认的问题，然后结束。**

对应规格：《Doctor Mode v1.2 诊断模式修复规格》。版本沿革：v1.2.0 为 Problem/Solution 边界版，v1.3.0 升级为 Problem Diagnosis，v1.4.0 移除 Solver 阶段，v1.5.0 引入引导与问题收敛机制（GUIDANCE 路径），**v1.6.0 移除 Grill-me 耦合，确立「只定义问题」的协议边界**。

## v1.6 的核心变更：Doctor 只到"问题定义清楚"为止

本技能的目的**不是**解决问题，而是**完善与定义问题**。因此 Doctor 不挂载、不预设任何解题性质的下游阶段。

```text
用户问题
   ↓
Doctor
   ↓
问题定义清楚（Problem Definition / Confirmed Diagnosis）
   ↓
DOCTOR_DONE  ← 协议唯一终态
```

- v1.4 已移除 `Solver`（方案 → 实现 → 验证）；
- v1.6 移除 `Grill-me` 作为 Doctor 的交接对象——它不是本协议的一环；
- 原本"诊断需要被挑战"的价值，改为 **Doctor 自己出具诊断前必须完成的自我质疑**（见 SKILL.md §19.1），作为内部质量门禁保留，不外包给任何外部阶段。

**用户拿到这份问题定义之后去干什么，Doctor 不代指定。**

## v1.5 的变更回顾：Doctor 有两条路径

Doctor 不再只有"找病因"一种模式。收到问题后先分流：

| 路径 | 典型问法 | Doctor 做什么 | 产物 | DOCTOR_DONE 之后 |
|---|---|---|---|---|
| **GUIDANCE**（需求型） | 「AI + 安全有哪些方向？」「我想做个 X」 | **把问题问对**：收敛背景、目的、范围、产物、下一步 | Problem Definition | 正式回答（Answer Drift Check） |
| **DIAGNOSIS**（症状型） | 「这个 API 能改别人数据」「数据库好慢」 | **找病因**：假设 → 取证 → 鉴别 → 根因 | Confirmed Diagnosis | **协议终止**（无下游阶段） |

判定依据：用户描述的是**一个想要的结果**（需求型）还是**一个反常的现象**（症状型）。

### GUIDANCE 的关键规则

- **至少两轮有效引导（P0）**：除非初始消息已给全"背景 + 目的 + 范围 + 目标/预期产物"。
  每轮必须推进一个新维度；同维度重复不计入轮数。
- **不得为凑满两轮而追问（P0）**：用户已给全信息时直接结束，再问就是无效引导。
- **方向 ≠ 目标**：「我想了解技术总览」只是 Direction，必须继续问到 Goal。
- **禁止擅自改变用户任务（P0）**：调研中发现更深的问题，只能作为回答中的一句补充，
  不得把用户的问题改掉。
- **Answer Drift 防护**：正式回答期间维护 `confirmed_goal / confirmed_scope / expected_output / next_step`
  锚点，每个分支自检是否服务于目标。

> 字段边界：这四个字段只能描述**用户要什么**；一旦描述**怎么做**，即落入禁止字段（solution / code / architecture / remediation），必须删除。

## v1.4 的变更回顾：移除 Solver 阶段

原流程为 `Doctor → Solver → Grill-me`。其中 **Solver（方案 → 实现 → 验证）已正式移除**，不再作为 Doctor 工作流中的一环；v1.6 起 Grill-me 同样不再是 Doctor 的交接对象（见文首 v1.6 变更节）。

现在 Doctor 的完整范围只有它自己：

```text
用户问题
   ↓
Doctor      → 把问题问对 / 找病因  → Problem Definition / Confirmed Diagnosis
   ↓
DOCTOR_DONE（终态）
```

**核心原则：Doctor 只负责把问题定义清楚。中间和之后都不设置任何解决方案阶段。**

## v1.3 的变更回顾：从 Problem Formation 升级为 Problem Diagnosis

Doctor 不再只是"帮你把问题定义清楚"，而是：

> **收集症状、背景、证据和上下文，建立假设，排除不成立的假设，最终确定「出了什么问题、为什么会出现这个问题」。**

```text
Doctor → What is wrong?  Why is it wrong?  → Confirmed Diagnosis
Doctor → What do you really want?          → Problem Definition
```

三个必须回答：`出了什么问题？` `为什么会出现？` `我们凭什么这么判断？`
四个绝不回答：`应该怎么修？` `应该怎么实现？` `应该使用什么技术？` `应该怎么部署？`

## 与 v1.2 的差异

| 项 | v1.2（边界版） | v1.3（诊断版） |
|---|---|---|
| 核心定位 | Problem Formation | **Problem Diagnosis** |
| 最终产物 | Problem Definition | **Confirmed Diagnosis** |
| 禁止重点 | 提前给方案 | 提前给方案 **+ 未经证据确诊** |
| 状态机 | BACKGROUND_DISCOVERY → PROBLEM_FRAME_DISCOVERY → PROBLEM_DEFINITION_DISCOVERY | **OBSERVATION_EXTRACTION → HYPOTHESIS_GENERATION → EVIDENCE_COLLECTION → DIFFERENTIAL_DIAGNOSIS → ROOT_CAUSE_ANALYSIS** |
| 提问评分 | `ProblemFrameImpact × ProblemDefinitionImpact` | **新增 `HypothesisDiscrimination`（能否区分竞争假设）** |
| 技术分析 | 可做，但边界模糊 | **明确允许深入技术分析（病因），只禁止给出解决方案** |

## 三层诊断状态（P0）

```text
Observed Symptom ≠ Confirmed Diagnosis
```

| 层 | 定义 | confirmed |
|---|---|---|
| **Observation** | 用户明确描述的现象 | — |
| **Hypothesis** | AI 根据现象产生的假设 | false |
| **Diagnosis** | 经足够证据确认的结论 | true |

用户只说「API 可以修改别人数据」时，手上只有现象，还缺：身份状态 / 资源关系 / 攻击方式 / 服务端行为 / 授权机制。
**此时直接写「成因清楚了」就是违规。**

## DOCTOR_DONE 是硬边界

```text
DOCTOR_DONE 之后，Doctor 不得继续生成任何 Solution 内容。
```

最典型的越界（实际发生过）：

```text
[Doctor Mode · DOCTOR_DONE]
诊断到这里就结束了。
已产出 越权漏洞_修复建议栏.md        ← 逻辑冲突（既然 DONE 了就不该还在给方案）
```

Doctor 确诊后停在这里：`DOCTOR_DONE` 是**终态**，不交接给任何后续阶段——
不是 Doctor 自己继续回答，更不是 Doctor 自己转去产出方案。

## 状态机（v1.6 双路径）

```text
INIT → PREFLIGHT → PATH_CLASSIFICATION
     ↓
 ┌───┴────────────────────────┐
GUIDANCE                    DIAGNOSIS
CONTEXT_ROUND               BACKGROUND_ANALYSIS
 ↓                           ↓
GOAL_ROUND                  OBSERVATION_EXTRACTION
 ↓                           ↓
SCOPE_ROUND（必要时）        HYPOTHESIS_GENERATION
 ↓                           ↓
PROBLEM_DEFINITION_READY    EVIDENCE_COLLECTION
 ↓                           ↓
USER_CONFIRMATION           DIFFERENTIAL_DIAGNOSIS
 │                           ↓
 │                          ROOT_CAUSE_ANALYSIS
 │                           ↓
 │                          DIAGNOSIS_READY
 │                           ↓
 │                          USER_CONFIRMATION
 │                           ├── rejected → MODEL_UPDATE
 │                           │              → HYPOTHESIS_GENERATION
 └─────────────┬─────────────┘
               ↓
          DOCTOR_DONE（终态）
               ↓
 ┌─────────────┴─────────────┐
GUIDANCE                   DIAGNOSIS
正式回答                    协议终止
+ Answer Drift Check       （无后续节点）
 ↓
（每个分支自检：
 是否服务于 confirmed_goal）
```

**状态机中不存在 Solution 状态，也不存在任何方案验证阶段。** DOCTOR_DONE 是唯一终态。

## 安装位置

```text
~/.workbuddy/skills/doctor-mode/
```

Windows 下等价于 `%USERPROFILE%\.workbuddy\skills\doctor-mode\`。

会话状态运行时写入**当前工作区**的 `.workbuddy/doctor-mode/session.json`，不写入技能目录本身，因此技能目录可安全纳入版本管理。

## 目录结构

```text
doctor-mode/
├── SKILL.md                                 # 主指令（加载入口）
├── README.md                                # 本文件
├── references/
│   ├── guidance-and-convergence.md           # 引导与问题收敛：七维表 / 两轮有效性 / 方向vs目标 / Answer Drift（v1.5）
│   ├── diagnosis-model.md                    # Diagnosis Model schema / 写入规则 / 状态机
│   ├── hypothesis-and-evidence.md            # 三层状态 / 假设生成 / 鉴别诊断 / 推翻后重诊断
│   ├── diagnosis-boundary-check.md           # 六分类 A–F / Guard 自检 / 越权案例对照
│   ├── question-selection.md                 # QScore / HypothesisDiscrimination / 术语翻译
│   ├── diagnosis-readiness-and-exit.md       # 诊断就绪 / EXIT / DOCTOR_DONE 终态
│   └── test-cases.md                         # Acceptance Test + 引导/漂移用例 + 30 回归案例
└── assets/
    └── diagnosis_model_template.json         # 状态文件空模板
```

## Diagnosis Model 字段边界

**允许保存（需求侧，v1.5 新增）**：

```text
confirmed_goal    confirmed_scope    expected_output    next_step
```

**禁止保存（方案侧）**：

```text
solution    fix    implementation    code    architecture    remediation
```

判定规则：字段内容描述**用户要什么** → 允许；描述**这件事怎么做** → 禁止，删除。
跨轮次状态机一旦存了方案，下一轮必然泄漏；也会让"这到底是问题定义还是解决方案"变得无法判断。

## 验收硬指标

| 指标 | 目标 |
|---|---|
| Unnecessary Question Rate | < 20% |
| Diagnosis Accuracy Rate | > 85% |
| Question Efficiency | 2–5 questions / diagnosis |
| Wrong Diagnosis Rate | < 10% |
| **Solution Leakage Rate** | **0%** |
| **Premature Diagnosis Rate** | **0%** |
| **Invalid Guidance Rate**（v1.5 新增） | **0%** |
| **Goal Substitution Rate**（v1.5 新增） | **0%** |

后四项是硬指标，任何一次都算失败。

- `Invalid Guidance Rate`：把同维度重复提问计入"两轮引导"、或在用户已给全信息后为凑数继续追问的比例。
- `Goal Substitution Rate`：因发现更深/更有趣的问题而擅自改变用户任务的比例。

## 最终边界

```text
Doctor → 把问题问对：需求型 → Problem Definition
       → 找病因：  症状型 → Confirmed Diagnosis
       → DOCTOR_DONE（终态）
```

```text
用户问题
   ↓
Doctor（GUIDANCE 或 DIAGNOSIS）
   ↓
问题已问对 / 病因已确诊
   ↓
GUIDANCE  → 正式回答（Answer Drift Check）
DIAGNOSIS → 协议终止，交付 Confirmed Diagnosis
```

|  | ✅ Doctor 负责 | ❌ 超出 Doctor |
|---|---|---|
| 关心的是 | 这到底是什么问题、为什么会出现、用户真正要什么 | 这个问题该怎么解决 |
| 典型产出 | 问题定义、根因、判定依据、范围与预期产物 | 修复建议、方案、选型、代码、SQL、架构、实施步骤 |
| 提问方向 | 「你是谁」「为什么问」「要什么结果」「多大范围」 | 「用什么框架」「怎么改」「怎么部署」 |
| 完成标志 | 用户确认：对，我要解决的就是这个 | ——不在本协议内—— |

> **Doctor 的职责在"问题被定义清楚"的那一刻结束。**

本目录只实现 Doctor 这一段，**不含任何下游阶段**（无 Solver、无方案验证环节）。也不含 Multi-Agent 编排、自动搜索、自动工具调用。
