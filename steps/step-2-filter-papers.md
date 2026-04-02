# Step 2: 筛选最有价值的5篇论文

## 🚨 严格约束

```
╔══════════════════════════════════════════════════════════════════╗
║  ❌ 绝对不能违反的规则：                                       ║
║  1. 筛选结果必须恰好5篇，不能多也不能少                         ║
║  2. 必须排除已经分析过的论文（查看 /home/cwh/coding/spatial_agi/papers/）║
║  3. 不能选择与昨天/前天重复的论文                               ║
║  4. 每篇论文编号必须唯一（01-05）                               ║
╚══════════════════════════════════════════════════════════════════╝
```

## 目标

从约100篇候选论文中筛选出**恰好5篇**与Spatial AGI最相关、最有价值的论文。

## 执行命令

```bash
cd ~/.openclaw/workspace/skills/spatial-agi-research/scripts

# 去重（排除已分析的论文）
python3 spatial_agi_filter_papers.py

# 输出去重后的论文列表到 /tmp/spatial_agi_papers_$(date +%Y-%m-%d).json
```

## 筛选标准

### 1. 相关性（0-100分）

**Spatial AGI核心相关（80-100分）**:
- 3D Gaussian Splatting
- Spatial Memory
- World Models
- Embodied AI
- Video Generation

**高度相关（60-80分）**:
- VLM 3D Understanding
- Spatial Reasoning
- Scene Understanding
- Robot Learning

**中等相关（40-60分）**:
- Computer Vision
- Deep Learning
- Neural Networks

### 2. 创新性（0-50分）

- **方法创新**: 新的算法、架构、表示方法
- **应用创新**: 新的应用场景、任务
- **性能提升**: 显著的性能改进

### 3. 影响力（0-30分）

- **机构**: 知名机构（MIT, Stanford, Google等）
- **作者**: 领域知名作者
- **引用**: 已有引用数（如果是旧论文）

### 4. 时效性（0-20分）

- **最近1周**: 20分
- **最近1月**: 15分
- **最近3月**: 10分
- **更早**: 5分

## 评分示例

```
论文: Matryoshka Gaussian Splatting
- 相关性: 95分（3D Gaussian Splatting核心相关）
- 创新性: 45分（俄罗斯套娃式表示，连续LoD）
- 影响力: 25分（剑桥大学）
- 时效性: 20分（2026-03-19）
总分: 185分
```

## 选择策略

1. **去重（最重要）**: 
   - 先列出 /home/cwh/coding/spatial_agi/papers/ 下所有已有论文
   - 排除所有已分析过的论文（按标题匹配）
   - 特别注意排除昨天和前天的论文
2. **排序**: 按总分降序排列
3. **多样性**: 确保覆盖不同主题（避免5篇都是同一主题）
4. **质量优先**: 宁可少选，不选低质量论文
5. **恰好5篇**: 最终输出必须恰好5篇，用序号01-05命名

## 输出

- **文件**: `/tmp/spatial_agi_papers_$(date +%Y-%m-%d).json`
- **内容**: 5篇精选论文的详细信息

## 手动筛选（可选）

如果自动筛选不够准确，可以手动调整：

```bash
# 查看候选论文
cat /tmp/today_papers.json | jq '.papers[] | {title, arxiv_url}'

# 手动编辑选择
vim /tmp/spatial_agi_papers_$(date +%Y-%m-%d).json
```

## 预计时间

- 10分钟（自动筛选）
- 5分钟（手动调整，如果需要）
