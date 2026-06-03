# 04 · Badcase Feedback Loop · 让 AI 越用越准

## 🎯 The Problem

LLM 的一个根本特性：**它不会学习**。

不像 ML 模型有训练循环，LLM 一次部署后就停在那里——同样的错误案例你不主动告诉它，它会一直重复犯同样的错误。

这意味着：**没有 badcase 闭环的 AI 系统，准确率永远停在初始版本**。

## 🔄 The Loop · Badcase 反哺机制

我设计了一个 5 步闭环让错误案例进入下一轮 Prompt 优化：

```
                  ┌────────────────────────────┐
                  │  ① AI 给出审核结果           │
                  │  decision: pass / review / reject
                  └─────────────┬───────────────┘
                                │
                  ┌─────────────▼───────────────┐
                  │  ② Coach 人工复核             │
                  │  · 同意 AI 判断 → confirm    │
                  │  · 不同意 → override          │
                  └─────────────┬───────────────┘
                                │
                                ▼
                  ┌─────────────────────────────┐
                  │  ③ 标记 Badcase               │
                  │  AI = pass, Coach = reject    │
                  │  AI = reject, Coach = pass    │
                  │  → 自动进入 Badcase 库         │
                  └─────────────┬───────────────┘
                                │
                  ┌─────────────▼───────────────┐
                  │  ④ Badcase 分类聚类           │
                  │  · 按出错维度聚类              │
                  │  · 按视频类型聚类              │
                  │  · 找重复出现的错误模式        │
                  └─────────────┬───────────────┘
                                │
                  ┌─────────────▼───────────────┐
                  │  ⑤ Prompt 优化               │
                  │  · 把典型 badcase 加进反例    │
                  │  · 加强相关维度的判断标准      │
                  │  · 调整 enum 边界              │
                  │  · 发布新版 prompt             │
                  └─────────────┬───────────────┘
                                │
                                ▼
                  ┌──────────────────────────────┐
                  │  下一轮审核用新版 prompt        │
                  │  → 同类错误显著减少             │
                  └──────────────────────────────┘
```

## 🛠️ Key Design Decisions

### 1. **Override 必填 reason**
当 Coach 不同意 AI 的判断时，必须填写至少一段 reason 说明"AI 错在哪"——这段 reason 就是下次 prompt 优化的最佳素材。

### 2. **Badcase 分类聚类（而不是个案处理）**
单个 badcase 不直接进 prompt——会让 prompt 越来越长越来越乱。先**聚类**：
- 按出错维度聚类（如"AI 把 product lead 类判成了 skit 类"）
- 按视频类型聚类
- 同一类聚到一定规模后再统一加进反例

### 3. **Prompt 版本化 + 回归测试**
每次 prompt 更新前用历史 badcase 跑一遍 → 确保新版本能修复老问题，又不会引入新问题。

### 4. **不只看 false negative，也看 false positive**
- **false negative**（AI 说 pass，Coach 改 reject）= 漏判，最严重
- **false positive**（AI 说 reject，Coach 改 pass）= 误判，体验差

两类 badcase 都要纳入优化数据。

## 📊 Impact

这个闭环上线后：

- ✅ Prompt 从 v1.0 → 多个迭代版本，每一版的准确率都有提升
- ✅ 同类错误案例**复发率显著下降**
- ✅ 新 Coach 学习成本降低——直接看 badcase 库就能理解判断边界

## 💡 Generalizable Takeaway

**任何 LLM 生产系统都需要一个 badcase 闭环**——不是 nice-to-have，是必需。
- 没有闭环 = 部署即过时
- 有闭环 = 每一次错误都在让系统变得更好

---

[← Back to README](../README.md)
