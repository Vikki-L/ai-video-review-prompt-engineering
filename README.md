# AI Video Review · Prompt Engineering System

> 基于 LLM 的多步视频审核 pipeline · 准确率 90%+ · 反 AI 幻觉策略 · Badcase 反哺机制 · 完整 Prompt 工程方法论

🎯 这是 [Creator Ops Portfolio](https://github.com/Vikki-L/creator-ops-portfolio) 的精选独立项目。

---

## TL;DR

**Problem**：一个北美 AI 教育出海产品的 Coach 团队每周要审核 2000+ 校园大使视频。人工审核耗时长、不同 Coach 标准不一致、新 Coach 上手周期长、错误案例无沉淀。

**Approach**：基于 LLM 的多步 AI 审核 pipeline，含「业务类别识别 → 模板匹配 → AI 多步审核 → 人工复核」完整链路，配套**反幻觉策略**（JSON Schema enum / 反例先于正例 / 二次自检 / decision 字段）+ Badcase 闭环。

**Result**：

- ✅ AI 自动判断准确率 _90%+_
- ✅ 作为 Coach 复核前置筛选，**统一标准**
- ✅ 新 Coach 上手周期显著缩短（看 AI 给出的判断 + 维度拆解就能学会）

---

## 🏗️ Repo Structure

```
ai-video-review-prompt-engineering/
├── README.md                          ← 你正在看
├── docs/
│   ├── 01-problem-and-context.md      ← 人工审核痛点 + 业务背景
│   ├── 02-system-design.md            ← 多步审核链路 + 视频类型 × 维度矩阵
│   ├── 03-anti-hallucination.md       ← ⭐ 反 AI 幻觉的 4 大核心策略
│   ├── 04-badcase-feedback-loop.md    ← Badcase 反哺机制 · 让 AI 越用越准
│   └── 05-results.md                  ← 量化效果 + 复盘
├── samples/
│   └── prompt-pattern-example.md      ← 抽象 prompt 模板示例（不是真实 prompt）
└── assets/                            ← (截图待添加)
```

> ⚠️ **真实 prompt 与业务规则未公开**：具体的视频类型分类、维度组成、prompt 文本、硬否决规则等因含商业敏感性暂不在本仓库公开。本仓库以**方法论 + 系统设计 + 反幻觉技术**为主，可在面试时单独 demo。

---

## 📐 System Highlights

### 完整审核链路（4 步）

```
业务类别识别 → 模板规则匹配 → AI 多步审核 → 人工复核
```

### 设计亮点（详见 docs/02 与 docs/03）

| 设计 | 解决什么问题 |
|------|------------|
| **按视频类型分类的 Prompt 体系** | 不同视频类型有完全不同的评判维度（无法用一个 prompt 包打天下）|
| **多维度审核**（含品牌露出 / 脚本结构 / 字幕 / 口播 / 模板命中 / 强弱营销识别 等）| 单维度评判会漏掉很多问题 |
| **一套客观硬否决规则**（如 AI 生成 / 时长过短 / 无人脸无产品 / 纯屏幕录制 等）| 把"主观判断"前置成"客观过滤" |
| **JSON Schema enum 锁死输出** | 杜绝 AI 输出非预期值（如 "maybe pass"、"basically ok" 等模糊词）|
| **反例先于正例** | 让 AI 先看到"什么是错的"，再看"什么是对的"，降低过度宽松倾向 |
| **二次自检** | 输出后让 AI 自己回头检查一遍 |
| **decision 字段**（reject / review / pass）| 主观"评级"改造成客观"决策"，便于下游自动化 |
| **Badcase 库反哺机制** | 错误案例沉淀回 Prompt 优化数据源，越用越准 |

---

## 💡 Key Insights from This Project

1. **Prompt Engineering 的核心不是"写好一句话"，而是"系统化构造一个能稳定输出的多步链路"**
2. **反幻觉的关键不是事后纠错，而是事前用 schema + 结构约束输入空间**
3. **把主观"评级"改成客观"decision 字段"**，让下游能直接消费 AI 输出，不需要再翻译
4. **Badcase 闭环 = 让 AI 越用越准**，否则 AI 永远在重复同样的错误

---

## 👤 About

**LIU Zibing (Vikki)** · zibingl2@illinois.edu

- 🎓 UIUC Grainger College of Engineering · 金融工程硕士
- 💼 AI 教育出海产品 · 产品助理（2025.01 – 至今）
- 🐙 [Main Portfolio](https://github.com/Vikki-L/creator-ops-portfolio)

---

*Last updated: 2026-06-03*
