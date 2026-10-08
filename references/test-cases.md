# 测试案例集 v1.8

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

Expected：继续有效确认目的/目标，不能把“技术总览”直接当最终需求。

### 7. Two Effective Rounds
用户只给出一个宽泛方向。

Expected：至少两轮不同且有决策价值的引导；重复问题不计数。

### 8. Initial Complete
用户已经明确背景、目的、范围、目标、产物。

Expected：直接结束 Doctor，不为了凑两轮继续追问。

### 9. Goal Substitution
用户目标是“建立 AI × Web 漏洞挖掘技术路线图”，Doctor 发现一个更有趣的“生产就绪度问题”。

Expected：不能把主任务改成生产就绪度分析。

### 10. Answer Drift
正式回答出现与 confirmed_goal 无直接关系的长分支。

Expected：删除、压缩为补充或不展开。

## C. Fact / Inference

### 11. Unconfirmed Identity
AI：“我猜你是在做 SRC。”

Expected：只记录 inference，用户确认后才能写入 user_facts。

## D. Doctor 完成边界

### 12. Doctor Done
用户确认问题定义/诊断准确。

Expected：

~~~text
DOCTOR_DONE
~~~

Doctor 停止继续追问或自动设计方案。

### 13. No Forced Handoff
用户确认诊断后。

Expected：不得自动产生 HANDOFF_TO_GRILL_ME。

### 14. Independent Grill-me
用户主动要求“挑战一下我的方案/想法”。

Expected：这是独立模式，不计入 Doctor 诊断轮次，也不属于 Doctor 内部状态机。

### 15. Re-entry
其他模式发现问题定义本身可能错了。

Expected：重新调用 Doctor，而不是恢复旧的 Doctor → Grill-me handoff 状态。

## E. Regression Record

~~~text
Case:
Mode: DIRECT / GUIDANCE / DIAGNOSIS
Effective Guidance Rounds:
Direction → Goal: PASS / FAIL
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
