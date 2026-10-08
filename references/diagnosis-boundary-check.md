# Diagnosis Boundary Check

> **Doctor 可以深入思考，但思考对象是问题与病因，不是解决方案。**

## 1. 六分类

| 类别 | 内容 | Doctor |
|---|---|---|
| Observation | 用户描述的现象 | ✅ |
| Hypothesis | AI 的暂时判断 | ✅ |
| Diagnosis | 问题类型/诊断 | ✅ |
| Evidence | 支撑或否定诊断的证据 | ✅ |
| Solution | 怎么解决、技术选型 | ❌ |
| Implementation | 代码、SQL、配置、部署、实施步骤 | ❌ |

GUIDANCE 的 Goal / Scope / Deliverable 属于 Problem Definition，允许；一旦开始描述“怎么做”，即越界。

## 2. 输出前 Guard

检查：

~~~text
1. 这是问题描述、假设、诊断还是证据？
2. 结论是否有证据？
3. 是否把 inference 写成 user fact？
4. 是否在证据不足时确诊？
5. GUIDANCE 是否推进了 Goal，而不是只收集背景？
6. 是否因为发现更深的问题而替换用户原目标？
7. 是否出现 solution / implementation？
8. 问题已经清楚后，是否仍在 Doctor 中制造新的诊断任务？
~~~

命中 Solution / Implementation / Remediation 时，内部删除并重写。

## 3. 典型边界

允许：

~~~text
“攻击者已认证，并通过修改资源标识实际修改了其他用户的数据。
当前最符合对象级授权缺失/BOLA，但还需要确认资源关系与服务端授权行为。”
~~~

禁止：

~~~text
“Service 层增加 owner_id 判断。”
“SQL 增加 WHERE owner_id = ?。”
“返回 403/404。”
“增加自动化测试并上线前复测。”
~~~

## 4. Doctor 完成后的关系

~~~text
Doctor
  ↓
Problem Definition / Confirmed Diagnosis
  ↓
DOCTOR_DONE
~~~

之后是否使用 Grill-me、继续普通对话或执行其他任务，由用户或上层 Agent 决定。

**不要在 Doctor 内生成 HANDOFF_TO_GRILL_ME。**

如果其他模式后来发现问题定义有误，可以重新调用 Doctor；这不是 Doctor 内部状态机。
