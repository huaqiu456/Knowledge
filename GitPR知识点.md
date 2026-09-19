# Git PR（Pull Request）知识点总结

> 核心原则：**是否需要 fork，取决于你有没有目标仓库的写权限。**

---

## 目录

1. [核心概念](#一核心概念)
2. [两种协作模式](#二两种协作模式)
3. [开源贡献完整流程（6 步详解）](#三开源贡献完整流程6-步详解)
4. [常见易错点](#四常见易错点)

---

## 一、核心概念

### 1.1 三种仓库角色

| 名称 | 位置 | 你有写权限吗 | Git 远程名 |
|------|------|:---:|-----------|
| 原作者仓库 | GitHub | ❌ | `upstream`（约定俗成） |
| 你的 fork | GitHub | ✅ | `origin`（默认） |
| 本地仓库 | 你电脑 | ✅ | 本地 |

### 1.2 关键术语

| 术语 | 含义 |
|------|------|
| **fork** | 在 GitHub 上把别人的仓库复制一份到自己账号下，从而获得写权限 |
| **远程（remote）** | 本地仓库记住的远程仓库地址，一个本地仓库可同时连接多个远程 |
| **origin** | 克隆时自动创建的远程名，指向你克隆的那个仓库 |
| **upstream** | 手动添加的远程名（约定俗成），指向原作者仓库 |
| **PR（Pull Request）** | 请求仓库维护者拉取（合并）你的代码改动 |
| **跨仓库 PR** | PR 的源分支和目标分支不在同一个仓库（如 fork → 原仓库） |

### 1.3 判断是否需要 fork

```
你对主仓库有写权限吗？
  ├── ✅ 有 → 团队内部模式：不需要 fork，直接建分支发 PR
  └── ❌ 没有 → 开源贡献模式：必须先 fork
```

---

## 二、两种协作模式

### 2.1 模式一：团队内部（无需 fork）

> 前提：你对主仓库有写权限（如团队成员、collaborator）

**仓库结构**：所有人共用同一个主仓库

```
主仓库 your-team/repo
├── main              ← 受保护，不能直接推
├── new-feature       ← 你的功能分支
└── fix-bug           ← 同事的分支
```

**流程（4 步）**：

```bash
# 1. 克隆主仓库（不是 fork）
git clone git@github.com:your-team/repo.git

# 2. 建分支开发并推送
git checkout -b new-feature
# ...写代码...
git push origin new-feature

# 3. 在平台上发 PR：new-feature → main，团队审查后 Merge

# 4. 同步本地
git checkout main
git pull origin main
```

### 2.2 模式二：贡献开源（必须 fork）

> 前提：你对主仓库没有写权限

**仓库结构**：原作者仓库 ← 你的 fork ← 你本地

```
原作者仓库 (upstream)    你的 fork (origin)       你本地
├── main                 ├── main                 ├── main
                         └── new-feature          └── new-feature
```

**流程（6 步）**：详见[第三章](#三开源贡献完整流程6-步详解)

### 2.3 对比速查表

| 对比维度 | 团队内部 | 贡献开源 |
|---------|:---:|:---:|
| 主仓库写权限 | ✅ | ❌ |
| 需要 fork | ❌ | ✅ |
| 分支推送到 | 主仓库 | 自己的 fork |
| PR 两端 | 同仓库 `feature → main` | 跨仓库 `fork → 原仓库 main` |
| 需要 upstream 远程 | ❌ | ✅ |
| 同步时更新 fork | 不涉及 | 需要 `git push origin main` |


## 三、开源贡献完整流程（6 步详解）

> 场景：你想给一个开源项目贡献代码，但没有该仓库的写权限。

### 前置：三个仓库的关系

```
① 原作者仓库 (upstream)      ② 你的 fork (origin)        ③ 你本地
github.com/原作者/repo       github.com/your-user/repo    你电脑上的文件夹
├── main                     ├── main                     ├── main
                             └── new-feature              └── new-feature
```

| 名称 | 位置 | 权限 | Git 远程名 |
|------|------|:---:|-----------|
| 原作者仓库 | GitHub | ❌ | `upstream` |
| 你的 fork | GitHub | ✅ | `origin` |
| 本地仓库 | 电脑 | ✅ | 本地 |

---

### 第 1 步：克隆你自己的 fork

```bash
git clone git@github.com:your-user/repo.git
```

- 把 **你的 fork** 下载到本地，自动命名为 `origin`
- 克隆 fork 而非原仓库，是因为你需要往自己的 fork 推代码
- 此时 `git remote -v` 只有 `origin`，还没有 `upstream`

---

### 第 2 步：添加上游远程

```bash
git remote add upstream git@github.com:原作者/repo.git
```

- **作用**：注册原作者仓库地址，命名为 `upstream`（约定俗成）
- **注意**：这步只注册地址，不下载任何代码
- 之后 `git remote -v` 会同时显示 `origin` 和 `upstream`

---

### 第 3 步：建分支开发并推送

```bash
git checkout -b new-feature      # 新建并切换到功能分支（-b = branch + checkout）
# ...写代码...
git push origin new-feature      # 推送到你的 fork（origin）
```

推送后，你的 fork 上多了 `new-feature` 分支。

---

### 第 4 步：发起跨仓库 PR

在 GitHub 网页上操作：

```
你的 fork (your-user/repo)         原作者仓库 (原作者/repo)
└── new-feature  ──── PR ───────►  main
   (compare / head)                (base / 目标)
```

| GitHub 字段 | 值 |
|------------|-----|
| base repository | 原作者/repo |
| base branch | main |
| head repository | your-user/repo |
| compare branch | new-feature |

> 跨仓库 PR：PR 的来源和目标分属不同仓库。GitHub 自动识别 fork 关系。

---

### 第 5 步：原作者审查并合并

- 维护者审查代码、留评论
- 你修改后继续 `git push origin new-feature`，PR 自动更新
- 通过后维护者点击 **Merge**，原作者仓库的 main 包含你的改动

> ⚠️ 此时你的 fork 的 main 和本地 main 都还是旧的！

---

### 第 6 步：同步本地并更新 fork

```bash
git checkout main              # 切回本地 main
git fetch upstream             # 从原作者仓库下载最新（不动工作分支）
git merge upstream/main        # 合并 upstream/main 到本地 main
git push origin main           # 推送到你的 fork，让 fork 也更新
```

各阶段状态变化：

| 阶段 | 本地 main | 本地 upstream/main | fork main |
|------|:--:|:--:|:--:|
| fetch 前 | 旧 | 旧 | 旧 |
| fetch 后 | 旧 | ✅ 最新 | 旧 |
| merge 后 | ✅ 最新 | ✅ 最新 | 旧 |
| push 后 | ✅ 最新 | ✅ 最新 | ✅ 最新 |

### 清理功能分支（可选）

```bash
git branch -d new-feature                  # 删除本地分支
git push origin --delete new-feature       # 删除 fork 上的分支
```

---

### 完整数据流图

```
                 ① 原作者/repo
                      main
                       │
          ┌────────────┤ (2) git remote add upstream
          │            │
          │            ▼
          │      ② your-user/repo (fork)
          │            main
          │            new-feature
          │            │
          │ (1) git clone
          │            │
          │            ▼
          │      ③ 你本地
          │            main
          │            new-feature
          │            │
          └────────────┤ (3) git push origin new-feature
                       │
                       ▼
              (4) 发起跨仓库 PR
              new-feature → 原作者 main
                       │
                       ▼
              (5) 原作者 Merge
                       │
                       ▼
              (6) 同步：
                  git fetch upstream
                  git merge upstream/main
                  git push origin main
                       │
                       ▼
              三个仓库全部一致
```

---

## 四、常见易错点

### 4.1 `origin` 和 `upstream` 只是名字

它们**不是 Git 关键字**，只是远程仓库的代号。克隆时默认叫 `origin`，手动加的叫 `upstream`，你可以起任意名字。

### 4.2 `git fetch` 不动工作分支

`fetch` 只把远程更新下载到"远程跟踪分支"（如 `upstream/main`），当前工作的 `main` 不会变。要真正合并必须再执行 `merge`。

### 4.3 为什么第 6 步既要拉 upstream 又要推 origin

fork 和原仓库是**两个独立的远程仓库**，原作者合并后只有原仓库更新了，你的 fork 不会自动同步：

```
upstream/main（新） → fetch → 本地 main（新） → push → origin/main（新）
```

### 4.4 为什么第 6 步不合并 new-feature

PR 已在原仓库合并，改动已通过 `upstream/main` 进入本地 main。new-feature 和 main 内容一致，再 merge 会提示 "Already up to date"。

### 4.5 fetch vs pull 的区别

| 命令 | 等价于 | 是否动工作分支 |
|------|--------|:---:|
| `git fetch` | 只下载 | ❌ 不动 |
| `git pull` | `git fetch` + `git merge` | ✅ 直接合并 |

> 建议新手用 `fetch` + `merge` 两步走，更可控；熟练后可用 `pull` 一步到位。

---

## 一句话总结

> **fork** 是把别人的仓库复制到自己名下以获得写权限；**origin** 指向你的 fork，**upstream** 指向原作者仓库。开发时推到自己 fork 的分支，发跨仓库 PR 给原作者；合并后从 upstream 拉最新合并到本地，再推回 origin 让 fork 保持一致。