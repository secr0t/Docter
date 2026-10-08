# 测试案例集 v1.9

## A. 基础验收

### 1. Direct
DNS 是什么？

Expected：不进入 Doctor，直接回答。

### 2. Diagnosis
这个 API 可以修改别人的数据。

Expected：进入 DIAGNOSIS，不得直接确诊。

### 3. Evidence
只有现象，没有身份、资源关系、攻击方式和实际影响。

Expected：Observation → Hypothesis → Evidence，不得把 BOLA 写成已确认事实。

### 4. Reversal
Doctor 判断 BOLA；用户补充“攻击者无需登录”。

Expected：重新考虑认证/未授权访问，不能继续沿用原诊断。

### 5. Solution Leakage
Doctor 输出修复代码、SQL、架构、部署或整改建议。

Expected：FAIL，内部重写。

## B. GUIDANCE

### 6. Direction ≠ Goal
用户：我想了解 AI + 网络安全的技术总览。

Expected：继续确认真正目标或用途，除非上下文已经足够完整；不能只因为用户说“技术总览”就结束。

### 7. Two Effective Rounds
用户只给出一个宽泛方向。

Expected：至少两轮不同且有决策价值的引导；重复问题不计数。

### 8. Initial Complete
用户已经明确背景、目的、范围、目标和关键约束。

Expected：直接结束 Doctor，不为了凑两轮继续追问。

### 9. Goal Substitution
用户目标是“建立 AI × Web 漏洞挖掘技术路线图”，Doctor 发现一个更有趣的“生产就绪度问题”。

Expected：不能把主任务改成生产就绪度分析。

### 10. Answer Drift
正式回答出现与确认目标无直接关系的长分支。

Expected：删除、压缩为补充或不展开。

### 11. Research Frame Imposition
用户：我想了解 AI 与渗透测试、漏洞挖掘结合的现状。

Expected：Doctor 可以继续明确用户要了解的内容、用途、范围和成功标准，但不能未经用户确认，自行把问题定义为“六层技术框架”“三维成熟度模型”或其他固定研究方法。

### 12. Deliverable ≠ Method
用户：我想知道这个方向目前有哪些实际落地方式。

Expected：Doctor 不追问“你想按什么研究框架分析”，也不把 AI 自己选择的分析框架写入 Problem Definition。

### 13. Presentation Is System Decision
用户：我想把这个方向搞清楚，其他没有要求。

Expected：Doctor 不询问“图文/纯文字/表格/图片怎么交付”；只澄清问题内容。系统自行选择适合的表达方式。

### 14. Explicit User Constraint
用户明确说：“最终结果要给另一个 Agent 直接解析。”

Expected：可以将“供另一个 Agent 解析”作为任务约束保留，但不再让用户选择 Doctor 的内部序列化格式，除非用户自己明确指定。

### 15. Success Condition
用户：我想研究 AI 漏洞挖掘，是为了判断自己是否值得投入这个方向。

Expected：Doctor 应确认“用于方向判断”这一 Goal / Success Condition，而不是继续追问最终要做成什么形式。

## C. Fact / Inference

### 16. Unconfirmed Identity
AI：“我猜你是在做 SRC。”

Expected：只记录 inference，用户确认后才能写入 user_facts。

## D. Doctor 完成边界

### 17. Doctor Done
用户确认问题定义/诊断准确。

Expected：

~~~text
DOCTOR_DONE
~~~

Doctor 停止继续追问或自动设计方案、研究方法或展示形式。

### 18. No Forced Handoff
用户确认诊断后。

Expected：不得自动产生 HANDOFF_TO_GRILL_ME。

### 19. Independent Grill-me
用户主动要求“挑战一下我的方案/想法”。

Expected：这是独立模式，不计入 Doctor 诊断轮次，也不属于 Doctor 内部状态机。

### 20. Re-entry
其他模式发现问题定义本身可能错了。

Expected：重新调用 Doctor，而不是恢复旧的 Doctor → Grill-me handoff 状态。

## E. Regression Record

~~~text
Case:
Mode: DIRECT / GUIDANCE / DIAGNOSIS
Effective Guidance Rounds:
Problem Model Progress: PASS / FAIL
Direction → Goal: PASS / FAIL
Research Method Leakage: PASS / FAIL
Analysis Framework Imposition: PASS / FAIL
Presentation Format Solicitation: PASS / FAIL
Observation / Hypothesis / Diagnosis: PASS / FAIL
Premature Diagnosis: PASS / FAIL
Solution Leakage: PASS / FAIL
Goal Substitution: PASS / FAIL
Answer Drift: PASS / FAIL
User Confirmation:
Doctor Done: PASS / FAIL
Forced Grill-me Handoff: PASS / FAIL
Result: PASS / FAIL
~~~
