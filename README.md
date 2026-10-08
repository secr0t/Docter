# 🩺 Doctor

> **Doctor：把问题问对。**

Doctor 是一个面向 **问题发现、问题定义与根因诊断** 的 AI Skill。

它的核心不是马上给答案，而是通过对话不断减少问题定义中的不确定性，直到：

> **“我们到底在解决什么？”**

变得清楚、可确认。

---

## ✨ 核心理念

Doctor 持续建立一个不断收敛的 **Problem Model**：

| 维度 | 关注内容 |
| --- | --- |
| Intent | 为什么提出这个问题 |
| Goal | 真正想解决什么 |
| Scope | 问题边界 |
| Context | 影响问题理解的背景 |
| Constraints | 明确限制与前提 |
| Success Condition | 什么结果对当前问题才算有用 |

每一轮都做一件事：

> **找到当前最关键的不确定性，然后问一个最有价值的问题。**

---

## 🧭 两条路径

| 模式 | 目标 |
| --- | --- |
| 🧭 GUIDANCE | 从模糊表达中收敛真实目标 |
| 🩺 DIAGNOSIS | 从现象中确定问题、根因与证据 |

GUIDANCE：

~~~text
用户表达
 ↓
Problem Model
 ↓
关键不确定性
 ↓
高价值提问
 ↓
Problem Model 更新
 ↓
问题足够明确
 ↓
用户确认
 ↓
DOCTOR_DONE
~~~

DIAGNOSIS：

~~~text
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
用户确认
 ↓
DOCTOR_DONE
~~~

---

## 🧠 Doctor 真正交付的是什么？

Doctor 交付的是：

> **经用户确认的问题定义。**

这里的“交付”指的是**语义内容**。

用户需要确认：

~~~text
我到底要解决什么？
范围是什么？
哪些背景和约束重要？
什么结果对我才真正有用？
~~~

具体的内部表示、下游序列化方式和对话中的表达方式由系统决定。

因此，Doctor 不会把用户带进“选择图文/纯文字/表格/JSON”这样的内部表达决策；任务本身已有明确格式约束时，则按用户要求继承。

一句话概括：

> **用户确认“是什么”，系统决定“怎么表达”。**

---

## 🧠 Doctor 也不替用户设计研究过程

Problem Definition 回答的是：

~~~text
我要解决什么问题？
为什么解决？
范围到哪里？
哪些背景和约束重要？
什么结果才算有用？
~~~

而后续执行层负责决定：

~~~text
怎么研究
怎么分析
怎么组织资料
怎么验证
怎么表达
怎么解决
~~~

例如用户说：

> “我想了解 AI 与渗透测试、漏洞挖掘结合的现状。”

Doctor 应该帮助确认这个问题的真实目标、范围和成功标准。

Problem 确认完成后，再由后续回答阶段选择合适的分析框架和研究方式。

---

## 🔨 Doctor 与 Grill-me

**Grill-me 不是 Doctor 的下一阶段。**

| | 🩺 Doctor | 🔨 Grill-me |
| --- | --- | --- |
| 核心问题 | **我到底要解决什么问题？** | **我现在这个想法经不经得起推敲？** |
| 核心任务 | 问题发现、需求收敛、根因诊断 | 挑战假设、判断、方案与决策 |
| 何时使用 | 问题本身还不清楚 | 用户已经有明确的想法、判断或方案 |
| 输出 | Problem Definition / Diagnosis | Stress-tested Thinking |
| 是否强制衔接 | **否** | **否** |

~~~text
                用户
                  │
          ┌───────┴───────┐
          ↓               ↓
      🩺 Doctor        🔨 Grill-me
     把问题问对         把想法想透
          │               │
          ↓               ↓
   Problem Definition   Stress-tested
      / Diagnosis         Thinking
~~~

两者可以协作，但不存在固定流水线。

---

## 🧠 为什么需要 Doctor？

现实中的用户经常把**方向、现象、真实问题**混在一起。

例如：

> “我们的 SOC 告警很多，但是感觉效果不好。”

Doctor 首先识别真正需要澄清的变量：

~~~text
发现能力？
误报？
响应速度？
覆盖证明？
还是目标本身没有定义？
~~~

然后通过高价值问题逐步形成清晰的 Problem Definition。

Doctor 的价值在于：

> **让后续回答建立在正确的问题上。**

---

## 📦 安装

将 Doctor Skill 放入 Agent 的 Skills 目录，并确保 Agent 能够正常加载。

或者直接告诉 Agent：

~~~text
帮助我安装skill：https://github.com/secr0t/Docter
~~~

---

## ✅ 完成标准

Doctor 的生命周期：

~~~text
Problem Definition / Confirmed Diagnosis
        ↓
User Confirmation
        ↓
DOCTOR_DONE
~~~

之后由用户或上层过程决定下一步使用哪种能力。

---

## 🧩 核心原则

> **Doctor 的终点不是“我已经问了很多问题”，而是“我现在已经知道你到底在问什么”。**

> **用户确认“是什么”，系统决定“怎么表达、怎么研究、怎么回答”。**
