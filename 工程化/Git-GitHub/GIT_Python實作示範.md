
# 本文檔由AI完全生程 所以會有很多問題 (用的模型太弱)

# Git Python 專案實作流程示範

本文檔提供完整的 Git 工作流程範例，包含初始化、分支管理、遠端推送、PR 合併、衝突處理等完整流程。

---

## 初始設定

### 1. 檢查環境並初始化專案

```bash
# 安裝 uv (Python 套件管理工具)
pip install uv

# 或者使用 curl 直接安裝
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 2. 初始化 Git 倉庫

```bash
git init

# 設定 .gitignore
cat > .gitignore << 'EOF'
__pycache__/
*.pyc
*.pyo
.pytest_cache/
.coverage
htmlcov/
.env
.venv/
venv/
ENV/
*.log
dist/
build/
*.egg-info/
.pytype/
.mypy_cache/
.ruff_cache/
EOF

# 新增 .gitignore
git add .gitignore

# 第一次提交 (chore: init project)
git commit -m "chore: init project"
```

**提交訊息：** `chore: init project`

---

## uv 專案設定

### 3. 初始化 uv 專案

```bash
# 使用 uv 建立虛擬環境並初始化 pyproject.toml
uv init .

# 新增開發依賴
uv add pytest black isort mypy

# 查看已安裝套件
uv pip list

# 提交 uv 專案設定 (feat: init uv project)
git add pyproject.toml uv.lock
git commit -m "feat: init uv project"
```

**提交訊息：** `feat: init uv project`

---

## 分支管理與遠端推送

### 4. 開分支並推送到遠端

```bash
# 建立功能分支 (branch feature-login)
git checkout -b feature/login-system

# 查看當前分支
git branch -v

# 推送到遠端並建立遠端分支 (push 追蹤)
git push -u origin feature/login-system

# 確認推送成功
git remote -v
```

**操作：**
- `git checkout -b feature/login-system` - 建立並切換到新分支
- `git push -u origin feature/login-system` - 推送到遠端並設定追蹤關係
- `-u` 參數 (或 `--set-upstream`) 會自動在下次推送時使用簡化指令

---

## 分支實作流程

### 5. 分支實作範例：新增使用者登入功能

```bash
# 編輯程式碼檔案
vim app/login.py

# 內容範例 (app/login.py)
"""
Login system implementation
"""

from typing import Optional, Dict
import hashlib


class User:
    def __init__(self, username: str, password_hash: str):
        self.username = username
        self.password_hash = password_hash
    
    def check_password(self, password: str) -> bool:
        return hashlib.sha256(password.encode()).hexdigest() == self.password_hash


class LoginSystem:
    def __init__(self):
        self.users: Dict[str, User] = {}
    
    def register(self, username: str, password: str) -> bool:
        """Register a new user"""
        if username in self.users:
            return False
        
        password_hash = hashlib.sha256(password.encode()).hexdigest()
        self.users[username] = User(username, password_hash)
        return True
    
    def login(self, username: str, password: str) -> Optional[User]:
        """Authenticate user"""
        if username not in self.users:
            return None
        
        if self.users[username].check_password(password):
            return self.users[username]
        
        return None


# 測試程式碼
if __name__ == "__main__":
    login_system = LoginSystem()
    
    # 註冊使用者
    success = login_system.register("alice", "password123")
    print(f"Registration: {success}")
    
    # 登入驗證
    user = login_system.login("alice", "password123")
    if user:
        print(f"Login successful for: {user.username}")
    else:
        print("Login failed")
```

```bash
# 新增程式碼檔案
git add app/login.py

# 提交變更 (feat: implement login system)
git commit -m "feat: implement login system"
```

**提交訊息：** `feat: implement login system`

---

## 推送到遠端並建立 Pull Request

### 6. 推送到遠端

```bash
# 推送當前分支到遠端
git push origin feature/login-system

# 在 GitHub/GitLab 網頁介面建立 Pull Request
# 步驟：
# 1. 進入專案首頁
# 2. 點擊 Compare & pull request 按鈕
# 3. 確保來源為 feature/login-system
# 4. 目標為 main 或 develop
# 5. 填寫 PR 描述：
"""
## What does this PR do?

Implement basic login system for user authentication.

## Type of change

- [ ] Bug fix
- [x] New feature
- [ ] Breaking change

## Checklist

- [x] Code follows the project style guide
- [x] Tests added/updated as necessary
- [x] Documentation updated if necessary

## Screenshots (if applicable)

"""
# 6. 邀請 Code Reviewer
# 7. 等待審查與合併
```

---

## 處理分支衝突

### 7. 模擬衝突情境：隊友也在修改同一檔案

```bash
# 假設隊友推送到遠端
# git push origin main

# 你拉取最新變更 (會發現衝突)
git pull origin main

# 衝突顯示範例：
"""
conflict marker in app/login.py:
<<<<<<< HEAD
    def login(self, username: str, password: str):
        # 你的程式碼
        ...
=======
    def login(self, username: str, password: str, remember_me: bool = False):
        # 隊友的程式碼 - 新增了 remember_me 參數
        ...
>>>>>>> main
"""
```

**衝突解決步驟：**

```bash
# 1. 編輯檔案解決衝突
vim app/login.py

# 合併後的內容 (保留雙方功能)
"""
from typing import Optional, Dict
import hashlib


class User:
    def __init__(self, username: str, password_hash: str):
        self.username = username
        self.password_hash = password_hash
    
    def check_password(self, password: str) -> bool:
        return hashlib.sha256(password.encode()).hexdigest() == self.password_hash


class LoginSystem:
    def __init__(self):
        self.users: Dict[str, User] = {}
    
    def register(self, username: str, password: str) -> bool:
        """Register a new user"""
        if username in self.users:
            return False
        
        password_hash = hashlib.sha256(password.encode()).hexdigest()
        self.users[username] = User(username, password_hash)
        return True
    
    def login(
        self, 
        username: str, 
        password: str, 
        remember_me: bool = False
    ) -> Optional[User]:
        """Authenticate user with optional remember me feature"""
        if username not in self.users:
            return None
        
        if self.users[username].check_password(password):
            # 新增 remember_me 功能
            if remember_me:
                # TODO: Implement remember me logic
                pass
            
            return self.users[username]
        
        return None
    
    def logout(self, username: str) -> bool:
        """Logout user"""
        if username in self.users:
            del self.users[username]
            return True
        return False


# 測試程式碼
if __name__ == "__main__":
    login_system = LoginSystem()
    
    # 註冊使用者
    success = login_system.register("alice", "password123")
    print(f"Registration: {success}")
    
    # 登入驗證
    user = login_system.login("alice", "password123")
    if user:
        print(f"Login successful for: {user.username}")
        
        # 測試 remember me
        user = login_system.login("alice", "password123", remember_me=True)
        if user:
            print("Remember me enabled")
    else:
        print("Login failed")
    
    # 登出
    login_system.logout("alice")
    print("User logged out")
"""

# 2. 標記衝突已解決
git add app/login.py

# 3. 繼續合併流程
git commit -m "Merge branch 'main' into feature/login-system"

# 4. 推送遠端 (PR 會自動更新)
git push origin feature/login-system
```

---

## 軟回朔 (Soft Reset)

### 8. 軟回朔：保留變更並回到上一個提交

```bash
# 情境：不小心提交了錯誤內容，想回到上一個提交但保留變更

# 查看當前提交歷史
git log --oneline -5

# 假設要回朔到 HEAD~1 (上一個提交)
# 軟回朔會保留工作區的變更
git reset --soft HEAD~1

# 檢查狀態
git status

# 輸出範例：
"""
On branch feature/login-system
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   app/login.py
        
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   app/login.py
"""

# 解釋：
# - --soft 會回朔到上一個提交，但保留變更在工作區和暫存區
# - 需要重新 add 並 commit
```

**後續操作：**

```bash
# 重新新增檔案到暫存區
git add app/login.py

# 重新提交 (可以修改提交訊息)
git commit -m "feat: implement login system with remember me"

# 或者保留當前提交訊息
git commit --amend -m "feat: implement login system"
```

---

## 強退回版本 (Hard Reset)

### 9. 強退回版本：完全還原到指定提交

**警告：** 這會刪除所有未提交的變更！生產環境慎用！

```bash
# 情境：想要完全還原到某個提交，放棄所有變更

# 查看當前與目標提交的差異
git log --oneline HEAD~3..HEAD

# 強退回 (會刪除所有變更)
git reset --hard HEAD~2

# 或者退回遠端特定分支的某次提交
git reset --hard origin/main@{1}

# 確認狀態
git status

# 輸出範例：
"""
On branch feature/login-system
nothing to commit, working tree clean
"""

# 解釋：
# - --hard 會同時還原工作區和暫存區
# - 所有未提交的變更都會被刪除
# - 此操作不可逆，請謹慎使用
```

**安全替代方案：**

```bash
# 如果只是想取消新增 (保留檔案)
git restore --staged app/login.py

# 如果想還原工作區但保留在暫存區
git reset --soft HEAD~1

# 如果想刪除檔案並還原到遠端版本
git checkout origin/main -- app/login.py
```

---

## 回朔到特定提交前 (Mixed Reset)

### 10. Mixed Reset：還原工作區，保留暫存區

```bash
# 情境：想還原工作區變更，但保留暫存區內容

git reset --mixed HEAD~1

# 或者簡寫為：
git reset HEAD~1

# 輸出範例：
"""
On branch feature/login-system
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   app/login.py
        
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   app/login.py
"""

# 解釋：
# - Mixed 是預設行為
# - 工作區變更被還原
# - 暫存區內容保留
```

---

## 使用 Rebase 整理提交歷史

### 11. Rebase 操作：重排提交歷史

```bash
# 情境：想將當前分支的提交應用到遠端分支的最新上

# 互動式 rebase (可調整提交順序、合併提交等)
git rebase -i origin/main

# 非互動式 rebase (自動應用變更)
git rebase origin/main

# 處理衝突後的操作
# 當遇到衝突時：
git add app/login.py
git rebase --continue

# 如果不想繼續這個 rebase
git rebase --abort

# 如果想要放棄整個 rebase，回到原來的狀態
git rebase --abort
```

**Rebase 與 Merge 的比較：**

| 特性 | Rebase | Merge |
|------|--------|-------|
| 提交歷史 | 線性化，乾淨整齊 | 保留合併分支點 |
| 衝突處理 | 逐個處理 | 一次性處理所有衝突 |
| 團隊協作 | 不建議用於公開分支 | 適合共享分支 |
| 使用時機 | 本地功能分支 | main/develop 等公共分支 |

---

## Cherry-pick：選擇性合併提交

### 12. Cherry-pick：應用特定提交

```bash
# 情境：想將某個提交的功能應用到當前分支，但不需要其歷史

# 查看提交 hash
git log --oneline

# 假設要 cherry-pick 的提交是 abc1234
git cherry-pick abc1234

# 或者 cherry-pick 多個提交
git cherry-pick abc1234 def5678

# 取消 cherry-pick (遇到衝突時)
git cherry-pick --abort

# 跳過衝突的提交
git cherry-pick --continue
```

---

## Stash：暫存未提交的變更

### 13. Stash 操作：暫時保存工作

```bash
# 情境：想切換分支但不想丟棄未提交的變更

# 新增所有變更到暫存區
git add .

# 儲存到 stash (保留檔案系統狀態)
git stash save "Work on login system"

# 或者不加說明 (預設名稱: WIP on branch-name)
git stash

# 查看 stash 列表
git stash list

# 輸出範例：
"""
stash@{0}:  Work on login system
stash@{1}:  WIP on feature/login-system
"""

# 恢復最後一個 stash
git stash pop

# 恢復指定 stash (保留在 stash 中)
git stash apply stash@{0}

# 恢復並刪除 stash
git stash show -p stash@{0} | git apply

# 刪除特定的 stash (不恢復內容)
git stash drop stash@{0}

# 清除所有 stash
git stash clear
```

---

## Fetch vs Pull：遠端同步策略

### 14. 遠端同步操作

```bash
# fetch: 僅下載遠端變更，不合併
git fetch origin

# pull: 先 fetch 再 merge
git pull origin main

# pull with rebase (推薦用於功能分支)
git pull --rebase origin main

# push 選項說明
git push              # 推送本地變更到遠端
git push -u origin branch-name   # 建立追蹤關係
git push --force      # 強制覆寫遠端 (慎用!)
git push --force-with-lease  # 安全的強制推送

# fetch + rebase + push 的完整流程
git fetch origin
git rebase origin/main
git push --force-with-lease origin feature/login-system
```

---

## 主動製造衝突：README.md 範例

### 17. README.md 衝突實作流程

這個範例示範如何主動製造衝突，讓學員理解衝突產生與解決的過程。

```bash
# === Step 1: 切回 main 分支 ===
git checkout main

# === Step 2: 在 main 上建立 README.md ===
cat > README.md << 'EOF'
# Python Git 專案

這是一個用於學習 Git 的版本控制專案。

## 功能列表
- [ ] 使用者登入系統
- [ ] 資料管理模組
- [ ] API 整合

## 安裝說明

```bash
pip install -r requirements.txt
```

## 開發者指南

請在 `feature/` 分支下建立新功能。

```bash
# === Step 3: 新增並提交到 main ===
git add README.md
git commit -m "docs: add initial README with project overview"

# === Step 4: 推送到遠端 ===
git push origin main
```

**狀態確認：**
```bash
git status
# 輸出：On branch main, working tree clean
```

---

### 18. 切換到新分支並製造衝突

```bash
# === Step 5: 開新分支 ===
git checkout -b feature/login-system

# === Step 6: 在同一個 README.md 新增內容 ===
cat >> README.md << 'EOF'

## 使用者登入系統

### 功能說明
提供安全的 SHA-256 密碼哈希驗證。

### API 端點

| 方法 | 路徑 | 說明 |
|------|------|------|
| POST | /api/auth/register | 註冊新用戶 |
| POST | /api/auth/login | 登入驗證 |
| GET | /api/user/profile | 獲取當前用戶資訊 |

### 錯誤代碼

| 代碼 | 說明 |
|------|------|
| 401 | 未授權 |
| 403 | 權限不足 |
| 404 | 資源不存在 |
| 422 | 請求無效 |
EOF

# === Step 7: 新增並提交 ===
git add README.md
git commit -m "docs: add login system documentation and API reference"

```

**狀態確認：**
```bash
git status
# 輸出：On branch feature/login-system, working tree clean
```

---

### 19. 拉取並觸發衝突

```bash
# === Step 8: 切回 main 並拉取最新 ===
git checkout main
git pull origin main

# === Step 9: 再次切換到 feature 分支 ===
git checkout feature/login-system

# === Step 10: 嘗試拉取 (會觸發衝突) ===
git pull origin main

```

**預期結果 - Git 會顯示：**
```
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

**README.md 中的衝突標記：**
```markdown
# Python Git 專案

這是一個用於學習 Git 的版本控制專案。

<<<<<<< HEAD
## 功能列表
- [ ] 使用者登入系統
- [ ] 資料管理模組
- [ ] API 整合

## 安裝說明

```bash
pip install -r requirements.txt
```


## 安裝說明

```bash
pip install -r requirements.txt
```


```bash
### === Step 12: 標記衝突已解決 ===
git add README.md

# === Step 13: 提交合併結果 ===
git commit -m "Merge branch 'main' into feature/login-system"
```


---

### 21. 推送到遠端 (PR 自動更新)

```bash
# === Step 14: 推送當前分支 ===
git push origin feature/login-system



# PR 會自動反映在 GitHub/GitLab 上
```

---

## 衝突類型說明

| 衝突類型       | 說明            | 解決方法          |
| ---------- | ------------- | ------------- |
| **新增檔案衝突** | 同一檔案兩人都新增了內容  | 合併雙方內容，保留完整功能 |
| **插入衝突**   | 在同一位置都進行了修改   | 選擇保留哪一方的修改或合併 |
| **取代衝突**   | 其中一人取代了另一人的內容 | 手動編輯保留需要的部分   |

---

## 主動製造衝突的教學價值

1. **讓學員親身體驗衝突產生過程**
   - 看到 Git 如何標記衝突區域
   - 理解 `<<<<<<<`、`=======`、`>>>>>>>` 的作用

2. **學習衝突解決技巧**
   - 識別哪些內容必須保留
   - 合併雙方的貢獻
   - 保持程式碼邏輯完整

3. **理解 Git 的合併機制**
   - Git 嘗試自動合併相同區域
   - 當兩者都修改同一行時需要手動解決

4. **培養團隊協作意識**
   - 避免在 PR 上有衝突
   - 在本地完成所有開發工作再推送到遠端

---

## Pull Request 合併流程

### 15. GitHub/GitLab PR 合併步驟

**在網頁介面操作：**

```bash
# 1. 在 GitHub 建立 Pull Request
#    - Source: feature/login-system
#    - Destination: main

# 2. 本地推送到遠端 (PR 會自動更新)
git push origin feature/login-system

# 3. 等待 Code Review
#    - 團隊成員審查程式碼
#    - 提出建議或要求修改

# 4. 回應審查意見並重新提交
#    修改檔案後：
git add .
git commit -m "fix: resolve review comments"
git push origin feature/login-system

# 5. Maintainer 合併 PR
#    在 GitHub 點擊 Merge pull request
#    選擇 merge method:
#    - squash and merge (壓縮所有提交為一個)
#    - rebase and merge (rebase 到目標分支)
#    - create a merge commit (保留所有提交歷史)

# 6. 合併後清理本地分支
git checkout main
git pull origin main

# 7. 刪除本地功能分支
git branch -d feature/login-system

# 8. 從遠端刪除已合併的分支
git push origin --delete feature/login-system
```

---

## 完整工作流程範例

### 16. 完整開發流程示範

```bash
# === Phase 1: 初始化專案 ===

git init
cat > .gitignore << 'EOF'
__pycache__/
*.pyc
venv/
.env
EOF
git add .gitignore
git commit -m "chore: init project"

uv init .
uv add pytest black
git add pyproject.toml uv.lock
git commit -m "feat: init uv project"


# === Phase 2: 開發功能分支 ===

git checkout -b feature/user-authentication

# 新增檔案並提交多次
echo "# Auth Module" > auth/__init__.py
git add auth/__init__.py
git commit -m "feat: add auth module skeleton"

vim auth/login.py
git add auth/login.py
git commit -m "feat: implement basic login logic"

vim auth/register.py
git add auth/register.py
git commit -m "feat: implement user registration"


# === Phase 3: 推送到遠端並建立 PR ===

git push -u origin feature/user-authentication

# GitHub 介面：建立 Pull Request
# Source: feature/user-authentication
# Destination: main
# Description:
"""
## Feature: User Authentication System

### What does this PR do?
Implements basic user authentication functionality including:
- User registration with password hashing
- Login validation
- Session management

### Type of change
- [x] New feature

### Checklist
- [x] Code follows project style guide
- [x] Tests added
- [x] Documentation updated

### Screenshots (if applicable)
"""


# === Phase 4: 處理審查意見 ===

# 修改程式碼回應審查
vim auth/login.py
git add auth/login.py
git commit -m "fix: improve password hashing security"

git push origin feature/user-authentication


# === Phase 5: PR 合併 ===

# GitHub 介面：Maintainer 點擊 Merge pull request
# 選擇 squash and merge


# === Phase 6: 清理本地分支 ===

git checkout main
git pull origin main
git branch -d feature/user-authentication
git push origin --delete feature/user-authentication


# === Phase 7: 建立下一個功能 ===

git checkout -b feature/payment-integration
```

---

## 常用指令速查表

### 基本操作

| 指令 | 說明 | 範例 |
|------|------|------|
| `git init` | 初始化 Git 倉庫 | `git init` |
| `git add .` | 新增所有變更 | `git add src/` |
| `git commit -m "msg"` | 提交變更 | `git commit -m "feat: new feature"` |
| `git status` | 檢視狀態 | `git status` |
| `git log --oneline` | 查看歷史 | `git log --oneline -10` |

### 分支操作

| 指令 | 說明 | 範例 |
|------|------|------|
| `git branch feature/name` | 建立分支 | `git checkout -b feature/login` |
| `git checkout -b name` | 切換並建立分支 | `git switch -c feature/payments` |
| `git merge branch` | 合併分支 | `git merge feature/login` |
| `git branch -d name` | 刪除分支 | `git push origin --delete feature/login` |

### 遠端操作

| 指令 | 說明 | 範例 |
|------|------|------|
| `git remote add origin url` | 設定遠端 | `git remote -v` |
| `git push -u origin branch` | 推送並追蹤 | `git push origin main` |
| `git pull origin main` | 拉取並合併 | `git fetch origin` |

### 進階操作

| 指令 | 說明 | 範例 |
|------|------|------|
| `git stash` | 暫存變更 | `git stash pop` |
| `git reset --soft HEAD~1` | 軟回朔 | `git reset --hard HEAD~2` |
| `git rebase -i main` | 互動式重排歷史 | `git rebase origin/main` |
| `git cherry-pick commit` | 選擇性合併提交 | `git cherry-pick abc1234` |

---

## 注意事項

### ⚠️ 危險操作警告

1. **不要使用 `git push --force`**
   - 除非確定是本地實驗分支
   - 會覆蓋遠端歷史，可能導致他人工作丟失

2. **謹慎使用 `git reset --hard`**
   - 會刪除所有未提交的變更
   - 生產環境絕對避免

3. **不要直接在 main/develop 上開發**
   - 建立功能分支進行開發
   - 減少衝突並保持歷史整齊

4. **提交訊息要清楚**
   - 使用 Conventional Commits 格式
   - 說明變更內容和原因

### ✅ 最佳實踐

1. **頻繁提交，小步快跑**
   ```bash
   git add .
   git commit -m "feat: implement X functionality"
   ```

2. **使用有意義的分支名稱**
   ```bash
   feature/login-system       # 功能新增
   bugfix/password-reset      # 錯誤修正
   chore/update-dependencies  # 雜項變更
   docs/update-readme         # 文件更新
   ```

3. **定期拉取遠端變更**
   ```bash
   git fetch origin
   git rebase origin/main
   ```

4. **在本地解決衝突，不要讓 PR 有衝突**
   - 完成功能後再推送到遠端
   - 避免在多人同時編輯同一檔案

---

## 總結

完整的 Git 工作流程包含：

1. **初始化專案**：`git init`, `uv init`, `.gitignore`
2. **分支管理**：建立功能分支、開發、合併
3. **遠端同步**：push, pull, fetch, rebase
4. **PR 流程**：審查、修改、合併、清理
5. **衝突處理**：識別、解決、標記完成
6. **進階操作**：stash, reset, rebase, cherry-pick

掌握這些概念和技巧，就能有效使用 Git 進行團隊協作開發。

---

*文件最後更新：2024*
```

### 15. GitHub/GitLab PR 合併步驟

**在網頁介面操作：**

```bash
# 1. 在 GitHub 建立 Pull Request
#    - Source: feature/login-system
#    - Destination: main

# 2. 本地推送到遠端 (PR 會自動更新)
git push origin feature/login-system

# 3. 等待 Code Review
#    - 團隊成員審查程式碼
#    - 提出建議或要求修改

# 4. 回應審查意見並重新提交
#    修改檔案後：
git add .
git commit -m "fix: resolve review comments"
git push origin feature/login-system

# 5. Maintainer 合併 PR
#    在 GitHub 點擊 Merge pull request
#    選擇 merge method:
#    - squash and merge (壓縮所有提交為一個)
#    - rebase and merge (rebase 到目標分支)
#    - create a merge commit (保留所有提交歷史)

# 6. 合併後清理本地分支
git checkout main
git pull origin main

# 7. 刪除本地功能分支
git branch -d feature/login-system

# 8. 從遠端刪除已合併的分支
git push origin --delete feature/login-system
```

---

## 完整工作流程範例

### 16. 完整開發流程示範

```bash
# === Phase 1: 初始化專案 ===

git init
cat > .gitignore << 'EOF'
__pycache__/
*.pyc
venv/
.env
EOF
git add .gitignore
git commit -m "chore: init project"

uv init .
uv add pytest black
git add pyproject.toml uv.lock
git commit -m "feat: init uv project"


# === Phase 2: 開發功能分支 ===

git checkout -b feature/user-authentication

# 新增檔案並提交多次
echo "# Auth Module" > auth/__init__.py
git add auth/__init__.py
git commit -m "feat: add auth module skeleton"

vim auth/login.py
git add auth/login.py
git commit -m "feat: implement basic login logic"

vim auth/register.py
git add auth/register.py
git commit -m "feat: implement user registration"


# === Phase 3: 推送到遠端並建立 PR ===

git push -u origin feature/user-authentication

# GitHub 介面：建立 Pull Request
# Source: feature/user-authentication
# Destination: main
# Description:
"""
## Feature: User Authentication System

### What does this PR do?
Implements basic user authentication functionality including:
- User registration with password hashing
- Login validation
- Session management

### Type of change
- [x] New feature

### Checklist
- [x] Code follows project style guide
- [x] Tests added
- [x] Documentation updated

### Screenshots (if applicable)
"""


# === Phase 4: 處理審查意見 ===

# 修改程式碼回應審查
vim auth/login.py
git add auth/login.py
git commit -m "fix: improve password hashing security"

git push origin feature/user-authentication


# === Phase 5: PR 合併 ===

# GitHub 介面：Maintainer 點擊 Merge pull request
# 選擇 squash and merge


# === Phase 6: 清理本地分支 ===

git checkout main
git pull origin main
git branch -d feature/user-authentication
git push origin --delete feature/user-authentication


# === Phase 7: 建立下一個功能 ===

git checkout -b feature/payment-integration
```

---

## 常用指令速查表

### 基本操作

| 指令 | 說明 | 範例 |
|------|------|------|
| `git init` | 初始化 Git 倉庫 | `git init` |
| `git add .` | 新增所有變更 | `git add src/` |
| `git commit -m "msg"` | 提交變更 | `git commit -m "feat: new feature"` |
| `git status` | 檢視狀態 | `git status` |
| `git log --oneline` | 查看歷史 | `git log --oneline -10` |

### 分支操作

| 指令 | 說明 | 範例 |
|------|------|------|
| `git branch feature/name` | 建立分支 | `git checkout -b feature/login` |
| `git checkout -b name` | 切換並建立分支 | `git switch -c feature/payments` |
| `git merge branch` | 合併分支 | `git merge feature/login` |
| `git branch -d name` | 刪除分支 | `git push origin --delete feature/login` |

### 遠端操作

| 指令 | 說明 | 範例 |
|------|------|------|
| `git remote add origin url` | 設定遠端 | `git remote -v` |
| `git push -u origin branch` | 推送並追蹤 | `git push origin main` |
| `git pull origin main` | 拉取並合併 | `git fetch origin` |

### 進階操作

| 指令 | 說明 | 範例 |
|------|------|------|
| `git stash` | 暫存變更 | `git stash pop` |
| `git reset --soft HEAD~1` | 軟回朔 | `git reset --hard HEAD~2` |
| `git rebase -i main` | 互動式重排歷史 | `git rebase origin/main` |
| `git cherry-pick commit` | 選擇性合併提交 | `git cherry-pick abc1234` |

---

## 注意事項

### ⚠️ 危險操作警告

1. **不要使用 `git push --force`**
   - 除非確定是本地實驗分支
   - 會覆蓋遠端歷史，可能導致他人工作丟失

2. **謹慎使用 `git reset --hard`**
   - 會刪除所有未提交的變更
   - 生產環境絕對避免

3. **不要直接在 main/develop 上開發**
   - 建立功能分支進行開發
   - 減少衝突並保持歷史整齊

4. **提交訊息要清楚**
   - 使用 Conventional Commits 格式
   - 說明變更內容和原因

### ✅ 最佳實踐

1. **頻繁提交，小步快跑**
   ```bash
   git add .
   git commit -m "feat: implement X functionality"
   ```

2. **使用有意義的分支名稱**
   ```bash
   feature/login-system       # 功能新增
   bugfix/password-reset      # 錯誤修正
   chore/update-dependencies  # 雜項變更
   docs/update-readme         # 文件更新
   ```

3. **定期拉取遠端變更**
   ```bash
   git fetch origin
   git rebase origin/main
   ```

4. **在本地解決衝突，不要讓 PR 有衝突**
   - 完成功能後再推送到遠端
   - 避免在多人同時編輯同一檔案

---

## 總結

完整的 Git 工作流程包含：

1. **初始化專案**：`git init`, `uv init`, `.gitignore`
2. **分支管理**：建立功能分支、開發、合併
3. **遠端同步**：push, pull, fetch, rebase
4. **PR 流程**：審查、修改、合併、清理
5. **衝突處理**：識別、解決、標記完成
6. **進階操作**：stash, reset, rebase, cherry-pick

掌握這些概念和技巧，就能有效使用 Git 進行團隊協作開發。

---

*文件最後更新：2024*
