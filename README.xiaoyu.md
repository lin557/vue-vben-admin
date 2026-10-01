# 工作流：`main` 严格跟随官方上游，自定义分支放你的改动

> 
> 核心原则：
> ✅ **本地 main 分支：永远只用来同步官方代码，绝不写任何自己的修改**
> ✅ **你的自定义分支（例如 `my-patch`）：存放你所有改动，定期把 main（官方最新代码）合并进来**
> ✅ GitHub 上 fork 的 `origin/main` 和本地 main 保持一致，永远等于上游官方代码

## 一、一次性初始化（只做一次）

```
# 克隆你的fork仓库
git clone https://github.com/你的名字/项目.git
cd 项目

# 添加上游官方仓库（仅一次）
git remote add upstream https://github.com/原作者/项目.git
git remote -v
```

## 二、日常操作：同步官方到 main（定期执行）

> 
> 每次想要拉取官方最新代码时执行：

```
# 切到本地main
git checkout main

# 拉取上游所有最新提交（不会自动合并，安全）
git fetch upstream

# 把上游main合并到本地main，此时本地main = 官方最新
git merge upstream/main

# 推送到你github fork仓库的main分支
git push origin main
```

> 
> ⚠️ 重点：**main 分支绝对不要在这里写任何代码、提交任何修改！**
> 本地 main 只做一件事：镜像官方仓库。

> 
> 可选：想用 rebase 保持 main 历史干净（推荐）

```
git checkout main
git fetch upstream
git rebase upstream/main
git push origin main
```

> 
> main 分支用 rebase 基本不会有冲突，因为 main 没有你的任何改动。

## 三、新建自定义分支，存放你的修改

```
# 从最新的main分支创建你的补丁分支
git checkout main
git checkout -b my-patch

# 在 my-patch 里面随便改代码、提交
git add .
git commit -m "自定义修改：xxxx"
git push origin my-patch
```

以后所有自己的修改，**全部在 `my-patch` 分支上提交，不要碰 main**。

## 四、当 main 同步了官方更新后，把更新合并进你的自定义分支

当你执行完上面【同步官方到 main】之后，`main` 已经是最新代码，需要把这些官方更新合并到你的 `my-patch`：

### 方案 A：merge（简单，推荐新手）

```
git checkout my-patch
git merge main
```

- 如果发生冲突：手动解决冲突文件，`git add`，然后 `git commit`
- 完成后推送到 fork：

```
git push origin my-patch
```

优点：逻辑直观，提交历史保留你的所有补丁，冲突处理更友好。

### 方案 B：rebase（提交历史干净，适合长期维护补丁）

```
git checkout my-patch
git rebase main
```

- 冲突：解决冲突，`git add`，`git rebase --continue`；放弃：`git rebase --abort`

> 
> ⚠️ rebase 会改写 my-patch 提交历史，**如果分支已经推送到 github，rebase 之后推送需要加 --force-with-lease**

```
git push --force-with-lease origin my-patch
```

## 完整工作流时序举例

1. `main` 同步官方 → origin/main 更新
2. `my-patch` 从 main 创建，写入你的自定义改动
3. 一段时间后，官方仓库更新了
4. 再次执行：`checkout main → fetch upstream → merge/rebase upstream/main → push origin main`
5. 切回 `my-patch`，`merge main` 或者 `rebase main`，把官方新代码并入你的分支，解决冲突
6. 继续在 `my-patch` 开发新功能

## 关键红线（一定要遵守，否则会乱）

1. ❌ **不要在 main 提交任何自己的修改**，main 只镜像上游
2. ❌ 不要在 github 网页点 `Sync fork` 之后又在本地 main 提交代码
3. ✅ 所有自定义修改只放在独立分支（my-patch）
4. ✅ 每次同步上游，一定先更新本地 main，再合并到自定义分支

## 小技巧

- 可以写别名简化命令，例如

```
# 一键同步上游到main
git config --global alias.sync-up '!git checkout main && git fetch upstream && git merge upstream/main && git push origin main'
```

之后执行 `git sync-up` 就自动同步官方到 main。
