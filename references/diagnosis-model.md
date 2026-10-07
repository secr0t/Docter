# Diagnosis Model v1.3

## 1. Schema

```json
{
  "session_id": "",
  "doctor_status": "active",
  "mode": "DOCTOR",
  "state": "INIT",
  "turn": 0,

  "target": {
    "text": "",
    "source": "user"
  },

  "background": {
    "value": null,
    "source": null,
    "confirmed": false,
    "impact": null
  },

  "observations": [],

  "context": [],

  "user_facts": [],
  "ai_inferences": [],

  "hypotheses": [],

  "evidence": [],

  "differential_diagnosis": [],

  "problem": {
    "type": null,
    "description": null
  },

  "root_cause": {
    "description": null,
    "confirmed": false
  },

  "diagnosis": {
    "problem": null,
    "problem_type": null,
    "root_cause": null,
    "evidence": [],
    "reasoning_summary": null,
    "confidence": 0,
    "confirmed": false
  },

  "unknowns": {
    "blocking": [],
    "important": [],
    "optional": []
  },

  "question_history": [],

  "question_budget": {
    "max": 7,
    "used": 0,
    "expose_to_user": false
  },

  "confirmation": {
    "status": "pending",
    "revision_count": 0
  }
}
```

## 2. 字段说明

| 字段 | 说明 |
|---|---|
| `observations` | **只放用户明确描述的现象**，AI 的推测一律不进这里 |
| `hypotheses` | 候选假设集，带 `confidence` / `status` |
| `evidence` | 诊断依据。必须能追溯到 observation 或用户明确确认 |
| `differential_diagnosis` | 鉴别过程记录：哪个假设被哪条证据支持/否定 |
| `problem` / `root_cause` | 中间态，确认后汇入 `diagnosis` |
| `diagnosis.confirmed` | **只有用户确认后才为 true** |
| `doctor_status` | `active` → `done`（仅 confirmed 后置） |
| `question_budget.expose_to_user` | **false**，永不显示「第 X/7 问」 |

## 3. 禁止保存的字段

```text
solution    fix    implementation    code    architecture
remediation    tech_stack（除非属于诊断约束）    fix_plan    checklist
```

原因：跨轮次状态机一旦存了方案，下一轮必然泄漏；且会让 Grill-me 失去可供验证的纯粹对象。

## 4. 子结构

### Observation

```json
{
  "id": "obs_001",
  "content": "修改请求中的 userId 后可以修改其他用户数据",
  "source": "user",
  "quote": "改一下 userId 就能改别人的数据"
}
```

**`source` 只能是 `user`。** AI 总结的现象如果添加了自己的理解，应归为 hypothesis。

### User Fact

```json
{
  "id": "fact_001",
  "fact": "用户是该系统的开发负责人",
  "source": "user",
  "confidence": 1.0,
  "confirmed": true,
  "quote": "我是开发负责人"
}
```

### AI Inference

```json
{
  "id": "inf_001",
  "inference": "用户可能是 SRC 黑盒测试人员",
  "source": "ai",
  "confidence": 0.72,
  "confirmed": false,
  "status": "open",
  "basis": ["用户问『越权怎么修』，措辞接近漏洞报告场景"]
}
```

`status`: `open` / `confirmed` / `rejected`

- **confirmed** → 转入 `user_facts`
- **rejected** → 留在 `ai_inferences` 标 rejected，**不得转入 user_facts**

### Hypothesis

```json
{
  "id": "h1",
  "name": "对象级授权缺失",
  "confidence": 0.65,
  "status": "active",
  "supporting": ["obs_001"],
  "contradicting": []
}
```

### Evidence

```json
{
  "id": "ev_001",
  "content": "攻击者已完成身份认证",
  "source": "user",
  "supports": ["h1"],
  "contradicts": ["h2"]
}
```

## 5. Inference → Fact 的唯一路径

```text
AI 生成 Hypothesis → 写入 ai_inferences（confirmed=false）
   ↓
向用户展示并标注「这是我的推断，不是你的确认」
   ↓
用户明确确认
   ↓
confirmed = true, status = "confirmed" → 转入 user_facts
```

**绝对禁止**：`AI Inference → 自动写入 User Fact`

## 6. 状态机

```text
INIT → PREFLIGHT → TARGET_ANALYSIS → BACKGROUND_ANALYSIS
     → OBSERVATION_EXTRACTION → HYPOTHESIS_GENERATION
     → EVIDENCE_COLLECTION → DIFFERENTIAL_DIAGNOSIS
     → ROOT_CAUSE_ANALYSIS → DIAGNOSIS_READY
     → USER_CONFIRMATION
          ├── rejected  → MODEL_UPDATE → HYPOTHESIS_GENERATION
          └── confirmed → DOCTOR_DONE → GRILL_ME
               │
               GRILL_ME
                 ├── 诊断成立 → VALIDATED_DIAGNOSIS（终态）
                 └── 诊断不成立 → DIAGNOSIS_REJECTED → HYPOTHESIS_GENERATION
```

`OBSERVATION_EXTRACTION` 在部分文档中记作 `SYMPTOM_IDENTIFICATION`，同一状态。

**状态机中不存在 Solution 状态。**

## 7. 每轮更新

1. 读回 `session.json`
2. 提取新 Observation / 新 Evidence / 新 Fact / 新 Inference
3. 更新 `hypotheses`（支持 → confidence↑；否定 → confidence↓，过低 → rejected）
4. 更新 `unknowns`，`question_budget.used += 1`
5. 诊断就绪判定（见 `diagnosis-readiness-and-exit.md`）
6. **跑 Diagnosis Boundary Check**
7. 写回 `session.json`

## 8. 用户推翻诊断

```text
diagnosis.confirmed = false, confirmation.status = "rejected"
原假设 → status = "rejected"，写入 contradicting 证据
新证据 → observations
回到 HYPOTHESIS_GENERATION（重新假设，不是改结论）
```

已确认的 user_facts **全部保留**，只修正被否定的部分。**不重建 session。**
