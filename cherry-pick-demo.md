# Git Cherry-Pick 完整演示教程

## 项目当前状态

### 分支结构图

```
    master(98e8e59) - Init Project
      ↗️
f194405 ← d917e03 ← 5a6d2da(feature/01) - add TestController
                    ↘️
                     0d8e2a5(feature/02) - add UserController
```

### 当前各分支内容

- **feature/01**: 包含 TestController.java
- **feature/02**: 包含 UserController.java
- **master**: 最新的主分支

## Cherry-Pick 实战演示

### 演示 1: 将 feature/02 的提交挑选到 feature/01

#### 🎯 目标

将 feature/02 分支中的 UserController.java 挑选到 feature/01 分支，这样 feature/01 就同时拥有两个控制器。

#### 📊 操作前的可视化图

```
feature/01分支:
d917e03 ← 5a6d2da ← HEAD
    |        |
    |        └── 添加了 TestController.java
    └── 基础提交

feature/02分支:
d917e03 ← 0d8e2a5
    |        |
    |        └── 添加了 UserController.java
    └── 基础提交
```

#### 🔧 操作命令序列

```bash
# 1. 确认当前状态
git status
git branch

# 2. 查看当前分支文件
ls -la src/main/java/org/study/gitdemo/web/
# 应该只看到 TestController.java

# 3. 查看提交历史
git log --oneline --graph --all -10

# 4. 查看 feature/02 分支的特定提交
git show 0d8e2a5 --name-only

# 5. 执行 cherry-pick
git cherry-pick 0d8e2a5

# 6. 验证结果
ls -la src/main/java/org/study/gitdemo/web/
# 现在应该同时看到 TestController.java 和 UserController.java

# 7. 查看新的提交历史
git log --oneline --graph -5
```

#### 📊 操作后的可视化图

```
feature/01分支 (操作后):
d917e03 ← 5a6d2da ← 新提交ID ← HEAD
    |        |         |
    |        |         └── UserController.java (从feature/02挑选)
    |        └── TestController.java
    └── 基础提交

feature/02分支 (不变):
d917e03 ← 0d8e2a5
    |        |
    |        └── UserController.java (原始提交)
    └── 基础提交
```

### 演示 2: 将提交挑选到新分支

#### 🎯 目标

创建一个新分支，并将两个功能分支的提交都挑选过来。

#### 🔧 操作命令序列

```bash
# 1. 从master创建新分支
git checkout master
git checkout -b feature/combined

# 2. 查看当前状态
git log --oneline -3

# 3. 挑选 feature/01 的提交
git cherry-pick 5a6d2da

# 4. 挑选 feature/02 的提交
git cherry-pick 0d8e2a5

# 5. 验证结果
ls -la src/main/java/org/study/gitdemo/web/
git log --oneline --graph -5

# 6. 查看完整的分支图
git log --oneline --graph --all -10
```

#### 📊 最终分支结构图

```
                     新提交2 ← HEAD (feature/combined)
                    ↗️         |
master(98e8e59) ← 新提交1      └── UserController.java
      ↗️             |
f194405 ← d917e03 ← 5a6d2da(feature/01) - TestController.java
                    ↘️
                     0d8e2a5(feature/02) - UserController.java
```

### 演示 3: 处理冲突情况

#### 🎯 目标

演示当 cherry-pick 遇到冲突时如何处理。

#### 🔧 创建冲突场景的命令

```bash
# 1. 在feature/01分支修改同一个文件
git checkout feature/01

# 2. 如果两个分支修改了相同文件的相同部分，会产生冲突
# 这时cherry-pick会停止并提示冲突

# 3. 冲突解决命令
git status                    # 查看冲突文件
git diff                      # 查看冲突详情
# 手动编辑冲突文件，解决冲突标记 <<<<< ===== >>>>>
git add <冲突文件>             # 标记冲突已解决
git cherry-pick --continue   # 继续cherry-pick

# 4. 如果想取消cherry-pick
git cherry-pick --abort
```

## 高级 Cherry-Pick 技巧

### 1. 批量挑选提交

```bash
# 挑选多个不连续的提交
git cherry-pick commit1 commit2 commit3

# 挑选连续的提交范围 (不包括commit1)
git cherry-pick commit1..commit3

# 挑选连续的提交范围 (包括commit1)
git cherry-pick commit1^..commit3
```

### 2. 不自动提交

```bash
# 只应用更改，不创建提交
git cherry-pick --no-commit 0d8e2a5
# 可以进行修改后再手动提交
git commit -m "自定义提交信息"
```

### 3. 修改提交信息

```bash
# 挑选时编辑提交信息
git cherry-pick --edit 0d8e2a5
```

### 4. 签名提交

```bash
# 在挑选的提交上添加签名
git cherry-pick --signoff 0d8e2a5
```

## 实验建议

### 🧪 实验步骤

1. **备份当前状态**: 先确保所有更改都已提交
2. **按顺序执行**: 按照演示 1 → 演示 2 → 演示 3 的顺序进行
3. **观察变化**: 每步都用 `git log --graph` 查看分支变化
4. **验证文件**: 每次操作后都检查文件系统的实际变化

### 🔍 关键观察点

- 新提交的哈希值与原提交不同
- 原分支保持不变
- 文件内容完全一致
- 提交历史的线性关系

### ⚠️ 注意事项

1. **测试环境**: 建议在测试分支上进行实验
2. **冲突准备**: 准备好处理可能的合并冲突
3. **回退方案**: 知道如何使用 `git reset` 回退操作
4. **提交依赖**: 确保挑选的提交不依赖其他未包含的更改

## Cherry-Pick 最佳实践

### ✅ 适用场景

- 将 bug 修复应用到多个分支
- 将特定功能移植到发布分支
- 从实验分支挑选成功的更改
- 重组提交历史

### ❌ 不推荐场景

- 大量提交的迁移（建议使用 merge 或 rebase）
- 包含复杂依赖关系的提交
- 频繁的跨分支同步（建议重新设计分支策略）

### 🎯 成功要点

1. **明确目标**: 知道为什么要挑选这个提交
2. **检查依赖**: 确保提交可以独立应用
3. **测试验证**: 挑选后要测试功能完整性
4. **文档记录**: 记录挑选的原因和影响
