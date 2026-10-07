# Hypothesis & Evidence

> **Observed Symptom ≠ Confirmed Diagnosis**

## 1. 三层状态

| 层 | 谁能产生 | `confirmed` | 输出时的措辞 |
|---|---|---|---|
| **Observation** | 只有用户 | — | 「你描述的现象是：…」 |
| **Hypothesis** | AI | `false` | 「我暂时判断这可能是…，这是我的推断」 |
| **Diagnosis** | AI + 证据 + 用户确认 | `true` | 「诊断结果：…，依据：…」 |

**升级路径只有一条**：

```text
Observation → Hypothesis →（索要并拿到证据）→ Diagnosis →（用户确认）→ Confirmed Diagnosis
```

任何一步跳级都算违规。

## 2. 为什么不能"一步确诊"

用户只说「这个 API 可以修改别人的数据」时，手上只有**现象**。
要确诊还缺这些证据维度：

```text
身份状态     攻击者是否已认证？
资源关系     目标资源是否属于另一个主体？
攻击方式     修改了哪个参数？
服务端行为   服务端是否接受并执行？
授权机制     服务端是否校验主体-资源归属？
```

缺身份状态就无法区分「越权」与「未授权访问」；
缺服务端行为就无法区分「权限问题」与「前端展示问题」。

## 3. Hypothesis 的数据结构

```json
{
  "hypotheses": [
    { "id": "h1", "name": "对象级授权缺失", "confidence": 0.65, "status": "active",
      "supporting": ["obs_001"], "contradicting": [] },
    { "id": "h2", "name": "认证绕过", "confidence": 0.20, "status": "active" },
    { "id": "h3", "name": "资源标识校验错误", "confidence": 0.15, "status": "active" }
  ]
}
```

`status`: `active` / `rejected` / `confirmed`

## 4. Differential Diagnosis（鉴别诊断）

```text
Evidence 进来
   ↓
逐条比对每个 active hypothesis
   ↓
支持 → confidence ↑（加入 supporting）
否定 → confidence ↓（加入 contradicting），低于阈值则 status = rejected
   ↓
某一假设 confidence 显著领先且证据充分 → DIAGNOSIS_READY
```

**判据**：不是"confidence 最高"就算完成，而是**证据充分**。
单一假设 confidence 0.9 但只有 1 条证据 → 继续收集。
两个假设 0.5 / 0.4 → 必须问一个能区分二者的问题，而不是取高的那个。

## 5. 高鉴别力问题示例

| 场景 | 问题 | 能区分 |
|---|---|---|
| 越权 | 「改 userId 后服务端返回成功，但数据没变，还是确实被修改了？」 | 权限问题 / 前端展示 / 业务逻辑 |
| 越权 | 「攻击者是否需要正常登录？」 | 越权 / 未授权访问 / 认证绕过 |
| 越权 | 「两个账号是否同一权限等级？」 | 水平越权 / 垂直越权 |
| 慢查询 | 「是单条查询慢，还是只有高峰期整体变慢？」 | 索引问题 / 资源竞争 / 容量问题 |
| 前端白屏 | 「是所有用户都白屏，还是只有特定浏览器？」 | 构建问题 / 兼容性问题 |

**低鉴别力（不问）**：Java 版本、框架选型、部署方式、团队规模——除非它们构成诊断约束。

## 6. 用户推翻诊断

```text
Doctor：「我判断这是对象级授权缺失。」
用户：「不对，实际上这个接口连登录都不需要。」
```

处理：

```text
1. diagnosis.confirmed = false, status = rejected
2. 推翻的证据写入 observations（新 Observation）
3. 原假设 h1 标 rejected（contradicting += 该证据）
4. HYPOTHESIS_GENERATION：重新纳入 认证绕过 / 匿名访问 / 未授权访问
5. EVIDENCE_COLLECTION：问新的鉴别性问题
6. 形成 New Diagnosis → 再次 USER_CONFIRMATION
```

**不得**：沿用「已认证用户水平越权」；不得跳过重新假设直接改结论；不得输出任何解决方案。

## 7. 用户说「不知道」

- 该证据点记为 `unknowns.optional`；
- 不得就同一点追问第二次（除非换一种问法能显著降低回答成本）；
- 依赖该证据的假设保持 `active` 但不提升 confidence；
- 若所有假设都因缺证据而无法收敛 → 直接向用户说明「现有信息不足以区分 A 和 B」，并给出鉴别性问题，而不是硬选一个。

## 8. 自检

```text
□ 我写的是 Observation、Hypothesis 还是 Diagnosis？标签对不对？
□ 这条结论背后有证据吗？证据是用户给的还是我自己推的？
□ 我有没有在只有现象时就写「成因清楚了」？
□ 我还没区分开的假设有哪些？下一个问题是冲着区分它们去的吗？
□ 用户推翻后，我是重新生成假设了，还是只是改了个结论？
```
