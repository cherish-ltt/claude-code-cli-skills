---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans - creates isolated git worktrees with smart directory selection and safety verification
---

# Git多工作区管理规范

## 功能概览
通过共享同一仓库的隔离工作区，实现多分支并行开发无需切换分支

**核心原则**：系统化目录选择 + 安全校验 = 可靠隔离
**启动声明**："正在使用「Git多工作区管理」技能建立隔离环境"

## 三级目录选择机制

### 第一优先级：检查现有目录
```bash
ls -d .worktrees 2>/dev/null     # 首选（隐藏目录）
ls -d worktrees 2>/dev/null      # 备选方案
```
→ 两者都存在时优先选用`.worktrees`

### 第二优先级：检查CLAUDE.md配置
```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```
→ 发现配置则直接采用

### 第三优先级：用户决策
呈现标准选项：
```
未找到工作区目录，请选择创建位置：
1. .worktrees/（项目内隐藏目录）
2. ~/.config/superpowers/worktrees/<项目名>/（全局目录）
请选择编号：
```

## 安全校验流程

### 项目内目录强制校验
```bash
git check-ignore -q .worktrees || git check-ignore -q worktrees
```
**未忽略时处理流程**：
1. 追加到.gitignore文件
2. 提交变更
3. 继续创建工作区

⚠️ **关键意义**：防止工作区内容误入版本控制

### 全局目录豁免校验

## 五步创建工作流

### 步骤1：识别项目名称
```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 步骤2：创建工作区
```bash
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```
路径自动适配：
- 项目内路径：`$LOCATION/$BRANCH_NAME`
- 全局路径：`~/.../worktrees/$project/$BRANCH_NAME`

### 步骤3：自动环境初始化
根据项目文件触发对应操作：
```bash
# 各技术栈检测
[ -f package.json ] && npm install
[ -f Cargo.toml ] && cargo build
[ -f requirements.txt ] && pip install -r requirements.txt
[ -f pyproject.toml ] && poetry install
[ -f go.mod ] && go mod download
```

### 步骤4：基准测试验证
执行项目标准测试命令：
```bash
npm test | cargo test | pytest | go test ./...
```
→ 测试失败时中断并报错
→ 测试通过后输出就绪报告

### 步骤5：环境就绪通告
```
工作区就绪路径：<完整路径>
测试通过率：<N>项测试全部通过
可开始开发：<特性名称>
```

## 避坑指南

❌ **跳过忽略校验**
→ 导致工作区文件污染git状态
✅ 必须使用`git check-ignore`验证

❌ **预设目录位置**
→ 违反项目规范
✅ 遵循三级选择机制

❌ **忽视测试失败**
→ 无法区分新旧缺陷
✅ 必须获得明确继续授权

❌ **硬编码安装命令**
→ 破坏不同技术栈兼容性
✅ 根据项目文件自动检测

## 典型工作流示例
```
[检测到现有.worktrees目录]
[验证已加入.gitignore]
[创建feature/auth工作区]
[执行npm install]
[运行全部47项测试通过]

工作区就绪路径：/project/.worktrees/auth
测试通过率：47项测试全部通过
可开始开发：认证功能模块
```

## 红线规则

🚫 **严禁行为**：
- 未校验.gitignore直接创建项目内工作区
- 跳过基准测试验证
- 在测试失败时强行继续
- 存在歧义时擅自决定目录位置
- 忽略CLAUDE.md配置

✅ **必须遵守**：
- 严格执行目录选择优先级
- 项目内目录必须校验忽略状态
- 自动识别项目类型初始化
- 确保基准测试干净状态

## 技能联动

▸ **被以下场景调用**：
- 「设计方案评审」通过后的实施阶段
- 任何需要隔离环境的开发场景

▸ **配套技能**：
- 「开发分支收尾」→ 工作完成后必须调用清理
- 「计划执行」→ 实际开发在此工作区进行