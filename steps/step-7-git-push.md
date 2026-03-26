# Step 7: Git提交和推送到GitHub（强化版）

## 目标

将今天的研究成果（论文文档、思考文档、论文列表）提交并推送到GitHub。

## 🚨 执行方式

```
╔════════════════════════════════════════════════════════════╗
║  ⚠️ 必须使用Subagent执行此Step                             ║
║                                                              ║
║  ❌ 错误：在主Session中直接执行git命令                      ║
║  ✅ 正确：启动Subagent执行完整的git流程                     ║
║                                                              ║
║  原因：                                                      ║
║  1. 避免主Session token限制                                ║
║  2. 确保push完整执行（不会被中断）                         ║
║  3. 独立session便于调试                                    ║
║  4. 可以完整记录git操作日志                                ║
╚════════════════════════════════════════════════════════════╝
```

## 调用方式

```bash
# 主Session应该这样调用：
sessions_spawn \
  --runtime subagent \
  --task "执行Step 7: Git推送

请阅读 ~/.openclaw/workspace/skills/spatial-agi-research/steps/step-7-git-push.md
并执行所有步骤，包括：
1. 检查git状态
2. 添加所有变更
3. 创建commit
4. 推送到GitHub
5. 验证push成功（git status必须显示'up to date'）
6. 返回commit ID和push状态

⚠️ 必须确保push成功，不能只commit不push！"
```

## 🚨 核心原则

```
╔════════════════════════════════════════════════════════════╗
║  ⚠️ 必须确保push成功，不能只commit不push                  ║
║                                                              ║
║  ❌ 常见错误：commit成功后就结束，忘记push                 ║
║  ✅ 正确做法：commit后必须验证push成功                     ║
║                                                              ║
║  🔍 强制验证：git status 必须显示 "up to date"             ║
╚════════════════════════════════════════════════════════════╝
```

## 执行步骤

### 1. 切换到博客目录

```bash
cd /home/cwh/coding/auto_blog/spatial_agi
```

### 2. 检查变更

```bash
git status
```

预期输出：
```
Changes not staged for commit:
  modified:   papers_list.md
  new file:   daily_thinking/YYYY-MM-DD.md
  new file:   papers/YYYY-MM-DD_01_xxx.md
  new file:   papers/YYYY-MM-DD_02_xxx.md
  new file:   papers/YYYY-MM-DD_03_xxx.md
  new file:   papers/YYYY-MM-DD_04_xxx.md
  new file:   papers/YYYY-MM-DD_05_xxx.md
```

### 3. 添加所有变更

```bash
git add .
```

### 4. 创建提交

```bash
DATE=$(date '+%Y-%m-%d')

git commit -m "feat: Spatial AGI Research - $DATE

- 分析5篇论文（arXiv最新）
- 生成论文深度分析文档
- 每日思考: ✅
- 更新论文列表

Spatial AGI Research Skill v7.0"
```

### 5. 推送到GitHub（必须执行）

```bash
# ⚠️ 必须推送到main分支，不是master分支
git push origin main

# 如果push失败，执行重试
if [ $? -ne 0 ]; then
    echo "❌ Push失败，尝试重试..."
    sleep 5
    
    # 重试机制（最多3次）
    RETRY=0
    while [ $RETRY -lt 3 ]; do
        echo "重试 $((RETRY+1))/3..."
        
        if git push origin main; then
            echo "✅ Push成功"
            break
        fi
        
        ((RETRY++))
        sleep 10
    done
    
    if [ $RETRY -eq 3 ]; then
        echo "❌ Push失败3次，停止任务"
        exit 1
    fi
fi
```

### 6. 🚨 强制验证push成功（必须执行）

```bash
# 检查分支状态
git status

# 预期输出必须包含：
# "Your branch is up to date with 'origin/main'"
# 或
# "nothing to commit, working tree clean"

# 如果显示：
# "Your branch is ahead of 'origin/main' by X commit"
# 说明push失败！必须重新push

# 自动检测并验证
STATUS=$(git status | grep "Your branch")

if echo "$STATUS" | grep -q "ahead of"; then
    echo "❌ 错误：本地commit未push到远程"
    echo "当前状态：$STATUS"
    echo "强制重新push..."
    git push origin main --force
    sleep 2
    # 再次验证
    STATUS=$(git status | grep "Your branch")
    if echo "$STATUS" | grep -q "ahead of"; then
        echo "❌ 仍然失败，停止任务"
        exit 1
    fi
fi

echo "✅ Push验证成功"
```

### 7. 最终确认

```bash
# 查看最新提交
git log --oneline -1

# 查看远程仓库状态
git remote -v

# 确认main分支
git branch -a | grep main
```

## 验证清单

```
╔════════════════════════════════════════════════════════════╗
║  ✅ 完成以下所有检查才算成功                               ║
╚════════════════════════════════════════════════════════════╝

□ git status 显示 "nothing to commit"
□ git status 显示 "up to date with 'origin/main'"
□ git log --oneline -1 显示最新commit
□ 没有显示 "Your branch is ahead of"
□ GitHub仓库可以访问：https://github.com/ahangchen/spatial_agi
```

## 常见错误及解决

### 错误1: Push被拒绝

```bash
# 拉取远程变更
git pull --rebase origin main

# 再次推送
git push origin main

# 验证
git status
```

### 错误2: 网络超时

```bash
# 检查网络连接
ping -c 3 github.com

# 重试机制（已包含在步骤5）
# 如果仍然失败，等待1分钟后重试
sleep 60
git push origin main
```

### 错误3: 认证失败

```bash
# 测试SSH连接
ssh -T git@github.com

# 如果失败，检查SSH密钥
ls -la ~/.ssh/id_rsa.pub

# 测试HTTPS连接
curl -I https://github.com
```

### 错误4: 本地有commit但未push

```bash
# 检查状态
git status

# 如果显示 "Your branch is ahead of 'origin/main'"
# 立即push
git push origin main

# 再次验证
git status
```

## 自动化脚本（可选）

创建自动化push脚本：

```bash
#!/bin/bash
# auto_push.sh

cd /home/cwh/coding/auto_blog/spatial_agi

# 检查是否有未提交的变更
if ! git diff-index --quiet HEAD --; then
    echo "发现未提交的变更，开始提交..."
    git add .
    DATE=$(date '+%Y-%m-%d')
    git commit -m "feat: Spatial AGI Research - $DATE"
fi

# 检查是否需要push
STATUS=$(git status | grep "Your branch")
if echo "$STATUS" | grep -q "ahead of"; then
    echo "发现未push的commit，开始推送..."
    git push origin main
    
    # 验证
    sleep 2
    STATUS=$(git status | grep "Your branch")
    if echo "$STATUS" | grep -q "ahead of"; then
        echo "❌ Push失败"
        exit 1
    else
        echo "✅ Push成功"
    fi
else
    echo "✅ 本地与远程同步"
fi
```

## 预期输出

### 成功示例

```
[main 9501eb8] feat: Spatial AGI Research - 2026-03-23
 8 files changed, 11512 insertions(+)
 create mode 100644 daily_thinking/2026-03-23.md
 create mode 100644 papers/2026-03-23_01_Under_One_Sun.md
 create mode 100644 papers/2026-03-23_02_Bridging_Semantic_Kinematic.md
 create mode 100644 papers/2026-03-23_03_EffectErase.md
 create mode 100644 papers/2026-03-23_04_Rethinking_Vector_Field.md
 create mode 100644 papers/2026-03-23_05_LVOmniBench.md
To github.com:ahangchen/spatial_agi.git
   743ca22..9501eb8  main -> main

✅ Push验证成功
On branch main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean
```

### 失败示例（需要纠正）

```
[main 9501eb8] feat: Spatial AGI Research - 2026-03-23
 8 files changed, 11512 insertions(+)

❌ 错误：本地commit未push到远程
当前状态：Your branch is ahead of 'origin/main' by 1 commit.
强制重新push...
To github.com:ahangchen/spatial_agi.git
   743ca22..9501eb8  main -> main

✅ Push验证成功
```

## 质量检查

在任务结束前，必须确认：

```bash
# 最终验证脚本
cd /home/cwh/coding/auto_blog/spatial_agi

# 检查1: 是否有未提交的变更
if ! git diff-index --quiet HEAD --; then
    echo "❌ 错误：有未提交的变更"
    exit 1
fi

# 检查2: 是否有未push的commit
if git status | grep -q "ahead of"; then
    echo "❌ 错误：有未push的commit"
    exit 1
fi

# 检查3: 远程仓库是否可访问
if ! git ls-remote origin main &>/dev/null; then
    echo "❌ 错误：无法访问远程仓库"
    exit 1
fi

echo "✅ 所有检查通过，任务完成"
```

## 预计时间

- 正常情况：1-2分钟
- 需要重试：3-5分钟
- 网络问题：5-10分钟

## 注意事项

1. ⚠️ **绝对不能只commit不push**
2. ⚠️ **必须验证git status显示"up to date"**
3. ⚠️ **如果push失败，必须重试直到成功**
4. ⚠️ **任务结束时必须再次确认远程仓库已更新**
