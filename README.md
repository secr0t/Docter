# 🩺 Doctor

> **Doctor：把问题问对。**

Doctor 是一个面向 **问题发现、问题定义与根因诊断** 的 AI Skill。

它的核心不是马上给答案，而是通过对话不断减少问题定义中的不确定性，直到：

> **“我们到底在解决什么？”**

变得清楚、可确认。

---

## ✨ 核心理念

Doctor 不先设计答案。

它先建立一个不断收敛的 **Problem Model**：

| 维度 | 关注内容 |
| --- | --- |
| Intent | 为什么提出这个问题 |
| Goal | 真正想解决什么 |
| Scope | 问题边界 |
| Context | 影响问题理解的背景 |
| Constraints | 明确限制与前提 |
| Success Condition | 什么结果对当前问题才算有用 |

Doctor 每轮都做一件事：

> **找到当前最关键的不确定性，然后问一个最有价值的问题。**

---

## 🧭 两条路径

| 模式 | 目标 |
| --- | --- |
| 🧭 GUIDANCE | 从模糊表达中收敛真实目标 |
| 🩺 DIAGNOSIS | 从现象中确定问题、根因与证据 |

GUIDANCE 的核心过程：

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

DIAGNOSIS 的核心过程：

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

这里的“交付”指的是**语义内容**，不是某一种视觉或文本格式。

用户需要确认：

~~~text
我到底要解决什么？
范围是什么？
哪些背景和约束重要？
什么结果对我才真正有用？
~~~

用户不需要参与决定：

~~~text
图文还是纯文字
表格还是流程图
Markdown 还是 JSON
如何拆研究框架
采用什么分析方法
最终分几章
~~~

这些属于系统后续的执行与表达决策。

一句话概括：

> **用户确认“是什么”，系统决定“怎么表达、怎么研究、怎么回答”。**

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

不存在固定的 Doctor → Grill-me 流水线。

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

两者可以协作，但不存在隶属关系。

---

## 🧠 为什么需要 Doctor？

现实中的用户经常把**方向、现象、真实问题**混在一起。

例如：

> “我们的 SOC 告警很多，但是感觉效果不好。”

这句话可能隐藏多个不同问题：

~~~text
发现能力不足？
误报太高？
响应太慢？
无法证明覆盖能力？
还是其实业务目标本身没有定义？
~~~

Doctor 的作用不是抢先选一个答案，而是通过关键提问，逐步确认：

> **用户真正需要解决的到底是什么。**

---

## 🧩 一个重要的设计原则

Doctor 只负责把问题定义清楚。

它不会替用户决定：

~~~text
怎么研究
怎么分析
怎么拆框架
怎么展示
怎么解决
~~~

例如用户说：

> “我想了解 AI 与渗透测试、漏洞挖掘结合的现状。”

Doctor 应该帮助确认：

- 你究竟想知道哪些内容；
- 为什么需要知道；
- 范围到哪里；
- 什么结果对你真正有用。

但 Doctor 不应该未经确认就自己规定：

> “我们按六层框架研究，再用三维成熟度模型分析，最后分六章输出。”

这是**回答方法**，不是 Problem Definition。

同样，Doctor 不需要询问：

> “你最后希望图文、表格还是纯文字？”

呈现形式属于系统内部的表达决策。

---

## 📦 安装

将 Doctor Skill 放入 Agent 的 Skills 目录，并确保 Agent 能够正常加载。

或者直接告诉 Agent：

~~~text
帮助我安装skill：https://github.com/secr0t/Docter
~~~

---

## ✅ 完成标准

Doctor 完成后只需要满足：

~~~text
Problem Definition / Confirmed Diagnosis
        ↓
User Confirmation
        ↓
DOCTOR_DONE
~~~

到这里 Doctor 就完成了。

具体的研究方法、分析框架、呈现方式和下一步行动，由后续执行层或用户自行决定。

---

## 🧩 核心原则

> **Doctor 的终点不是“我已经问了很多问题”，而是“我现在已经知道你到底在问什么”。**

> **用户确认“是什么”，系统决定“怎么表达、怎么研究、怎么回答”。**
