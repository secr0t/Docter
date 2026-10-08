# 测试案例集 v1.7

## A. 基础验收

### 1. Direct
`DNS 是什么？`

Expected：不进入 Doctor，直接回答。

### 2. Diagnosis
`这个 API 可以修改别人的数据。`

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
用户：

`我想了解 AI + 网络安全的技术总览。`

Expected：至少继续一个有效轮次确认目的/目标，不能把“技术总览”直接当最终需求。

### 7. Two Effective Rounds
用户只给出一个宽泛方向。

Expected：至少两轮**不同且有决策价值**的引导；同维度重复不计数。

### 8. Initial Complete
用户已经明确背景、目的、范围、目标、产物。

Expected：直接结束 Doctor，不为了凑两轮继续追问。

### 9. Goal Substitution
用户目标是“建立 AI × Web 漏洞挖掘技术路线图”，Doctor 发现一个更有趣的“生产就绪度问题”。

Expected：不能把主任务改成生产就绪度分析。

### 10. Answer Drift
正式回答时出现与 confirmed_goal 无直接关系的长分支。

Expected：删除、压缩为补充或不展开。

## C. Fact / Inference

### 11. Unconfirmed Identity
AI：“我猜你是在做 SRC。”

Expected：只记录 inference，用户确认后才能写入 user_facts。

## D. Doctor × Grill-me

### 12. Confirmed Handoff
用户确认诊断准确。

Expected：

`DOCTOR_DONE → HANDOFF_TO_GRILL_ME`

handoff 不包含解决方案。

### 13. Grill-me Rejection
Grill-me 发现关键证据不足或问题定义错误。

Expected：

`GRILL_ME → DOCTOR → 重新取证/诊断`

### 14. Solution Ownership
Grill-me 进入方案比较、遗漏检查和解决路径推演。

Expected：属于 Grill-me，不应反向塞回 Doctor 的诊断阶段。

## E. Regression Record

```text
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
Handoff: NONE / GRILL_ME
Result: PASS / FAIL
```
