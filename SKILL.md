# Spatial AGI Research Skill - v7.0（模块化版本）

## 概述

完整的Spatial AGI研究流程skill，通过模块化的subagent执行每日研究任务。

**核心特点**:
- ✅ 每天精读5篇论文（质量 > 数量）
- ✅ 使用GLM WebReader回答3个核心问题
- ✅ 生成详细的论文分析文档（至少500行）
- ✅ 每日思考文档（延续性研究）
- ✅ 强制质量验证（11项检查）
- ✅ 自动Git提交和推送

**版本**: v7.1（GLM WebReader正式版）
**最后更新**: 2026-03-23
**Git仓库**: git@github.com:ahangchen/openclaw_research_skill.git

---

## 🚨 强制要求

```
╔════════════════════════════════════════════════════════════╗
║  ⚠️ 执行前必须满足以下条件                                 ║
╚════════════════════════════════════════════════════════════╝

1. ✅ GitHub仓库已关联（main分支）
2. ✅ GLM-5可用（用于论文分析）
3. ✅ 目录已创建
```

---

## 📋 完整流程（7个Steps）

### 执行顺序

```
Step 0: 完成度检查 → 检查昨天任务是否完整
  ↓
Step 1: 搜索论文 → 使用arXiv API搜索100篇候选论文
  ↓
Step 2: 筛选论文 → 从100篇中精选5篇最有价值的论文
  ↓
Step 3: 精读论文 → 5个独立Subagent并行分析5篇论文
  ↓
Step 4: 更新列表 → 更新papers_list.md
  ↓
Step 5: 生成思考 → 基于完整论文分析生成每日思考
  ↓
Step 6: 验证质量 → 11项检查确保思考文档符合要求
  ↓
Step 7: Git推送 → 提交并推送到GitHub
```

---

## 🔧 如何执行

### 方法1: 自动执行（推荐）

由Cron任务自动触发，每天早上7点执行：

```bash
# Cron payload会自动调用所有subagent
# 详见: ~/.openclaw/cron/jobs.json
```

### 方法2: 手动执行

按顺序执行每个step的subagent：

```bash
# Step 0: 完成度检查
cd ~/.openclaw/workspace/skills/spatial-agi-research
# 阅读并执行 steps/step-0-completion-check.md

# Step 1: 搜索论文
# 阅读并执行 steps/step-1-search-papers.md

# Step 2: 筛选论文
# 阅读并执行 steps/step-2-filter-papers.md

# Step 3: 精读论文（使用Subagent）
# 阅读并执行 steps/step-3-analyze-paper.md

# Step 4: 更新列表
# 阅读并执行 steps/step-4-update-paper-list.md

# Step 5: 生成思考
# 阅读并执行 steps/step-5-generate-thinking.md

# Step 6: 验证质量
# 阅读并执行 steps/step-6-validate-thinking.md

# Step 7: Git推送
# 阅读并执行 steps/step-7-git-push.md
```

---

## 📁 文件结构

```
spatial-agi-research/
├── SKILL.md                      # 主文档（本文档）
├── EXECUTION_CHECKLIST.md       # 执行检查清单
├── QUICK_START.md               # 快速开始指南
├── steps/                       # 模块化步骤文档
│   ├── step-0-completion-check.md
│   ├── step-1-search-papers.md
│   ├── step-2-filter-papers.md
│   ├── step-3-analyze-paper.md
│   ├── step-4-update-paper-list.md
│   ├── step-5-generate-thinking.md
│   ├── step-6-validate-thinking.md
│   └── step-7-git-push.md
└── scripts/                     # 自动化脚本
    ├── spatial_agi_daily_robust.sh
    ├── spatial_agi_filter_papers.py
    ├── search_arxiv.py
    ├── check_spatial_agi_status.sh
    └── validate_thinking.sh
```

---

## 🎯 质量标准

### 论文文档

- ✅ 至少500行
- ✅ 包含完整的3个核心问题分析
- ✅ 包含与Spatial AGI的关系分析
- ✅ 包含个人思考和见解

### 每日思考

- ✅ 至少800行
- ✅ 包含11项必需内容（见Step 6）
- ✅ 通过质量验证脚本

---

## 📊 时间估算

| Step | 时间 | 备注 |
|------|------|------|
| Step 0: 完成度检查 | 1分钟 | 自动检查 |
| Step 1: 搜索论文 | 10分钟 | 自动执行 |
| Step 2: 筛选论文 | 10分钟 | 半自动 |
| Step 3: 精读论文 | 90分钟 | 5个Subagent串行 |
| Step 4: 更新列表 | 5分钟 | 手动 |
| Step 5: 生成思考 | 40分钟 | 需要深度思考 |
| Step 6: 验证质量 | 2分钟 | 自动验证 |
| Step 7: Git推送 | 2分钟 | 自动执行 |
| **总计** | **~3小时** | |

**并行优化**（Step 3）:
- 5篇论文并行处理: 20-30分钟
- 总时间: ~1.5小时

---

## 🚨 常见问题

### Q1: 为什么需要模块化？

**A**: 原SKILL.md文档过大（90KB），在执行时容易丢失细节。模块化后：
- ✅ 每个step有独立的详细文档
- ✅ 更容易维护和更新
- ✅ 减少主文档的token消耗
- ✅ 更清晰的执行流程

### Q2: Subagent如何调用？

**A**: 每个step文档包含详细的执行命令，Subagent会：
1. 读取对应的step文档
2. 按照文档中的步骤执行
3. 返回执行结果

### Q3: 如何确保质量？

**A**: 
1. ✅ Step 3强制使用Subagent（避免token限制）
2. ✅ Step 6强制验证（11项检查）
3. ✅ 最多3次重试机制
4. ✅ 失败则停止任务并报告错误

---

## 📚 参考文档

- **详细执行指南**: `steps/step-*.md`
- **执行检查清单**: `EXECUTION_CHECKLIST.md`
- **快速开始**: `QUICK_START.md`
- **脚本目录**: `scripts/`

---

## 🔗 相关链接

- **GitHub仓库**: https://github.com/ahangchen/openclaw_research_skill
- **博客仓库**: https://github.com/ahangchen/spatial_agi

---

**关键词**: `#spatial-agi` `#research-skill` `#modular` `#subagent` `#quality-validation`

**维护者**: OpenClaw AI
**版本历史**: v6.9 → v7.0（模块化重构）→ v7.1（GLM WebReader正式版）
