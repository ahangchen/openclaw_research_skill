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

## 🏗️ 架构设计：Subagent串行执行

### 核心原则

```
╔════════════════════════════════════════════════════════════╗
║  🎯 所有Step必须使用Subagent执行                           ║
║                                                              ║
║  ✅ 主Session只负责调度（串行发起subagent）                ║
║  ✅ 每个Step独立subagent（互不干扰）                       ║
║  ✅ 避免token限制和上下文污染                              ║
║  ✅ 确保每个Step完整执行                                   ║
║                                                              ║
║  ❌ 禁止在主Session中直接执行任务                          ║
║  ❌ 禁止跳过任何Step                                       ║
╚════════════════════════════════════════════════════════════╝
```

### 执行架构

```
主Session (Cron/Main)
    │
    ├─→ Subagent[Step 0]: 完成度检查
    │       └─→ 返回结果 → 主Session记录
    │
    ├─→ Subagent[Step 1]: 搜索论文
    │       └─→ 返回论文列表 → 主Session记录
    │
    ├─→ Subagent[Step 2]: 筛选论文
    │       └─→ 返回5篇精选 → 主Session记录
    │
    ├─→ Subagent[Step 3.1]: 精读论文1
    ├─→ Subagent[Step 3.2]: 精读论文2  ← 可串行
    ├─→ Subagent[Step 3.3]: 精读论文3
    ├─→ Subagent[Step 3.4]: 精读论文4
    ├─→ Subagent[Step 3.5]: 精读论文5
    │       └─→ 返回5篇文档 → 主Session记录
    │
    ├─→ Subagent[Step 4]: 更新论文列表
    │       └─→ 返回状态 → 主Session记录
    │
    ├─→ Subagent[Step 5]: 生成思考文档
    │       └─→ 返回文档路径 → 主Session记录
    │
    ├─→ Subagent[Step 6]: 验证质量
    │       └─→ 返回验证结果 → 主Session记录
    │
    └─→ Subagent[Step 7]: Git推送
            └─→ 返回commit ID → 主Session记录
            
主Session生成完成报告
```

### 调度代码模板

```bash
# 主Session调度代码（伪代码）

# Step 0: 完成度检查
result_0=$(sessions_spawn --runtime subagent --task "执行Step 0: 完成度检查...")

# Step 1: 搜索论文
result_1=$(sessions_spawn --runtime subagent --task "执行Step 1: 搜索论文...")

# Step 2: 筛选论文
result_2=$(sessions_spawn --runtime subagent --task "执行Step 2: 筛选论文...")

# Step 3: 精读论文（串行）
for i in 1 2 3 4 5; do
    result_3_$i=$(sessions_spawn --runtime subagent --task "执行Step 3.$i: 精读论文...")
done

# Step 4: 更新列表
result_4=$(sessions_spawn --runtime subagent --task "执行Step 4: 更新列表...")

# Step 5: 生成思考
result_5=$(sessions_spawn --runtime subagent --task "执行Step 5: 生成思考...")

# Step 6: 验证质量
result_6=$(sessions_spawn --runtime subagent --task "执行Step 6: 验证质量...")

# Step 7: Git推送
result_7=$(sessions_spawn --runtime subagent --task "执行Step 7: Git推送...")

# 生成完成报告
generate_completion_report
```

### 为什么必须使用Subagent？

1. **Token限制**: 主Session有token限制，长任务容易超限
2. **上下文污染**: 主Session上下文会累积，影响后续步骤
3. **独立性**: 每个Step独立执行，失败可以单独重试
4. **可追溯**: 每个subagent有独立的session ID，便于调试
5. **完整性**: 确保每个Step完整执行，不会因为主Session结束而中断

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
Step 3: 精读论文 → 5个独立Subagent串行分析5篇论文
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

### ⚠️ 重要：所有Step必须使用Subagent

```
╔════════════════════════════════════════════════════════════╗
║  ❌ 错误做法：在主Session中直接执行Step                    ║
║  ✅ 正确做法：为每个Step启动独立Subagent                   ║
╚════════════════════════════════════════════════════════════╝
```

### 方法1: Cron自动执行（推荐）

由Cron任务自动触发，每天早上7点执行：

```bash
# Cron payload会自动调用所有subagent
# 详见: ~/.openclaw/cron/jobs.json

# ⚠️ 确保cron job配置正确，使用subagent执行每个step
```

### 方法2: 手动执行（使用Subagent）

**必须**为每个Step启动独立Subagent：

```bash
# Step 0: 完成度检查（启动Subagent）
sessions_spawn \
  --runtime subagent \
  --task "阅读 ~/.openclaw/workspace/skills/spatial-agi-research/steps/step-0-completion-check.md 并执行"

# Step 1: 搜索论文（启动Subagent）
sessions_spawn \
  --runtime subagent \
  --task "阅读 ~/.openclaw/workspace/skills/spatial-agi-research/steps/step-1-search-papers.md 并执行"

# Step 2: 筛选论文（启动Subagent）
sessions_spawn \
  --runtime subagent \
  --task "阅读 ~/.openclaw/workspace/skills/spatial-agi-research/steps/step-2-filter-papers.md 并执行"

# Step 3: 精读论文（5个Subagent串行）
for i in 1 2 3 4 5; do
  sessions_spawn \
    --runtime subagent \
    --task "阅读 ~/.openclaw/workspace/skills/spatial-agi-research/steps/step-3-analyze-paper.md 并精读论文$i"
done

# Step 4: 更新列表（启动Subagent）
sessions_spawn \
  --runtime subagent \
  --task "阅读 ~/.openclaw/workspace/skills/spatial-agi-research/steps/step-4-update-paper-list.md 并执行"

# Step 5: 生成思考（启动Subagent）
sessions_spawn \
  --runtime subagent \
  --task "阅读 ~/.openclaw/workspace/skills/spatial-agi-research/steps/step-5-generate-thinking.md 并执行"

# Step 6: 验证质量（启动Subagent）
sessions_spawn \
  --runtime subagent \
  --task "阅读 ~/.openclaw/workspace/skills/spatial-agi-research/steps/step-6-validate-thinking.md 并执行"

# Step 7: Git推送（启动Subagent）
sessions_spawn \
  --runtime subagent \
  --task "阅读 ~/.openclaw/workspace/skills/spatial-agi-research/steps/step-7-git-push.md 并执行"
```

### ❌ 错误示例

```bash
# ❌ 错误：直接在主Session中执行
cd ~/.openclaw/workspace/skills/spatial-agi-research
bash steps/step-7-git-push.md  # 这会在主Session中执行，容易出问题
```

### ✅ 正确示例

```bash
# ✅ 正确：启动Subagent执行
sessions_spawn \
  --runtime subagent \
  --task "执行Step 7: Git推送，阅读 steps/step-7-git-push.md 并执行所有步骤"
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

**串行执行说明**（所有Step）:
- 所有Subagent必须串行执行：等上一个Step完成后才启动下一个
- Step 3的5篇论文也必须串行：等论文1精读完成后再启动论文2
- 原因：避免超过5个活跃Subagent限制，确保每个Step完整执行

---

## 🚨 常见问题

### Q1: 为什么所有Step都必须使用Subagent？

**A**: 有5个关键原因：

1. **Token限制**: 主Session有token限制（通常100k-200k），长任务容易超限
   - Step 3论文分析需要大量token
   - Step 5思考生成需要大量token
   - 主Session会累积所有上下文

2. **上下文污染**: 主Session上下文会累积，影响后续步骤
   - 前面步骤的错误会污染后面步骤
   - 上下文过长会降低模型性能

3. **独立性**: 每个Step独立执行，失败可以单独重试
   - Step 7失败不影响Step 1-6的结果
   - 可以针对特定Step进行调试

4. **可追溯**: 每个subagent有独立的session ID
   - 可以查看每个Step的完整执行日志
   - 便于问题定位和调试

5. **完整性**: 确保每个Step完整执行
   - 不会因为主Session结束而中断
   - 每个Subagent有自己的生命周期

### Q2: 如何正确调用Subagent？

**A**: 使用`sessions_spawn`工具：

```bash
# ✅ 正确示例
sessions_spawn \
  --runtime subagent \
  --task "执行Step X: [任务描述]，阅读 steps/step-X-xxx.md 并执行"

# ❌ 错误示例：在主Session中直接执行
bash steps/step-7-git-push.md  # 不要这样做！
```

**参数说明**：
- `--runtime subagent`: 必须指定为subagent
- `--task`: 任务描述，包含step文档路径
- `--timeout`: 可选，超时时间（秒）

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
