# docuverse-decide-eval · 名词表准度

首次接手请读[入库指南](ONBOARDING.md)，其中列有开发时间线、当前实现和第一项验收任务。

独立评测仓（**不改**主仓 `docuverse-shanchule-peek-main`）。

## 一键推送到 GitHub（自带工具，无需部署）

本仓库自带推送脚本 `scripts/push-to-github.{ps1,sh}`，用 `gh` CLI 一键完成
**git init → 加文件 → 提交 → 在 GitHub 创建仓库 → 推送**。
下次你从 GitHub clone 下来这个项目，脚本就在里面，**直接用，不用重新配置**。

### 前置（一次性）

```powershell
# 1) 装 gh（已装可跳过）
winget install --id GitHub.cli
# 2) 登录
gh auth login
```

### 使用

**PowerShell（Windows，推荐）**
```powershell
.\scripts\push-to-github.ps1                 # 默认：私有仓、当前目录名
.\scripts\push-to-github.ps1 -Visibility public
.\scripts\push-to-github.ps1 -Message "feat: 调整 gold 题" -RepoName my-fork
.\scripts\push-to-github.ps1 -Force -DryRun  # 看一遍将要做什么再正式跑
```

**Bash（Git Bash / WSL / macOS / Linux）**
```bash
./scripts/push-to-github.sh
VISIBILITY=public ./scripts/push-to-github.sh
MESSAGE="feat: 调整 gold 题" REPO_NAME=my-fork ./scripts/push-to-github.sh
FORCE=1 DRY_RUN=1 ./scripts/push-to-github.sh
```

**或通过 npm**
```bash
npm run publish         # Windows 下调用 PowerShell 版
npm run publish:sh      # 类 Unix 环境
```

### 智能行为

| 场景 | 行为 |
| --- | --- |
| 当前没 `.git` | 自动 `git init -b main` |
| 远程仓库不存在 | `gh repo create` 创建后推送 |
| 远程仓库已存在但未设 remote | 询问用 https 还是 ssh 后 `git remote add origin` |
| 远程已有更新 | 自动 `pull --rebase` 再 push |
| 未设 git user | 自动用 GitHub 用户名占位 |
| 未登录 gh | 引导 `gh auth login` |
| 工作区干净 | 跳过 commit，直接 push 或结束 |
| 远程仓已显式存在 | `-NoCreate` / `NO_CREATE=1` 避免误建 |

幂等：重复跑不会重复创建仓库，远程配过的不会重配。

---

## 默认策略：名词表

```
建 S（规则去重）→（可选）残文 LLM 补 S₁
→ 勾选 S_q → 字符串定位 → ±n 异名词拓扑袋 → decide 过闸 → 对照 gold
```

## 启动 Web（导入语料）

```powershell
cd C:\Users\Administrator\Desktop\docuverse-decide-eval
# 复制 .env.example → .env 并填 LLM_API_KEY
& "C:\Program Files\cursor\resources\app\resources\helpers\node.exe" --experimental-strip-types src/server/main.ts
```

打开：**http://127.0.0.1:5188/**

1. 上传主仓导出的 `*.docuverse.json`  
2. 填 gold 题（`goldPages` / `goldBookKeys`）  
3. 「跑名词表评测」

## CLI

```powershell
node --experimental-strip-types scripts/eval-decide.ts --corpus path\to\book.docuverse.json --eval fixtures\eval.example.json
```

可选：`--residual-llm`

## 冒烟（无 LLM）

```powershell
node --experimental-strip-types scripts/smoke-noun.ts
```
