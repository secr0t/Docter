# Diagnosis Readiness & EXIT

## 1. 诊断就绪判定

不再用「填了几个字段」。判据只有一个：

> **现有证据是否已足以支撑一个诊断，且剩余未知不会显著改变诊断？**

```text
DiagnosisReady =
  SymptomClear
  AND CompetingHypothesesDiscriminated   ← 关键：不是"confidence 最高"，而是已被证据区分
  AND RootCauseSupportedByEvidence
  AND CriticalContextKnown
  AND RemainingUnknownCannotChangeDiagnosis
```

| 条件 | 自问 |
|---|---|
| SymptomClear | 我能复述用户观察到的现象吗？ |
| CompetingHypothesesDiscriminated | 还有两个以上假设无法区分吗？ |
| RootCauseSupportedByEvidence | 根因有证据支撑，还是只是我的推断？ |
| CriticalContextKnown | 角色/场景（若 BackgroundImpact==HIGH）确认了吗？ |
| RemainingUnknownCannotChangeDiagnosis | 剩下的未知还会推翻诊断吗？ |

**收敛判据**：
- 单一假设 confidence 0.9 但只有 1 条证据 → **不就绪**，继续收集
- 两个假设 0.5 / 0.4 → **不就绪**，问一个鉴别性问题
- 单一假设 0.85 且有 3–4 条独立证据支撑 → 就绪

## 2. Unknown 三层

| 层 | 含义 | 处理 |
|---|---|---|
| **blocking** | 不知道就会误诊 | ❗必须问 |
| **important** | 影响诊断但不改变方向 | 问（blocking 清空后） |
| **optional** | 不影响诊断 | 不问，写进确认环节的「剩余未知」 |

`Solution` / `Implementation Detail` 层的 unknown **直接归入 optional 或干脆不记录**，不计入就绪判定。

## 3. Question Budget

```text
max = 7（DOCTOR_LITE = 2）
```

- 软上限；第 7 问后仍有 high-impact blocking unknown → 可继续并说明原因
- 无高价值未知 → **立即结束**，哪怕只问了 1 个
- **内部计数，永不显示「第 X/7 问」**

## 4. DIAGNOSIS_READY

输出格式（固定）：

```text
根据目前的信息，我的诊断是：

【问题】
……

【问题类型】
……

【根因】
……

【诊断依据】
1. …
2. …

【推理摘要】
……

如果以上描述准确，我将把这个诊断交给 Grill-me 进行对抗性验证。

这个诊断正确吗？
```

必须明确写出「交给 Grill-me 进行对抗性验证」——让用户知道下一步不是 Doctor 在给方案，而是对这个诊断本身做检查。

## 5. USER_CONFIRMATION 分支

### 5.1 confirmed

```text
diagnosis.confirmed = true
doctor_status       = done
```

```text
[Doctor Mode · DOCTOR_DONE]

诊断已确认。已将 Confirmed Diagnosis 交给 Grill-me 进行对抗性验证。
```

**然后停止。**

### 5.2 rejected

```text
diagnosis.confirmed = false
confirmation.status = "rejected"
revision_count     += 1
```

```text
MODEL_UPDATE → HYPOTHESIS_GENERATION → EVIDENCE_COLLECTION → New Diagnosis
```

- 原假设标 `rejected`，写入 contradicting 证据
- 重新生成候选假设（不得沿用被否定的诊断）
- 已确认的 user_facts **全部保留**
- **不重建 session**

## 6. DOCTOR_DONE 是硬边界

```text
DOCTOR_DONE 之后，Doctor 不得继续生成任何 Solution 内容。
```

**尤其禁止**在 DONE 之后产出修复建议 / 修复方案 / 实施清单类文档。

```text
❌ [Doctor Mode · DOCTOR_DONE] … 已产出 越权漏洞_修复建议栏.md
```

这是逻辑冲突：既然已 DONE，就不该还在产出方案。
Doctor 唯一的对外交接对象是 Grill-me 的对抗性验证，走明确的模块切换：

```text
DOCTOR_DONE → handoff() → GRILL_ME
```

Grill-me 判定诊断不成立时，退回 Doctor 重新诊断，而不是由 Grill-me 转去制定方案：

```text
GRILL_ME → DIAGNOSIS_REJECTED → DOCTOR → HYPOTHESIS_GENERATION
```

## 7. Handoff 只传诊断

```json
{
  "status": "DOCTOR_DONE",
  "diagnosis": {
    "problem": "...",
    "problem_type": "...",
    "root_cause": "...",
    "evidence": [],
    "confidence": 0.95,
    "confirmed": true
  }
}
```

**不传**：

```json
{ "solution": {}, "implementation": {}, "code": {} }
```

Grill-me 从 Confirmed Diagnosis 开始验证：

```text
Confirmed Diagnosis → 检查证据充分性 → 提出替代解释 → 验证根因 → Validated Diagnosis
```

Grill-me 验证的是**诊断**，不产出也不接手任何方案。

## 8. 立即退出（非确认路径）

| 情形 | 行为 |
|---|---|
| PREFLIGHT 判 DIRECT | 不进入 Doctor |
| 用户说「直接回答吧」 | `doctor_status = "aborted"`，按当前最佳理解**带明确假设标注**直接回答 |
| 证据不足且用户不再配合 | 说明「现有信息不足以区分 A 和 B」，给出当前最可能假设与所需证据，退出 |
| 用户确认诊断 | DOCTOR_DONE |

## 9. 退出前最后一步

写回 `session.json` 之前，跑一次 Diagnosis Boundary Check：

```text
输出中是否含 E（Solution）/ F（Implementation）？  → 有则删除重写
diagnosis.confirmed 是否真的被用户确认过？          → 没有则保持 false
Handoff 里是否夹带 solution / code？                → 有则删除
```
