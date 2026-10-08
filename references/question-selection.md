# Question Selection v1.5

## 1. 评分模型

```text
QuestionScore =
DiagnosticImpact                    ← DIAGNOSIS 主干
× (HypothesisDiscrimination         ← DIAGNOSIS 权重最高
   OR DimensionAdvance)             ← GUIDANCE 权重最高（v1.5 新增）
× Uncertainty
× Answerability
÷ InteractionCost
```

| 维度 | 含义 | 打分提示 |
|---|---|---|
| **DiagnosticImpact** | 答案是否影响最终诊断/问题定义 | 决定问题类型 = 1.0；只影响细节 = 0.1 |
| **HypothesisDiscrimination** | **能否区分多个竞争假设**（DIAGNOSIS） | 一问就能排除一半假设 = 1.0；对假设无区分力 = 0.1 |
| **DimensionAdvance** | **是否推进了一个新的需求维度**（GUIDANCE，v1.5） | 从背景推到目的 = 1.0；同维度补充 = 0.1 |
| **Uncertainty** | 当前对此有多不确定 | 完全没头绪 = 1.0；有倾向 = 0.6；已能推断 = 0.2 |
| **Answerability** | 用户是否容易回答 | 选择题 = 1.0；一句话事实 = 0.8；需先查证 = 0.4 |
| **InteractionCost** | 回答的认知成本 | 选一下 = 1；回忆 = 2；需判断权衡 = 3；需学概念 = 5 |

路径不同，主导因子不同：

```text
DIAGNOSIS → 看 HypothesisDiscrimination（能否排除竞争假设）
GUIDANCE  → 看 DimensionAdvance（是否推进 Context → Goal → Scope 的新维度）
```

## 2. HypothesisDiscrimination 是核心（v1.3 新增）

```text
✅「修改 userId 后，服务端返回成功但数据没变，还是数据确实被修改？」
   → 一次区分：权限问题 / 前端展示问题 / 业务逻辑问题     → 1.0

✅「攻击者是否需要正常登录？」
   → 区分：越权 / 未授权访问 / 认证绕过                   → 1.0

✅「两个账号是否同一权限等级？」
   → 区分：水平越权 / 垂直越权                            → 0.9

❌「你用 Java 还是 Go？」
   → 对诊断无区分力                                       → 0.1，不问
```

**当两个假设 confidence 接近时（如 0.5 / 0.4），必须问一个能区分二者的问题，而不是取高的那个直接确诊。**

## 2B. DimensionAdvance 是 GUIDANCE 的核心（v1.5 新增）

```text
✅「你准备把这个结果用于技术选型、自己实现，还是只是建立认知？」
   → 一次推进：产物形态 + 下一步                          → 1.0

✅「你为什么需要这个技术总览？」
   → 从 Direction 推到 Goal                                → 1.0

❌「你是做什么行业的？」（前一轮已确认是安全从业者）
   → 同维度重复，无效引导，不计入轮数                      → 0.1，不问
```

**有效引导计数规则**：只有推进了新维度才 +1。
连续两轮问同一维度 = **Invalid Guidance**，不得据此进入 Problem Definition。

## 3. 硬性规则

```text
Question → Does the answer materially change diagnosis?
                                                    No → 不问
```

这一条优先于评分。评分再高，只要不改诊断，就不问。

**不追求收集最多信息，追求用最少的问题确定病因。**

## 4. 层级

```text
Diagnosis（症状/假设/证据/根因）  >  Solution  >  Implementation Detail
✅ Doctor 只在这一层问                ❌ 默认不问
```

例外：只有当实现层信息构成**诊断约束**时才问（如「不能改动现有鉴权框架」会限制根因判断范围），且问法要转成约束视角：「有没有不能改动的部分？」

## 5. Background 询问条件

```text
BackgroundImpact == HIGH → 问
否则                     → 跳过，直接进 OBSERVATION_EXTRACTION
```

HIGH 判据：换一个角色，诊断方向会变。

```text
「越权怎么修？」   → HIGH（研究员要判别类型、开发要定位代码、甲方要评估风险）
「DNS 是什么？」   → LOW（且整体 DIRECT）
「数据库好慢。」   → MID（先问现象，不急问角色）
```

选项模板（按场景裁剪）：

```text
A. 做安全测试 / SRC 漏洞挖掘     B. 开发自己的系统
C. 甲方安全排查                  D. 学习 / 理解原理
E. 其他 / 不确定
```

**必须同时声明这是推断**：

```text
从你的描述看，我暂时猜你可能是在做安全测试——这是我的推断，不是你的确认。
```

## 6. 专业术语翻译

| 内部判断项 | 禁止问法 | 推荐问法 |
|---|---|---|
| BOLA / IDOR | 「这是 BOLA 还是 IDOR？」 | 「是登录后改一下请求里的 id，就能操作别人的数据吗？」 |
| 水平/垂直越权 | 「是水平还是垂直越权？」 | 「这两个账号的权限等级一样吗？」 |
| 认证 vs 授权 | 「这是认证问题还是授权问题？」 | 「操作之前需要正常登录吗？登录之后系统还会判断这条数据是不是你的吗？」 |
| SSR vs CSR | 「你选 SSR 还是 CSR？」 | 「这个页面需要靠搜索引擎拿到大量自然流量吗？」 |
| SOC 成熟度 | 「你的 SOC 目标 L 几？」 | 「现在是告警没人看，还是有人看但分析不出来？」 |
| 慢查询类型 | 「是 IO 还是 CPU 瓶颈？」 | 「是某一条查询一直慢，还是高峰期全部变慢？」 |

专业术语可留在 Diagnosis Model，但不得出现在问题文本中。
**例外**：术语本身是待确认的**问题分类**时，可用「我暂时判断这可能属于 X，对吗？」确认。

## 7. 选项与形式

```text
A. …
B. …
C. …
D. …
E. 其他 / 不确定 / 你推荐
```

- 一次只问 1 个问题
- 最多 4 个实选项 + 兜底项
- 互斥且覆盖主要可能
- 有推荐可标「（推荐）」，但不替用户决定
- 允许「不知道 / 你定」
- **输出不带「第 X/7 问」**（内部 `question_budget.expose_to_user: false`）

## 8. 用户说「不知道」

- 该证据记为 `unknowns.optional`
- 不就同一点追问第二次（除非换问法能显著降低回答成本）
- 依赖该证据的假设保持 `active` 但不提升 confidence
- 若现有信息不足以收敛 → 明确告诉用户「现有信息不足以区分 A 和 B」，并给出鉴别性问题，而不是硬选一个
