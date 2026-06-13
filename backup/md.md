# GitHub 推送合并教程

## 目录

1. 引言：Git与GitHub的基本概念
2. 准备工作：安装Git、配置用户信息、创建SSH密钥
3. 本地仓库的创建与初始提交
4. 远程仓库关联与推送（Push）
5. 合并操作详解：Merge、Rebase与Fast-forward
6. Pull Request（PR）工作流：从创建到合并
7. 冲突的产生与解决方法
8. 高级话题：合并策略、Squash、Cherry-pick
9. 最佳实践与常见错误
10. 总结

---

## 1. 引言

Git是目前最流行的分布式版本控制系统，而GitHub是基于Git的代码托管平台。在团队协作中，`push`（推送）和`merge`（合并）是最核心的操作之一。推送将本地修改上传到远程仓库，合并则将不同分支的更改整合到一起。本教程将从零开始，系统性地讲解推送与合并的所有细节，涵盖基础操作、冲突处理、Pull Request工作流以及高级策略，帮助你成为一名Git与GitHub的高手。

---

## 2. 准备工作

### 2.1 安装Git

- **Windows**: 下载Git for Windows（https://git-scm.com/），安装时选择默认选项，建议勾选“Git Bash”。
- **macOS**: 通过Homebrew安装 `brew install git`，或安装Xcode Command Line Tools。
- **Linux**: 使用包管理器，如 `sudo apt install git`（Debian/Ubuntu）。

安装完成后，在终端运行 `git --version` 确认。

### 2.2 配置用户信息

每次提交都会记录作者信息，因此必须配置用户名和邮箱：

```
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

同时推荐配置默认分支名为 `main`（GitHub默认）：

```
git config --global init.defaultBranch main
```

### 2.3 创建SSH密钥（推荐）

使用SSH协议可以免密推送，推荐采用。

```
ssh-keygen -t ed25519 -C "your.email@example.com"
```

按提示选择保存路径，然后复制公钥：

```
cat ~/.ssh/id_ed25519.pub
```

登录GitHub，进入 Settings → SSH and GPG keys → New SSH key，粘贴并保存。测试连接：

```
ssh -T git@github.com
```

如果出现 “Hi username! You've successfully authenticated”，则配置成功。

---

## 3. 本地仓库创建与首次提交

### 3.1 创建新仓库

在本地创建一个项目目录并初始化Git仓库：

```
mkdir my-project
cd my-project
git init
```

此时在 `.git` 目录下生成版本数据库。

### 3.2 添加文件并提交

新建一个 `README.md` 文件，然后执行：

```
git add README.md
git commit -m "first commit"
```

也可以使用 `git add .` 添加所有文件。提交之后，本地历史中就有了一个初始快照。

### 3.3 查看状态与日志

- `git status` 显示工作区和暂存区的状态。
- `git log` 显示提交历史。

---

## 4. 推送（Push）到远程仓库

### 4.1 在GitHub上创建远程仓库

登录GitHub，点击右上角“+” → New repository，输入仓库名（建议与本地项目同名），选择公开或私有，不要勾选“Initialize with README”（因为我们已有）。创建后得到远程仓库URL，例如：

```
git@github.com:yourusername/my-project.git
```

### 4.2 关联远程仓库

将本地仓库与远程仓库关联：

```
git remote add origin git@github.com:yourusername/my-project.git
```

其中 `origin` 是远程仓库的默认别名，可以自定义。

### 4.3 首次推送

第一次推送时需要将本地分支与远程分支建立联系：

```
git push -u origin main
```

- `-u` 或 `--set-upstream` 建立上游跟踪，之后只需 `git push` 即可。
- 推送成功后，GitHub上的仓库就有了 `main` 分支。

### 4.4 常规推送

当本地有新的提交时，使用：

```
git push
```

如果本地分支与远程分支名称不同，可以指定：

```
git push origin local-branch-name:remote-branch-name
```

### 4.5 推送时可能出现的问题

- **非快进（non-fast-forward）错误**：远程分支有本地没有的提交时，推送会被拒绝。需要先拉取（pull）再推送。
- **权限问题**：检查SSH密钥或HTTPS凭据是否正确。
- **网络问题**：切换协议（SSH/HTTPS）或使用代理。

### 4.6 推送到不同分支

开发中通常有多个分支，例如 `feature-x`。创建并切换到新分支：

```
git checkout -b feature-x
```

做一些修改并提交，然后推送：

```
git push -u origin feature-x
```

这样远程就多了一个 `feature-x` 分支。

---

## 5. 合并（Merge）操作详解

### 5.1 什么是合并？

合并是将两个分支的修改整合到一起。最常见的场景是将功能分支合并入主分支（如 `main`）。Git提供多种合并方式：`merge`、`rebase`、`cherry-pick`，本节重点讲解 `merge`。

### 5.2 基本的 git merge

假设我们当前在 `main` 分支，想要合并 `feature-x`：

```
git checkout main
git pull origin main        # 先让本地main保持最新
git merge feature-x
```

执行 `merge` 后，Git会自动创建一个新的合并提交（merge commit），除非是快进合并（fast-forward）。

### 5.3 Fast-forward 合并

如果当前分支（`main`）没有新的提交，而目标分支（`feature-x`）只是领先于它，Git会直接移动指针，不产生新的合并提交。这种称为快进合并。

使用 `--no-ff` 可以强制创建一个合并提交，即使可以快进：

```
git merge --no-ff feature-x
```

这在团队协作中常用来保留分支历史。

### 5.4 合并分支时的冲突

如果两个分支修改了同一文件的同一部分，Git无法自动合并，就会产生冲突。冲突文件会被标记为冲突状态，需要手动解决。

### 5.5 使用 GitHub 网页界面合并 Pull Request

GitHub提供了方便的按钮来合并Pull Request（PR），一般有几种选项：
- **Create a merge commit**：与本地 `git merge --no-ff` 效果相同。
- **Squash and merge**：将PR中的所有提交压缩成一个提交后合并，适合保持主分支整洁。
- **Rebase and merge**：在目标分支上重新应用PR的每个提交（相当于本地 rebase），历史更线性。

---

## 6. Pull Request 工作流

Pull Request 是GitHub的核心协作机制。它允许开发者在合并分支之前进行代码审查和讨论。

### 6.1 创建 Pull Request

当你推送一个功能分支到远程后，可以在GitHub仓库页面点击 “Compare & pull request”。选择基础分支（如 `main`）和比较分支（如 `feature-x`），填写标题和描述，然后创建PR。

### 6.2 代码审查

团队成员可以查看PR中的每次提交，发表评论，甚至提出修改建议。当审查通过后，就可以合并。

### 6.3 合并 Pull Request

点击 “Merge pull request” 按钮，选择合适的合并方式（见5.5）。合并后，PR的状态变为 “Merged”。

### 6.4 更新本地仓库

合并后，本地 `main` 分支落后于远程。需要执行：

```
git checkout main
git pull origin main
```

删除已合并的分支（可选）：

```
git branch -d feature-x
git push origin --delete feature-x
```

---

## 7. 冲突的产生与解决

### 7.1 冲突场景

冲突通常发生在以下情况：
- 两个分支修改了同一个文件的同一行。
- 一个分支删除了文件，而另一个分支修改了它。
- 一个分支修改了文件的某个部分，另一个分支在相同区域有不同的修改。

### 7.2 解决冲突的步骤

1. 运行 `git merge` 或 `git pull` 后，Git会提示冲突文件。
2. 查看冲突文件：Git会用 `<<<<<<<`, `=======`, `>>>>>>>` 标记冲突区域。
3. 手动编辑文件，选择保留哪一方的更改，或者融合两者。
4. 删除冲突标记。
5. 保存文件。
6. 使用 `git add <file>` 将解决后的文件标记为已解决。
7. 执行 `git commit` 完成合并（不需要提供信息，Git会自动生成合并提交消息）。

### 7.3 使用图形化工具

推荐使用 `git mergetool` 调用外部工具（如 Beyond Compare、Meld、VS Code）。安装后运行：

```
git mergetool
```

图形界面可以更直观地比较左右两边的修改。

### 7.4 避免冲突的技巧

- 频繁同步：经常 `git pull` 保持分支最新。
- 小粒度提交：每个功能点单独提交，减小冲突范围。
- 良好沟通：与团队成员协调修改范围。

---

## 8. 高级话题

### 8.1 合并策略（Merge Strategies）

Git支持多种合并策略，最常用的是 `recursive`（默认）。通过 `-s` 参数指定：

```
git merge -s ours feature-x   # 完全忽略feature-x的更改，采用当前分支版本
git merge -s subtree feature-x # 适用于子目录合并
```

此外，`ort` 策略（默认）是 `recursive` 的重写版，性能更好。

### 8.2 Rebase 与交互式 rebase

`git rebase` 是将一个分支的提交“移植”到另一个分支的基点上。与 merge 不同，它不会产生合并提交，历史更加线性。

基本用法：

```
git checkout feature-x
git rebase main
```

交互式 rebase（`-i`）允许编辑、压缩、重排提交：

```
git rebase -i HEAD~3
```

注意：**不要对已经推送到远程的分支进行 rebase**，除非你强制推送（`git push --force`），但强制推送会破坏其他协作者的工作，应谨慎使用。

### 8.3 Squash 合并

将多个提交压缩成一个。在本地可以用：

```
git merge --squash feature-x
git commit -m "Add feature X"
```

在GitHub PR合并时选择 “Squash and merge” 效果相同。

### 8.4 Cherry-pick

选择某个分支上的一个或多个提交应用到当前分支：

```
git checkout main
git cherry-pick <commit-hash>
```

这在仅需要修改的一个小部分时非常有用。

### 8.5 标签（Tags）与推送

标签通常用于版本发布。创建标签：

```
git tag v1.0.0
```

推送标签到远程：

```
git push origin v1.0.0
```

或者推送所有标签：

```
git push --tags
```

---

## 9. 最佳实践与常见错误

### 9.1 最佳实践

- **保持提交粒度适中**：一个提交只做一件事，描述清晰。
- **经常推送**：避免本地长期不推送导致重大冲突。
- **在合并前先更新目标分支**：`git checkout main && git pull`。
- **使用分支保护规则**：在GitHub仓库设置中，可以要求PR必须通过审查、必须通过CI才能合并。
- **慎用强制推送**：除非在单人分支或者已确认他人无影响的情况下使用 `git push --force-with-lease`（比 `--force` 更安全）。
- **写好的合并提交消息**：明确说明合并了哪些功能。

### 9.2 常见错误及解决方案

| 错误 | 表现 | 解决方法 |
|------|------|----------|
| 非快进拒绝 | `! [rejected] non-fast-forward` | 先 `git pull --rebase` 再 `git push` |
| 冲突混乱 | 不知道如何解决 | 使用 `git status` 查看未合并文件，用 `mergetool` |
| 误提交到错误分支 | 提交在错误分支上 | 使用 `git cherry-pick` 移植，或 `git reset` |
| 想撤销合并 | 合并后发现有问题 | 使用 `git reset --hard ORIG_HEAD`（谨慎）|
| 分支名混淆 | 推送到错误分支 | 使用 `git push origin HEAD:correct-branch` |

### 9.3 工作流推荐

对于中小团队，推荐 **GitHub Flow**：
- `main` 分支始终可部署。
- 从 `main` 创建功能分支。
- 提交并推送。
- 创建Pull Request。
- 审查通过后合并到 `main`。

对于需要版本发布管理的项目，可以采用 **Git Flow**，包含 `develop`、`release` 等分支。

---

## 10. 总结

推送与合并是Git日常工作流中最频繁的操作。掌握它们不仅需要记住命令，更要理解背后的原理：引用、快进、三方合并、冲突标记等。本教程覆盖了从配置到高级技巧的各个方面，希望你能通过实践加深理解。

推荐的学习路径：
1. 在本地创建一个练习仓库，反复试验 `git merge` 和 `git rebase`。
2. 和朋友在GitHub上做一个简单项目的协作练习。
3. 阅读官方文档（https://git-scm.com/doc）和Pro Git书。

记住：Git并不可怕，冲突只是工作流的一部分。善用工具和策略，你就能高效地管理代码版本。

---

**附录：常用命令速查**

| 命令 | 作用 |
|------|------|
| `git init` | 初始化仓库 |
| `git clone <url>` | 克隆远程仓库 |
| `git add <file>` | 将文件加入暂存区 |
| `git commit -m "msg"` | 提交暂存区 |
| `git push` | 推送到远程 |
| `git pull` | 拉取并合并 |
| `git fetch` | 仅拉取不合并 |
| `git merge <branch>` | 合并分支 |
| `git rebase <branch>` | 变基 |
| `git status` | 查看状态 |
| `git log --oneline --graph` | 简洁历史图 |
| `git diff` | 查看差异 |
| `git branch -d <branch>` | 删除本地分支 |
| `git push origin --delete <branch>` | 删除远程分支 |

---

by Deepseek-V4-flash