---
name: academic-workflow
description: >-
  学术研究全流程工作台。整合选题发现(research-direction-auto/discovery)、
  逆向文献插补(reverse-literature-interpolation)、文献综述(literature-review-v2)四大skill，
  支持单步执行和端到端流水线。从研究方向探索到综述产出一站完成。
argument-hint: "[模式] [参数]"
---

# 学术研究全流程工作台 Academic Workflow

整合四个核心学术 skill，覆盖从**研究方向探索 → 文献检索 → 综述写作**的完整 pipeline。

## 子 Skill 一览

| Skill | 功能 | 适用场景 |
|-------|------|---------|
| `research-direction-auto` | **全自动**选题发现：热点扫描→维度拆解→多库检索→缺口分析→选题 | 对领域已有了解，想快速获得选题建议 |
| `research-direction-discovery` | **交互式**选题发现：同上流程但每步需用户确认 | 刚进入新领域，需要逐步探索和反馈 |
| `reverse-literature-interpolation` | 逆向文献插补：根据研究框架/摘要/大纲反查所需文献 | 已有研究框架，需要针对性补全参考文献 |
| `literature-review-v2` | 文献综述生成：从论文文件夹自动提取+在线补全+撰写综述 | 已收集论文PDF，需要生成综述正文 |

## 操作模式

本 skill 支持 **4 种操作模式**：

```
1. 单步模式   → 直接调用某个子 skill，传入对应参数
2. 全自动流水线 → auto-pipeline: 一键选题→补文献→写综述
3. 交互式流水线 → interactive-pipeline: 分步确认的完整流程
4. 半程模式   → 从中间某个阶段开始（如已有选题后直接补文献+综述）
```

---

## 模式 1：单步模式

直接调用任一子 skill，与单独使用该 skill 效果相同。

```
academic-workflow 选题-auto [研究领域]
academic-workflow 选题-interactive [研究领域]
academic-workflow 插补 [研究框架/摘要/研究问题]
academic-workflow 综述 [论文文件夹路径]
```

### 模式选择指引

根据你的研究阶段选择：

```
你处于哪个阶段？
│
├── 还没有研究方向 / 想找选题
│   ├── 对领域熟悉、想快速出结果 → 选题-auto
│   └── 刚进入新领域、想逐步探索 → 选题-interactive
│
├── 已有研究框架/大纲，需要找文献支撑
│   └── 插补
│
├── 已收集好论文PDF，需要写综述
│   └── 综述
│
└── 想从选题到综述全流程 → 流水线模式（见模式2/3）
```

---

## 模式 2：全自动流水线 (auto-pipeline)

```
academic-workflow auto-pipeline [研究领域]
```

### 执行流程

```
输入: 研究领域/关键词
    │
    ▼
┌── Phase 1: 选题发现 ──────────────────────────────────┐
│  调用 research-direction-auto                          │
│  → 全自动完成热点扫描、维度拆解、多库检索、           │
│    缺口分析、选题生成                                  │
│  → 产出: research-direction-report.md                 │
└──────────────────────┬────────────────────────────────┘
                       │
                       ▼
┌── Phase 2: 文献逆向插补 ──────────────────────────────┐
│  以 Phase 1 产出的选题和研究框架为输入                 │
│  调用 reverse-literature-interpolation                 │
│  → 对每个研究维度检索并下载文献                        │
│  → 产出: literature-interpolation-result.md + PDFs     │
└──────────────────────┬────────────────────────────────┘
                       │
                       ▼
┌── Phase 3: 文献综述写作 ──────────────────────────────┐
│  以 Phase 2 下载的文献文件夹为输入                     │
│  调用 literature-review-v2                              │
│  → 自动提取、在线补全、撰写综述                        │
│  → 产出: literature-review-summary.md                  │
└──────────────────────┬────────────────────────────────┘
                       │
                       ▼
              完整研究报告: final-report.md
```

### 阶段产出传递

| 阶段 | 产出物 | 传给下阶段的内容 |
|------|--------|----------------|
| Phase 1 | `research-direction-report.md` | 选题列表、研究问题、各维度搜索词 |
| Phase 2 | `literature-interpolation-result.md` + `papers/` 文件夹 | 文献集合、PDF 文件路径 |
| Phase 3 | `literature-review-summary.md` | 最终综述正文 |

### 状态追踪

流水线会在工作目录生成 `.academic-workflow-state.json` 记录当前进度，中断后可恢复：

```json
{
  "mode": "auto-pipeline",
  "phase": "completed_1", 
  "research_area": "情感计算",
  "outputs": {
    "phase1_report": "research-direction-report.md",
    "phase2_report": "literature-interpolation-result.md",
    "phase3_report": "literature-review-summary.md"
  }
}
```

**恢复方法**: 再次执行同一命令，检测到状态文件后询问是否继续。

---

## 模式 3：交互式流水线 (interactive-pipeline)

```
academic-workflow interactive-pipeline [研究领域]
```

与全自动流水线的区别在于 Phase 1 使用 `research-direction-discovery`（需要用户每步确认），Phase 2-3 流程相同。

### 执行流程

```
输入: 研究领域/关键词
    │
    ▼
┌── Phase 1: 选题发现 (交互式) ────────────────────────┐
│  调用 research-direction-discovery                     │
│  → 展示热点简报、确认维度、确认文献清单、              │
│    确认分析报告、确认缺口优先级                        │
│  → 产出: research-direction-report.md                 │
└──────────────────────┬────────────────────────────────┘
                       │
                       ▼
┌── Phase 2: 文献逆向插补 ──────────────────────────────┐
│  [可选] 如果用户收集了更多论文，可传入补充路径          │
│  调用 reverse-literature-interpolation                 │
└──────────────────────┬────────────────────────────────┘
                       │
                       ▼
┌── Phase 3: 文献综述写作 ──────────────────────────────┐
│  调用 literature-review-v2                              │
└──────────────────────┬────────────────────────────────┘
                       │
                       ▼
              完整研究报告
```

### 交互节点

流水线会在以下节点暂停并等待用户确认：

1. **Phase 1 完成后**: 展示选题结果 → 确认是否继续插补和综述？
2. **Phase 2 开始前**: 确认搜索词和研究维度
3. **Phase 2 完成后**: 展示文献收集结果 → 确认是否进入综述？
4. **Phase 3 完成后**: 展示最终综述

---

## 模式 4：半程模式

从中间阶段开始，适用于已有部分成果的场景。

```
# 已有研究框架，直接插补+综述
academic-workflow pipeline-from-interpolation [研究框架]

# 已有论文文件夹，直接写综述
academic-workflow pipeline-from-review [论文文件夹路径]

# 已有选题报告，用选题结果做插补+综述
academic-workflow pipeline-from-report [选题报告路径]
```

### 从选题报告继续

如果已有 `research-direction-report.md`，可从中自动提取研究问题和维度，然后执行插补和综述：

```
academic-workflow pipeline-from-report research-direction-report.md
```

提取策略：

```
读取 research-direction-report.md
  → 解析"研究问题"和"选题建议"部分
  → 提取关键词、搜索词、研究维度
  → 作为 reverse-literature-interpolation 的输入
```

### 从研究框架继续

```
academic-workflow pipeline-from-interpolation "[研究框架描述]"
```

直接进入 `reverse-literature-interpolation` → Phase 2 → `literature-review-v2` → Phase 3。

---

## 实现指引

### 调用链

在执行流水线时，按以下方式调用子 skill：

```yaml
# Phase 1: 选题发现
Skill({"skill": "research-direction-auto", "args": "<研究领域>"})
# 或
Skill({"skill": "research-direction-discovery", "args": "<研究领域>"})

# Phase 2: 逆向文献插补
Skill({"skill": "reverse-literature-interpolation", "args": "<研究框架/选题>"})

# Phase 3: 文献综述
Skill({"skill": "literature-review-v2", "args": "<论文文件夹路径>"})
```

### 用户输入解析

`$ARGUMENTS` 的格式：

```
academic-workflow <mode> [parameters]
```

| 模式关键字 | 对应模式 | 参数 |
|-----------|---------|------|
| `auto-pipeline` | 模式2 | 研究领域/关键词 |
| `interactive-pipeline` | 模式3 | 研究领域/关键词 |
| `pipeline-from-interpolation` | 模式4 | 研究框架/摘要/研究问题 |
| `pipeline-from-review` | 模式4 | 论文文件夹路径 |
| `pipeline-from-report` | 模式4 | 选题报告路径 |
| `选题-auto` / `direction-auto` | 模式1 | 研究领域 |
| `选题-interactive` / `direction-discovery` | 模式1 | 研究领域 |
| `插补` / `interpolation` | 模式1 | 研究框架/摘要/研究问题 |
| `综述` / `review` | 模式1 | 论文文件夹路径 |

如果未指定模式，默认进入模式选择交互：
```
请选择操作模式：
1. 单步 — 直接运行某个子skill
2. 全自动流水线 — 从选题到综述一键跑通
3. 交互式流水线 — 分步确认，逐步推进
4. 半程模式 — 从中间阶段开始
```

### Phase 间数据传递

**Phase 1 → Phase 2**:
- 传递内容：研究问题列表、各维度的中英文搜索词、研究框架描述
- 传递方式：从 `research-direction-report.md` 中读取"研究问题"和"选题建议"部分
- 构造 Phase 2 的输入参数：将研究问题+选题描述拼接为研究框架文本

**Phase 2 → Phase 3**:
- 传递内容：下载的 PDF 文件路径列表
- 传递方式：从 `literature-interpolation-result.md` 中提取 `[PDF]` 标记的文件路径
- 构造 Phase 3 的输入参数：`papers/` 文件夹路径

### 失败处理

| 情况 | 处理 |
|------|------|
| Phase 1 产出为空或无合适选题 | 提示用户手动输入研究框架，跳至 Phase 2 |
| Phase 2 文献下载太少（< 5 篇） | 提示用户补充文献来源，仍继续 Phase 3 |
| Phase 2 未生成插补报告 | 跳过 Phase 2，直接进入 Phase 3 并提示文献不足 |
| Phase 3 论文文件夹不存在 | 提示用户指定路径或使用已下载的文献 |
| 状态文件损坏 | 忽略状态文件，从头开始 |

### 输出汇总

流水线结束后，生成最终汇总文件 `academic-workflow-report.md`，包含：

```markdown
# 学术研究工作流报告

**生成时间**: 2026-06-21
**工作模式**: auto-pipeline
**研究领域**: <输入>

## 一、流程概览

| 阶段 | 耗时 | 产出 |
|------|------|------|
| Phase 1: 选题发现 | 15分钟 | [选题报告](research-direction-report.md) |
| Phase 2: 文献插补 | 20分钟 | 下载 N 篇文献，[文献列表](literature-interpolation-result.md) |
| Phase 3: 综述写作 | 5分钟 | [文献综述](literature-review-summary.md) |

## 二、核心产出摘要

### 推荐选题 Top 3
1. <选题一>
2. <选题二>
3. <选题三>

### 文献统计
- 总检索文献: N 篇
- 成功下载: N 篇
- 综述引用: N 篇

### 综述摘要
<综述结论/研究述要>

---

## 文件清单
- [选题报告](research-direction-report.md)
- [文献插补结果](literature-interpolation-result.md)
- [文献综述](literature-review-summary.md)
- [全流程报告](academic-workflow-report.md)（本文件）
```

---

## 边界情况

| 情况 | 处理 |
|------|------|
| **输入为空** | 展示所有可用模式和示例，让用户选择 |
| **模式名拼写错误** | 尝试模糊匹配（如 "auto" → auto-pipeline，"全自动" → auto-pipeline） |
| **流水线中途中断** | 检测 `.academic-workflow-state.json`，询问是否继续 |
| **用户想跳过某个 Phase** | 在当前 Phase 开始前询问是否跳过，记录到状态文件 |
| **用户想替换某个 Phase 模式** | 如 auto-pipeline 中 Phase 1 想用交互版，手动调用后更新状态文件 |
| **文件夹路径不存在** | 提示路径错误，询问手动输入或使用对话中提到的路径 |

## 关联的子 Skills

- `research-direction-auto` — 全自动选题发现
- `research-direction-discovery` — 交互式选题发现
- `reverse-literature-interpolation` — 逆向文献插补
- `literature-review-v2` — 文献综述生成
- `gs-search` / `cnki-search` / `wos-search` — 底层检索 skill（被以上 skill 自动调用）
