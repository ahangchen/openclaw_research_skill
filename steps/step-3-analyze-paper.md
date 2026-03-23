# Step 3: 使用GLM WebReader精读每篇论文

## 目标

为每篇筛选出的论文使用GLM WebReader进行深度分析。

## 核心原则

```
╔════════════════════════════════════════════════════════════╗
║  🎯 质量第一原则                                           ║
║                                                              ║
║  ✅ 每篇论文15-20分钟是正常的                              ║
║  ✅ 深度分析比快速完成更重要                                ║
║  ✅ 使用Subagent避免token限制                              ║
║  ✅ 每篇论文独立处理，互不干扰                             ║
║                                                              ║
║  ❌ 不要为了省时而创建精简版文档                           ║
║  ❌ 不要因为token不足而降低质量                            ║
╚════════════════════════════════════════════════════════════╝
```

## 执行流程

### 1. 读取筛选后的论文列表

```bash
PAPERS_FILE="/tmp/spatial_agi_papers_$(date +%Y-%m-%d).json"

# 提取论文信息
PAPERS=$(cat "$PAPERS_FILE" | jq -r '.papers[] | "\(.title)|\(.arxiv_url)|\(.pdf_url)"')
```

### 2. 为每篇论文启动Subagent

```bash
for PAPER_INFO in $PAPERS; do
  IFS='|' read -r TITLE ARXIV_URL PDF_URL <<< "$PAPER_INFO"
  
  # 生成paper_id（用于文件命名）
  PAPER_ID=$(echo "$TITLE" | sed 's/[^a-zA-Z0-9]/_/g')
  
  echo "📚 处理论文: $TITLE"
  
  # 启动Subagent
  sessions_spawn \
    --mode run \
    --runtime subagent \
    --task "精读论文: $TITLE
    
论文信息:
- 标题: $TITLE
- arXiv: $ARXIV_URL
- PDF: $PDF_URL
- Paper ID: $PAPER_ID

要求:
1. 使用web_fetch工具读取arXiv HTML页面
2. 询问3个核心问题（核心算法、与Spatial AGI关系、创新点/局限）
3. 创建详细markdown文档（至少500行）
4. 保存到 /home/cwh/coding/auto_blog/spatial_agi/papers/

3个核心问题:
Q1: 这篇文章的核心算法原理是什么？请详细描述：1) 核心思想和动机，2) 主要技术方法，3) 算法流程和关键步骤，4) 输入输出。
Q2: 这篇文章与通用空间智能（Spatial AGI）有什么关系？请分析：1) 如何理解和表示空间，2) 如何处理空间关系，3) 对Spatial AGI有什么启发，4) 可以应用到哪些Spatial AGI场景。
Q3: 基于前面的分析，这个方法的主要创新点和局限性是什么？与其他相关工作相比有什么优势和劣势？

输出格式:
- 返回文档路径
- 返回文档行数" \
    --timeout 1500 \
    --run-timeout 1500
done

echo "✅ 所有论文精读完成"
```

## GLM WebReader精读流程

### Phase 1: 读取arXiv页面

```bash
# 读取arXiv HTML页面
ARXIV_HTML=$(echo "$ARXIV_URL" | sed 's|/abs/|/html/|')

echo "📥 读取arXiv页面: $ARXIV_HTML"
# 使用web_fetch工具读取
# web_fetch会自动提取markdown格式的内容
```

### Phase 2: 分析论文内容

基于读取的内容，回答3个核心问题：

**Q1: 核心算法原理**
- 核心思想和动机
- 主要技术方法
- 算法流程和关键步骤
- 输入输出

**Q2: 与Spatial AGI的关系**
- 如何理解和表示空间
- 如何处理空间关系
- 对Spatial AGI有什么启发
- 可以应用到哪些Spatial AGI场景

**Q3: 创新点和局限性**
- 主要创新点
- 主要局限性
- 与其他相关工作的对比

### Phase 3: 创建详细文档

```bash
# 使用收集的信息创建markdown文档
# 至少500行，包含完整的分析
# 保存到: /home/cwh/coding/auto_blog/spatial_agi/papers/YYYY-MM-DD_XX_paper_title.md
```

## 文档模板

```markdown
# [论文标题]

**发表日期**: YYYY-MM-DD  
**arXiv链接**: https://arxiv.org/abs/xxxx  
**PDF链接**: https://arxiv.org/pdf/xxxx  
**HTML版本**: https://arxiv.org/html/xxxx  
**作者**: 作者列表

## 核心问题

### Q1: 核心算法原理

**问题**: 这篇文章的核心算法原理是什么？

**分析**:
[基于GLM WebReader分析]

1. **核心思想和动机**
   [详细描述]

2. **主要技术方法**
   [详细描述]

3. **算法流程和关键步骤**
   [详细描述]

4. **输入输出**
   [详细描述]

### Q2: 与Spatial AGI的关系

**问题**: 这篇文章与通用空间智能（Spatial AGI）有什么关系？

**分析**:
[基于GLM WebReader分析]

1. **如何理解和表示空间**
   [分析]

2. **如何处理空间关系**
   [分析]

3. **对Spatial AGI的启发**
   [分析]

4. **可以应用的Spatial AGI场景**
   [分析]

### Q3: 创新点和局限性

**问题**: 基于前面的分析，这个方法的主要创新点和局限性是什么？

**分析**:
[基于GLM WebReader分析]

1. **主要创新点**
   [列出]

2. **主要局限性**
   [列出]

3. **与其他相关工作的对比**
   [对比分析]

## 核心技术发现

- 发现1
- 发现2
- 发现3

## 与Spatial AGI的关系

### 直接贡献
[描述]

### 技术启发
[描述]

### 应用场景
[描述]

## 个人思考

### 最令人兴奋的发现
[思考]

### 潜在局限
[思考]

### 与昨日研究的关联
[思考]

## 关键数据

- 模型参数
- 数据集
- 性能指标

## 总结

### 核心发现总结
[总结]

### 对Spatial AGI的意义
[总结]

---

**文档创建时间**: YYYY-MM-DD
**分析方法**: GLM WebReader
```

## 质量要求

- ✅ 文档至少500行
- ✅ 包含完整的3个问题分析
- ✅ 包含与Spatial AGI的关系分析
- ✅ 包含个人思考和见解
- ✅ 包含关键数据

## GLM WebReader优势

| 维度 | GLM WebReader |
|------|---------------|
| 响应速度 | ⭐⭐⭐⭐⭐（<10秒） |
| 稳定性 | ⭐⭐⭐⭐⭐（高可用） |
| 理解能力 | ⭐⭐⭐⭐（GLM-5，128K上下文） |
| 无需代理 | ⭐⭐⭐⭐⭐ |
| 简单直接 | ⭐⭐⭐⭐⭐ |

## 注意事项

1. ✅ **必须使用Subagent**：避免主session的token限制
2. ✅ **每篇独立上下文**：互不干扰
3. ✅ **质量优先**：宁可多花时间，也要确保质量
4. ✅ **3个核心问题**：必须完整回答

## 预计时间

- **单篇论文**: 15-20分钟
- **5篇论文（串行）**: 75-100分钟
- **5篇论文（并行）**: 20-30分钟
