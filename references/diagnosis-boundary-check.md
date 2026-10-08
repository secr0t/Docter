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

GUIDANCE 中的 Goal / Scope / Deliverable 同样属于 Problem Definition，允许；一旦开始描述“怎么做”，即越界。

## 2. 输出前 Guard

逐段检查：

```text
1. 这是问题描述、假设、诊断还是证据？
2. 这个结论是否有证据？
3. 是否把 inference 写成 user fact？
4. 是否在证据不足时确诊？
5. GUIDANCE 是否真的推进了 Goal，而不是只收集背景？
6. 是否因为发现更深的问题而替换用户原目标？
7. 是否出现 solution / implementation？
```

命中 Solution / Implementation / Remediation 时，内部删除并重写，不向用户暴露 Guard 过程。

## 3. 典型边界

### 允许

```text
“攻击者已认证，并通过修改资源标识实际修改了其他用户的数据。
当前最符合对象级授权缺失/BOLA，但还需要确认资源关系与服务端授权行为。”
```

### 禁止

```text
“Service 层增加 owner_id 判断。”
“SQL 增加 WHERE owner_id = ?。”
“返回 403/404。”
“增加自动化测试并上线前复测。”
```

## 4. Handoff

Doctor 确认后：

`DOCTOR_DONE → HANDOFF_TO_GRILL_ME`

handoff 是问题定义/诊断资料，不是方案。

如果 Grill-me 发现问题定义错误：

`GRILL_ME → DOCTOR`

重新诊断，而不是让 Doctor 为旧结论辩护。
