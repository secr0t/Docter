# 测试案例集 v1.4

## A. Acceptance Tests（8 项，v1.4 验收）

### Test 1：简单问题

```text
输入：DNS 是什么？
```
**Expected**：Doctor 不介入，直接回答。

---

### Test 2：需要诊断的问题

```text
输入：这个 API 可以修改别人的数据。
```
**Expected**：必须进入 Doctor。

---

### Test 3：未经证据不得确诊

```text
用户只说：API 可以修改别人数据。
```
**Expected**：不得直接写「根因就是对象级授权缺失」。
必须走 `Observation → Hypothesis → 索要证据`。
缺失的证据维度：身份状态 / 资源关系 / 攻击方式 / 服务端行为 / 授权机制。

---

### Test 4：Hypothesis 可以存在

```text
允许：「可能是 BOLA。」
```
**Expected**：`confirmed = false`，且输出中必须标注为推断。

---

### Test 5：诊断确认后停止

```text
用户：对，诊断准确。
```
**Expected**：

```text
doctor_status      = done
diagnosis.confirmed = true
```

DOCTOR_DONE 后交接给 **Grill-me** 做对抗性验证（`DOCTOR_DONE → handoff() → GRILL_ME`）。
**不能继续输出解决方案**，不得产出修复建议文档，也不得转入任何以生成方案为目的的环节。

---

### Test 6：Solution Leakage

以下在 Doctor 阶段**全部判违规**：

```text
建议使用……    可以增加……    修改 SQL……
增加 AOP……    增加拦截器……  返回 403……
返回 404……    增加测试……    上线前复测……
```

---

### Test 7：用户推翻诊断

```text
Doctor：我判断这是对象级授权缺失。
用户：不对，攻击者根本不需要登录。
```
**Expected**：重新诊断，重新纳入 `认证绕过 / 匿名访问 / 未授权访问`。
**不得**继续沿用「已认证用户水平越权」，**不得**输出任何解决方案。

---

### Test 8：Fact / Inference

```text
AI：我猜你是在做 SRC。
```
**Expected**：不得自动写 `user_role = SRC`，必须等确认。
用户说「不是，我是开发」→ inference 标 `rejected`，user_facts 写「开发」。

---

## A2. Doctor ↔ Grill-me 闭环验收（v1.4 新增，4 项）

> 移除 Solver 后，Doctor 唯一的对外交接对象是 Grill-me。以下 4 例专门用于验证这条链路，
> 替代原先 `Doctor → Solver → Grill-me` 的验收方式。

### Case 1：正常诊断闭环

```text
用户提供现象
↓
Doctor 追问 + 鉴别诊断
↓
Doctor 形成 Confirmed Diagnosis
↓
Grill-me 挑战（证据是否充分 / 有无替代解释 / 根因是否成立）
↓
诊断成立 → Validated Diagnosis
```

**Expected**：全链路不出现任何方案性环节；Doctor 在 DOCTOR_DONE 停止。

---

### Case 2：证据不足（不得因无 Solver 而提前结束）

```text
用户提供现象
↓
Doctor 判断 CompetingHypothesesDiscriminated = false
↓
继续追问
```

**Expected**：不得因为"后面反正没人会来解决问题"就降低收敛标准或提前给结论。
`Premature Diagnosis Rate = 0%` 在两阶段架构下**依然适用**。

---

### Case 3：Grill-me 推翻诊断

```text
Doctor → Confirmed Diagnosis（例如：对象级授权缺失）
↓
Grill-me 发现关键证据不足，或存在未被考虑的替代解释
↓
DIAGNOSIS_REJECTED
↓
退回 Doctor → 重新假设 → 重新取证 → 新诊断
```

**Expected**：回流到 Doctor 重新诊断。
**Grill-me 不得**自己跨过 Doctor 直接改写诊断结论，**更不得**自己转去制定方案。

---

### Case 4：Doctor 试图给方案（越界判定）

Doctor 输出下列任一形式：

```text
建议增加……   建议修改……   建议部署……
Service 层加校验   返回 403   上线前复测   修复清单
```

**Expected**：判定为 **Doctor 职责越界**，该段删除后重新生成。
不得以"这是交接给下一阶段的内容"为由绕过判定——交接对象只允许是 Grill-me，且只传诊断。

---

## B. 越权案例：完整诊断脚本

```text
用户：doctor-mode 我发现这个 API 可以修改别人的数据，这个越权怎么修？

── 第一步：确认角色 ──
你现在是在：
A. 做安全测试 / SRC 漏洞挖掘
B. 开发自己的系统
C. 甲方安全排查
D. 其他

── 第二步：确认现象 ──
修改请求中的 userId / resourceId 后，是否可以实际修改另一个账号的数据？

── 第三步：确认身份 ──
攻击者是否已经正常登录？

── 第四步：确认对象关系 ──
两个账号是否属于同一权限等级？

── 第五步：确认实际影响 ──
数据是否已经实际发生修改？

── 第六步：建立诊断 ──
【问题】已认证用户可以修改其他用户数据。
【问题类型】水平越权 / BOLA。
【根因】服务端没有对当前登录主体与目标资源之间的对象级授权关系进行有效验证。
【依据】攻击者已认证；仅修改资源标识；可定位其他用户资源；服务端实际执行修改。

── 第七步：用户确认 ──
我的诊断是：这是一个对象级授权缺失导致的水平越权问题。
根因是服务端没有有效验证当前登录主体对目标资源的操作权限。
这个诊断是否准确？

用户：对。

→ DOCTOR_DONE，停止。
```

**停止后不再输出**：

```text
Service 怎么改 / SQL 怎么写 / AOP 怎么做
返回 403 还是 404 / 怎么测试 / 怎么治理全站
```

**尤其不得**产出 `越权漏洞_修复建议栏.md` 之类的文档。

---

## C. 诊断 vs 解决方案：快速判定表

| 输出 | 判定 |
|---|---|
| 「可能是对象级授权缺失」 | ✅ Hypothesis |
| 「根因是服务端未验证主体-资源归属」 | ✅ Diagnosis（需证据 + 确认） |
| 「依据是：已认证、仅改 id、服务端执行」 | ✅ Evidence |
| 「应该在 Service 层加 owner_id 判断」 | ❌ Solution |
| `UPDATE ... WHERE id=? AND owner_id=?` | ❌ Implementation |
| 「建议返回 404 防止枚举」 | ❌ Solution |
| 「建议增加自动化测试」 | ❌ Solution |
| 「DOCTOR_DONE 后产出修复建议文档」 | ❌ 硬边界违规 |

---

## D. 回归集（30 例）

### D1. Technology（10）

| # | Target | 模式 | 首个诊断性问题 |
|---|---|---|---|
| 1 | 我要做一个前端页面。 | DOCTOR | 主要用于产品展示、后台管理、数据大屏、给领导汇报，还是其他？ |
| 2 | 帮我写个爬虫。 | DOCTOR | 要抓哪类数据，抓下来做什么用？ |
| 3 | 数据库好慢，怎么办？ | DOCTOR | 是某一条查询一直慢，还是高峰期整体变慢？ |
| 4 | 我想上 Kubernetes。 | DOCTOR | 想解决部署效率、资源利用率，还是环境一致性？ |
| 5 | 帮我做个登录功能。 | DOCTOR_LITE | 给内部后台用，还是面向外部用户？ |
| 6 | 用 Vue 3 + TS 写 PC 端管理后台登录页，响应式，输出完整代码。 | DIRECT | —— |
| 7 | 我的代码有 bug。 | DOCTOR | 具体现象：报错、结果不对，还是偶发卡死？ |
| 8 | 帮我优化一下性能。 | DOCTOR | 现在卡在哪里，有可量化目标吗？ |
| 9 | Python 里怎么把列表去重？ | DIRECT | —— |
| 10 | 我要做个 AI 安全产品。 | DOCTOR | 技术研究、产品原型，还是商业化产品？ |

### D2. Business（5）

| # | Target | 模式 | 首个诊断性问题 |
|---|---|---|---|
| 11 | 我要建设 SOC。 | DOCTOR | 为了合规、建运营能力、解决告警没人分析，还是没想清楚？ |
| 12 | 我们想做数字化转型。 | DOCTOR | 推动力是降本、客户要求，还是管理层要求？ |
| 13 | 帮我做一份年度规划。 | DOCTOR | 这份规划给谁看、要达成什么决定？ |
| 14 | 想提升团队效率。 | DOCTOR | 最具体的痛点是返工多、沟通慢，还是排期不准？ |
| 15 | 我想融资。 | DOCTOR | 现在处于准备材料、已接触投资人，还是条款谈判？ |

### D3. Product（5）

| # | Target | 模式 | 首个诊断性问题 |
|---|---|---|---|
| 16 | 我要做个 App。 | DOCTOR | 解决用户在什么场景下的什么问题？ |
| 17 | 帮我设计个官网。 | DOCTOR | 核心目标是获客、品牌展示，还是招聘？ |
| 18 | 我们要加一个新功能。 | DOCTOR | 这个功能要解决的用户问题是什么？ |
| 19 | 帮我写个 PRD。 | DOCTOR | 读者是谁，他们要据此做什么决定？ |
| 20 | 产品留存低怎么办？ | DOCTOR | 流失集中在新手引导、首次价值，还是长期使用？ |

### D4. Professional（5）

| # | Target | 模式 | 首个诊断性问题 |
|---|---|---|---|
| 21 | UG 刻字固定轮廓铣为什么要把数量改为公差？ | DOCTOR | 你说的是控制刀路疏密的那个参数吗？想懂原理还是要参数建议？ |
| 22 | 收敛体可以补面吗？ | DOCTOR | 是 UG/NX 里的对象吗？是缺面要补回完整实体吗？ |
| 23 | 等保 2.0 三级怎么过？ | DOCTOR | 首次定级备案，还是已备案准备测评整改？ |
| 24 | 工控网怎么隔离？ | DOCTOR | 当前网络结构如何，驱动是合规还是实际风险？ |
| 25 | 这个 API 越权怎么修？ | DOCTOR | 你是以什么角色处理：安全测试 / 开发 / 甲方安全 / 学习？ |

### D5. Everyday / Direct（5，全部 DIRECT）

| # | Target | 说明 |
|---|---|---|
| 26 | DNS 是什么？ | 知识问答 |
| 27 | 东京天气？ | 事实查询 |
| 28 | 帮我把这段话翻译成英文。 | 任务明确 |
| 29 | 「直接回答吧。」 | 用户明确跳过 |
| 30 | 1GB 等于多少 MB？ | 事实查询 |

**D5 组出现任何提问 → Unnecessary Question Rate 直接失分。**

---

## E. 验收指标

| 指标 | 目标 |
|---|---|
| Unnecessary Question Rate | < 20% |
| Diagnosis Accuracy Rate | > 85% |
| Question Efficiency | 2–5 questions / diagnosis |
| Wrong Diagnosis Rate | < 10% |
| **Solution Leakage Rate** | **0%** |
| **Premature Diagnosis Rate（v1.3 新增）** | **0%** |

**Premature Diagnosis Rate**：只有现象就写「根因就是 X」的比例。
与 Solution Leakage 并列为两项硬指标——**任何一次都算失败**。

## F. 记录模板

```text
Case #:
Target:
判定模式: DIRECT / DOCTOR_LITE / DOCTOR / FORCED_DOCTOR
Observation / Hypothesis / Diagnosis 分层是否正确:
是否过早确诊: ✅否 / ❌是
是否泄漏 Solution: ✅无 / ❌有（指出内容）
鉴别诊断是否执行:
首个问题（实际 / 期望）:
问题鉴别力: 高 / 中 / 低
总问题数:
Confirmed Diagnosis:
用户是否确认:
DOCTOR_DONE 后是否正确停止:
交接对象是否为 Grill-me、且只传诊断: ✅是 / ❌否（指出内容）
Grill-me 验证结果: 通过 / 退回（退回原因）
结论: PASS / FAIL
```
