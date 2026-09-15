# Git 版本控制系統完整教學

> *(備註: 該文件由AI自動完成)*

## 目錄

1. [Git 簡介](#1-git-簡介)
2. [Git 安裝](#2-git-安裝)
3. [基本環境設定](#3-基本環境設定)
4. [本地 Git 操作](#4-本地-git-操作)
5. [分支管理](#5-分支管理)
6. [遠端 GitHub 整合](#6-遠端-github-整合)
7. [團隊協作流程](#7-團隊協作流程)
8. [Git 排錯表](#8-git-排錯表)
9. [常用 Git 指令表](#9-常用-git-指令表)

---

## 1. Git 簡介

### 什麼是 Git？

Git 是一個分散式版本控制系統，由 Linus Torvalds 於 2005 年開發。它能追蹤專案的所有變更、維護完整的修改歷史、支援多人協作開發。

### Git 的主要功能

- 版本追蹤：記錄每一次檔案變更
- 分支管理：支援多線程開發
- 衝突解決：自動偵測並提示手動處理衝突
- 回滾操作：快速還原到任何歷史版本
- 遠端同步：與 GitHub、GitLab 等遠端倉庫整合

### Git 的架構概念

- **工作區 (Working Directory)**：當前正在編輯的檔案
- **暫存區 (Staging Area/Index)**：準備提交變更的區域
- **本地倉庫 (Local Repository)**：包含所有版本歷史和分支資訊
- **遠端倉庫 (Remote Repository)**：GitHub、GitLab 等伺服器上的倉庫

---

## 2. Git 安裝

### Windows 系統安裝

1. 訪問 https://git-scm.com/download/win
2. 下載並執行安裝程式
3. 依照預設設定進行安裝
4. 安裝完成後，開啟終端機測試：

```bash
git --version
```

### macOS 系統安裝

使用 Homebrew 安裝：

```bash
brew install git
```

或使用 macports：

```bash
sudo port self update
sudo port install git
```

### Linux 系統安裝

使用 apt (Ubuntu/Debian)：

```bash
sudo apt update
sudo apt install git
```

使用 yum (CentOS/RHEL)：

```bash
sudo yum install git
```

使用 dnf (Fedora)：

```bash
sudo dnf install git
```

### 驗證安裝

執行以下指令確認 Git 已成功安裝：

```bash
git --version
git config --list
```

---

## 3. 基本環境設定

### 設定使用者資訊

Git 需要知道你的姓名和電子郵件，以便在提交時記錄資訊：

```bash
# 永久設定 (推薦)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 暫時設定 (僅當前專案)
git config user.name "Project Name"
git config user.email "project.email@example.com"
```

### 設定 SSH 金鑰 (用於遠端存取)

1. 生成 SSH 金鑰：

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

2. 複製公鑰內容：

```bash
cat ~/.ssh/id_ed25519.pub
```

3. 將公鑰新增到 GitHub 帳號的 SSH Keys 設定中

4. 測試連線：

```bash
ssh -T git@github.com
```

### 設定 Git 安全相關選項 (推薦)

```bash
# 強制使用 SSL
git config --global http.postBuffer 524288000
git config --global core.editor "vim"  # 可改為你常用的編輯器
git config --global push.default simple
git config --global init.defaultBranch main
```

---

## 4. 本地 Git 操作

### 初始化 Git 倉庫

在專案根目錄執行：

```bash
git init
```

這會建立 `.git` 目錄，包含所有版本控制資訊。

### 檢視狀態

```bash
# 查看所有變更狀態
git status

# 查看工作區變更
git diff

# 查看暫存區變更
git diff --cached
```

### 新增檔案到暫存區

```bash
# 新增單一檔案
git add filename.txt

# 新增所有變更
git add .

# 新增特定目錄下所有檔案
git add src/

# 新增並忽略特定檔案 (加入 .gitignore)
echo "node_modules/" >> .gitignore
git add .gitignore
```

### 提交變更

```bash
# 提交暫存區的變更
git commit -m "描述本次提交的訊息"

# 使用預設編輯器撰寫詳細提交訊息
git commit

# 快速提交 (不建議)
git commit -am "description"
```

### 查看提交歷史

```bash
# 查看所有提交記錄
git log

# 簡化顯示
git log --oneline

# 按時間排序
git log --reverse

# 查看檔案變更歷史
git log --follow filename.txt
```

### 回退操作

```bash
# 還原工作區，取消新增
git restore filename.txt

# 從暫存區移除 (不刪除檔案)
git restore --staged filename.txt

# 回滾到上一個提交
git reset --hard HEAD~1

# 回滾並保留變更 (進入暫存區)
git reset --soft HEAD~1
```

---

## 5. 分支管理

### 建立分支

```bash
# 建立新分支 (不切換)
git branch feature-login

# 建立並切換到新分支
git checkout -b feature-login

# Git 2.23+ 新版本建議用法
git switch -c feature-login
```

### 切換分支

```bash
# 使用 checkout 切換分支
git checkout feature-login

# 使用 switch 切換分支 (Git 2.23+)
git switch feature-login

# 強制切換到遠端分支
git checkout origin/main --force
```

### 合併分支

```bash
# 將指定分支合併到當前分支
git merge feature-login

# 快速合併 (Git 2.9+)
git merge --no-ff feature-login

# 處理衝突後手動合併
# 1. 編輯檔案解決衝突
# 2. 標記衝突已解決
git add filename.txt
# 3. 繼續合併流程
git commit
```

### 分支刪除

```bash
# 建立並刪除分支 (不切換)
git branch -d feature-login

# 強制刪除 (不建議)
git branch -D feature-login

# 刪除遠端分支
git push origin --delete feature-login
```

### 查看分支

```bash
# 查看所有本地分支
git branch

# 查看所有分支 (含遠端)
git branch -a

# 查看包含當前分支的資訊
git branch -v
```

---

## 6. 遠端 GitHub 整合

### 配置遠端倉庫

```bash
# 添加遠端倉庫 (預設名稱 origin)
git remote add origin https://github.com/username/repo.git

# 查看遠端設定
git remote -v

# 修改遠端地址
git remote set-url origin https://github.com/newowner/newrepo.git

# 移除遠端
git remote remove origin
```

### 推送程式碼到遠端

```bash
# 推送到遠端並建立新分支 (第一次推送)
git push -u origin feature-login

# 推送到遠端主分支
git push origin main

# 強制推送 (不建議用於團隊專案)
git push --force origin main

# 推送到特定遠端
git push upstream main
```

### 從遠端拉取程式碼

```bash
# 拉取最新變更
git pull origin main

# 先拉取再合併 (推薦)
git fetch origin
git merge origin/main

# 直接覆寫本地分支
git pull --rebase origin main
```

### 同步遠端和當地分支

```bash
# 推送到遠端並自動拉取
git push --set-upstream origin feature-login

# 簡寫命令
git push -u origin feature-login:feature-login
```

---

## 7. 團隊協作流程

### Git Flow 工作流程

Git Flow 是一種常用的分支管理策略，適合團隊專案開發。

#### 分支類型

- **main/master**：主線，用於穩定版本發布
- **develop**：開發線，整合所有功能
- **feature/**：功能分支，用於新功能開發
- **release/**：預发布分支，準備版本發布
- **hotfix/**：熱修复分支，處理緊急問題

#### 典型工作流程

1. **從 develop 建立功能分支**

```bash
git checkout develop
git pull origin develop
git checkout -b feature-new-feature
```

2. **開發並提交程式碼**

```bash
# 在功能分支上持續開發
git add .
git commit -m "feat: 新增使用者登入功能"
```

3. **建立 Pull Request (Pull Request)**

- 在 GitHub 介面建立 Pull Request
- 從 feature-new-feature 合併到 develop
- 等待 Code Review 通過

4. **合併後推回本地**

```bash
git checkout develop
git pull origin develop
```

5. **功能完成後清理分支**

```bash
git branch -d feature-new-feature
git push origin --delete feature-new-feature
```

### GitHub Flow (簡化流程)

適合小型團隊或快速開發：

1. 從 main 建立新分支
2. 開發並提交程式碼
3. 推送到遠端
4. 建立 Pull Request
5. Code Review 後合併到 main
6. 刪除功能分支

### 處理衝突

#### 常見衝突類型

- **檔案新增衝突**：同一檔案兩人都新增了內容
- **插入衝突**：在同一位置都進行了修改
- **取代衝突**：其中一人取代了另一人的內容

#### 解決衝突步驟

1. **拉取最新程式碼**

```bash
git pull origin main
```

2. **查看衝突檔案**

衝突標記格式：

```
<<<<<<< HEAD
你的程式碼
=======
隊友的程式碼
>>>>>>> branch-name
```

3. **編輯檔案解決衝突**

保留需要的內容，刪除 `<<<<<<<`、`=======`、`>>>>>>>` 標記

4. **標記衝突已解決並提交**

```bash
git add filename.txt
git commit -m "Resolve conflict: ..."
```

5. **繼續合併流程**

```bash
git push origin main
```

### Code Review 最佳實踐

- 使用 Pull Request 進行程式碼審查
- 設定審查人數要求 (至少 1 人)
- 設定程式碼規範 (Linting)
- 限制提交訊息長度
- 避免在 PR 中推入大量變更
- 及時回應審查意見

---

## 8. Git 排錯表

| 錯誤訊息 | 可能原因 | 解決方案 |
|---------|---------|---------|
| `fatal: Not a git repository` | 不在 Git 倉庫內 | 使用 `git init` 初始化倉庫，或進入正確的目錄 |
| `fatal: Authentication failed` | SSH/HTTPS 認證失敗 | 檢查 SSH 金鑰或 HTTPS Token，重新配置遠端連線 |
| `remote: Repository not found` | 倉庫不存在或名稱錯誤 | 檢查倉庫名稱和大小寫，確認已建立倉庫 |
| `fatal: Could not read Username` | 未輸入認證資訊 | 使用 SSH 金鑰或輸入 HTTPS Token |
| `error: Your local changes to the working tree have been modified` | 有未提交的變更 | 先提交變更或使用 `git reset --hard HEAD` 覆寫 |
| `error: Your branch is behind remote branch` | 本地分支落後於遠端 | 使用 `git pull origin main` 拉取並合併 |
| `error: The remote end hung up unexpectedly` | 網路問題或 SSH 連線中斷 | 檢查網路，重新配置 SSH 金鑰 |
| `fatal: ambiguous argument 'main'` | 分支名稱錯誤 | 確認遠端倉庫的主分支名稱 (可能是 master) |
| `error: Failed to push some refs` | 有未合併的提交 | 使用 `git push --force-with-lease` 或先拉取再推 |
| `fatal: Authentication failed for '/upload/...'` | GitHub Token 過期 | 重新生成新的 Personal Access Token |
| `error: Cannot rebase - A current cherry-pick is in progress` | 之前的 cherry-pick 未完成 | 執行 `git cherry-pick --abort` |
| `error: unable to access 'https://...'` | SSL 憑證問題 | 檢查日期時間設定，或更新 SSH 金鑰 |
| `Your branch is not up to date with origin/branch-name` | 分支不同步 | 使用 `git fetch && git rebase origin/branch-name` |
| `fatal: remote 'origin' does not exist` | 未設定遠端 | 使用 `git remote add origin <url>` 設定遠端 |

---

## 9. 常用 Git 指令表

### 基本操作

| 指令 | 說明 | 範例 |
|-----|------|------|
| `git init` | 初始化新的 Git 倉庫 | `git init` |
| `git clone <url>` | 複製遠端倉庫到本地 | `git clone https://github.com/user/repo.git` |
| `git status` | 檢視工作區狀態 | `git status` |
| `git add .` | 新增所有變更到暫存區 | `git add src/` |
| `git commit -m "訊息"` | 提交變更 | `git commit -m "完成登入功能"` |
| `git log` | 查看提交歷史 | `git log --oneline` |
| `git branch` | 列出所有分支 | `git branch -a` |
| `git checkout <branch>` | 切換到指定分支 | `git checkout develop` |
| `git merge <branch>` | 合併指定分支 | `git merge feature-login` |

### 遠端操作

| 指令 | 說明 | 範例 |
|-----|------|------|
| `git remote -v` | 列出所有遠端倉庫 | `git remote -v` |
| `git push origin <branch>` | 推送到遠端 | `git push origin main` |
| `git pull origin <branch>` | 從遠端拉取並合併 | `git pull origin main` |
| `git fetch origin` | 僅下載遠端變更 | `git fetch origin` |
| `git branch -M <name>` | 重新命名本地分支 | `git branch -M feature-login` |

### 進階操作

| 指令 | 說明 | 範例 |
|-----|------|------|
| `git stash` | 暫存未提交的變更 | `git stash` |
| `git stash pop` | 恢復最後一個暫存的變更 | `git stash pop` |
| `git reset --hard HEAD~1` | 回滾到上一個提交 | `git reset --hard HEAD~2` |
| `git revert <commit>` | 建立新提交來取消特定變更 | `git revert abc123def` |
| `git cherry-pick <commit>` | 將特定提交合併到當前分支 | `git cherry-pick abc123def` |
| `git rebase origin/main` | 重排提交歷史 (Git Rebase) | `git rebase -i HEAD~5` |

### 雜項

| 指令 | 說明 | 範例 |
|-----|------|------|
| `git config --list` | 查看所有 Git 設定 | `git config --list --global` |
| `git config --global user.name "Name"` | 設定使用者名稱 | `git config --global user.email "email@test.com"` |
| `git clean -fd` | 刪除未追蹤的檔案 | `git clean -fdx node_modules/` |
| `git diff` | 比較工作區與暫存區 | `git diff src/main.c` |
| `git diff --cached` | 比較暫存區與最後一次提交 | `git diff HEAD~1` |

---

## 補充：.gitignore 常用範例

### Node.js 專案

```
node_modules/
dist/
build/
.DS_Store
.env
*.log
```

### Python 專案

```
__pycache__/
*.pyc
*.pyo
venv/
env/
.env
*.egg-info/
```

### 通用範例

```
*.swp
*.swo
.DS_Store
Thumbs.db
*.log
npm-debug.log*
yarn-debug.log*
```

---

## 總結

Git 是現代軟體開發不可或缺的版本控制工具。掌握 Git 的基本操作、分支管理和團隊協作流程，將大幅提升專案效率和程式碼品質。

### 學習建議

1. **熟練基本指令**：init, clone, add, commit, push, pull
2. **理解分支概念**：掌握 feature branch 的工作模式
3. **練習衝突解決**：在真實環境中處理過衝突會更熟悉
4. **善用 GitHub/GitLab**：使用 Pull Request 進行程式碼審查
5. **定期同步遠端**：避免長時間隔離導致合併困難

### 進階主題參考

- Git Hooks (提交前自動執行腳本)
- Git Submodule (子模組管理)
- Git LFS (大型檔案版本控制)
- GitHub Actions (自動化工作流程)
- 持續整合/持續部署 (CI/CD)

---

*文件最後更新：2024*