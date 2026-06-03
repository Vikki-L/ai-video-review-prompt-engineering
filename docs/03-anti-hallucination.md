# 03 · Anti-Hallucination Strategies · 反 AI 幻觉的 4 大核心策略

> ⭐ 这是整套 pipeline 的**技术核心**。让 LLM 输出可信、可消费、可上线，不是靠"prompt 写得好"，而是靠**结构化约束**。

## 🎯 The Core Problem

LLM 的两个老毛病：

1. **输出格式漂移**：让它输出 JSON，它给你 markdown；让它输出 enum，它给你"basically OK"
2. **判断过度宽松**：缺少反例 → AI 倾向于"给个 pass 算了"

我用了 4 个策略系统性解决。

---

## 🛡️ Strategy 1 · JSON Schema enum 锁死输出

### 问题
让 AI 输出"决策"，它可能给你：
- ❌ `"basically pass"` （不是 enum 内的值）
- ❌ `"pass, but..."` （多余前后缀）
- ❌ `"通过"` （语言飘了）
- ❌ `"yes"` （概念错位）

### 方案
在 prompt 里**强制 schema + enum**：

```
Output strictly in JSON matching this schema:
{
  "decision": <one of: "reject" | "review" | "pass">,
  "reason": <string, max 200 chars>,
  "violations": <array of violation codes>
}

DO NOT include any text outside this JSON.
DO NOT add new fields.
DO NOT use any value for "decision" other than the 3 enum options.
```

→ AI 输出空间被压死，下游可以安全 `json.loads()` 后直接消费。

---

## 🛡️ Strategy 2 · 反例先于正例

### 问题
人类老师教学先讲"什么是对的"再讲"什么是错的"。但 LLM 反过来——**先看反例反而判断更严**。

只给正例的 prompt 容易让 AI 默认"给个 pass 算了"——因为它没有清晰的"不通过"参照标准。

### 方案
Prompt 结构按这个顺序：

```
1. ❌ NEGATIVE EXAMPLES (What to reject)
   - Example: <bad case 1 with brief reason>
   - Example: <bad case 2 with brief reason>
   - Example: <bad case 3 with brief reason>

2. ⚠️ EDGE CASES (When to escalate to human review)
   - Example: <ambiguous case 1>
   - Example: <ambiguous case 2>

3. ✅ POSITIVE EXAMPLES (What to pass)
   - Example: <good case 1>
   - Example: <good case 2>
```

→ AI 看完 reject 案例后再判断当前视频，倾向显著趋严。

---

## 🛡️ Strategy 3 · 二次自检（Self-Verification）

### 问题
LLM 第一遍输出有时会"想当然"——给个看似合理的答案，但回头自查会发现明显问题。

### 方案
在主 prompt 后追加一个"自检"步骤：

```
After producing your initial answer, perform a self-check:

1. Re-read your "decision" field.
2. Look at "violations" — did you include all violations 
   that actually apply?
3. If decision is "pass" but there are ANY violations listed,
   reconsider — should this be "review" instead?
4. If you find your initial answer needs adjustment, 
   output the CORRECTED answer.
```

→ 一些"假阳性 pass" 在自检阶段被纠正成 review。

---

## 🛡️ Strategy 4 · decision 字段（不是 rating）

### 问题
传统做法让 AI 输出"质量评分 1-10 分"——但**这是主观的**：
- 不同时段同一个 AI 的"7 分"标准不一样
- 下游不知道"7 分"应该 pass 还是 review

### 方案
把主观"评级"改造成**客观决策字段**：

```python
"decision": "reject" | "review" | "pass"
```

每个 decision 都对应**明确的下一步动作**：

| Decision | 下一步 |
|----------|--------|
| `reject` | 自动拒掉，通知 Creator 修改 |
| `review` | 进入 Coach 人工审核队列 |
| `pass` | 自动入审核通过 |

→ AI 输出直接被下游 pipeline 消费，**不需要再翻译**。

---

## 🔬 Why These 4 Work Together

| 策略 | 解决什么 |
|------|---------|
| ① Schema + enum | **输出格式**漂移 |
| ② 反例先于正例 | **判断过度宽松** |
| ③ 二次自检 | **想当然的假阳性** |
| ④ decision 字段 | **主观波动 + 下游难消费** |

四个加在一起，把 LLM 从"可能有用的 demo"变成"能稳定生产的 pipeline"——准确率 90%+ 就来自这套组合拳。

---

## 💡 Generalizable Takeaways

这套方法不只用于视频审核。任何需要 LLM 做**结构化判断**的场景都适用：

- 客服意图分类
- 工单优先级判定
- 内容合规审核
- 文档分类入库

核心思路：**用结构约束输入空间 + 用反例校准判断倾向 + 用自检捕获假阳性 + 用 enum 锁死输出**。

---

[← Back to README](../README.md)
