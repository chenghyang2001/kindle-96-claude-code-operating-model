# K96 快速演練課表

依據：原書目錄（截圖 5–18）。書上沒有隨書程式碼；以下提示詞是依各章小節自行撰寫的演練版本，盡量用到該章點名的機制。
共用素材：第 1 章建立的小專案 `taskbox`（Python 待辦清單 CLI＋pytest），後面各章先複製一份再加工，章與章互不干擾。

每段提示詞前面都要加：「全自動執行：不要問我任何問題、不要用選擇題，所有選擇一律採用你的推薦，直接做完並回報」。

範圍限制（每章都適用）：

- hooks、MCP、plugin、權限一律寫在練習專案的 `.claude/settings.json`／`.claude/settings.local.json`／`.mcp.json`，不可動全域 `~/.claude`。
- 不設 `ANTHROPIC_API_KEY`；`claude -p` 與 Agent SDK 走已登入的訂閱。
- 不建 GitHub repo、不 push、不跑 GitHub Actions。
- 練習裡呼叫 `claude -p` 時一律加 `--max-turns`（≤ 8），避免失控。

## 第 1 章 Agent 框架與四層架構（k96-ch01-practice）

1. 提示詞：

> 用 Python 標準函式庫做一個小專案 taskbox：`taskbox.py` 提供 add／list／done／remove 四個子指令，資料存在 `tasks.json`；`tests/test_taskbox.py` 用 pytest 測四個指令。用 uv 建虛擬環境並裝 pytest，跑測試全部通過。

1. 提示詞：

> 寫 `ARCHITECTURE-NOTES.md`：用本書的四層模型（記憶層 CLAUDE.md、擴充層 Skills／Sub-agents／Hooks／斜線指令、整合層 MCP、程式層 Agent SDK）說明「如果 taskbox 要交給 Claude Code 長期維護」每一層各放什麼檔案、何時觸發（比較四種觸發機制：使用者主動、模型自動判斷、事件、外部呼叫）。最後用 `claude -p "列出本專案的檔案並說明 taskbox.py 的功能" --output-format stream-json --verbose --max-turns 5` 跑一次，統計 stream 裡的訊息類型與 tool_use 次數，畫出這次請求的資料流（使用者 → 模型 → 工具 → 結果 → 回覆），附在文件最後。

驗證：pytest 通過；文件有四層對照表與實際 stream-json 統計。

## 第 2 章 記憶系統 CLAUDE.md（k96-ch02-practice）

1. 把 `k96-ch01-practice` 的 taskbox 複製到本章資料夾（不含 .venv）。
2. 提示詞：

> 依本書「WHY／WHAT／HOW 三問」寫本專案的 `CLAUDE.md`（為什麼這樣設計、要做與不要做、怎麼一步步做：環境、測試指令、提交規則），控制在 60 行內。另外：
>
> - `CLAUDE.local.md`：個人偏好（例如回覆用繁體中文），並加進 .gitignore
> - 條件規則 `.claude/rules/tests.md`：frontmatter 用 `paths: ["tests/**"]`，規定測試命名與 fixture 寫法
> - 寫 `MEMORY-NOTES.md`：說明本書的五層記憶架構各在哪個檔案，本專案用到哪幾層
> 驗證：用 `claude -p "本專案的測試指令是什麼？測試檔有什麼命名規則？只回答兩行" --max-turns 3`，確認回答引用了 CLAUDE.md 與條件規則。

驗證：`claude -p` 的回答和 CLAUDE.md 內容一致。

## 第 3 章 可重用的 Skills（k96-ch03-practice）

1. 把 taskbox 複製到本章資料夾，`git init` 並做第一次 commit。
2. 提示詞：

> 在本專案 `.claude/skills/` 建三個 skill，照本書做法：
>
> 1. `smart-commit`（任務型）：讀 `git diff --staged` 產生 Conventional Commit 訊息後提交；frontmatter 寫清楚 description（何時用、何時不用）、`allowed-tools` 只開 `Bash(git diff:*)`、`Bash(git commit:*)`、`Bash(git status:*)`；支援 `$ARGUMENTS` 當 scope
> 2. `taskbox-conventions`（參考型）：本專案的程式風格，SKILL.md 當目錄（router），細節放 `references/style.md`
> 3. `task-report`（組合型，漸進揭露）：讀 `tasks.json` 算完成率、各狀態數量，SKILL.md 只放流程，計算腳本放 `scripts/report.py`，報表範本放 `references/template.md`
> 每個 SKILL.md 都在 500 行內。寫 `SKILLS-NOTES.md` 說明三者的觸發條件與防止過度觸發的寫法。
> 驗證：改一行 taskbox.py 並 `git add`，跑 `claude -p "/smart-commit cli" --max-turns 6`，確認產生了 commit；再跑 `claude -p "幫我看目前待辦完成率" --max-turns 6`，確認有用到 task-report。

驗證：`git log` 有 smart-commit 產生的 commit；報表有數字。

## 第 4 章 Sub-agents 與任務委派（k96-ch04-practice）

1. 把 taskbox 複製到本章資料夾。
2. 提示詞：

> 在本專案 `.claude/agents/` 建三個 sub-agent：
>
> - `code-reader`（唯讀模式）：tools 只給 Read、Grep、Glob，找出 taskbox 的潛在 bug，只回報不修改
> - `test-runner`（執行模式）：負責跑 pytest、只回傳失敗摘要（把大量輸出擋在主對話外）
> - `doc-writer`：依 code-reader 的報告寫 `REVIEW.md`
> 然後示範兩種模式：(a) 平行：同時派兩個 code-reader 分別看 `taskbox.py` 與 `tests/`；(b) 管線：code-reader → 修正 → test-runner → doc-writer。最後寫 `SUBAGENTS-NOTES.md`：每個 agent 對應的設計模式（單一職責、艙壁、MapReduce、責任鏈），以及這次主對話省下多少 context（概估）。

驗證：`REVIEW.md` 存在；修正後 pytest 通過。

## 第 5 章 Hooks：從準則到護欄（k96-ch05-practice，isolated）

1. 把 taskbox 複製到本章資料夾。本章練習視窗用 `--setting-sources project,local`，排除全域 hook 干擾。
2. 提示詞：

> 用 Python 寫 hook 腳本放在 `.claude/hooks/`，設定寫在本專案 `.claude/settings.json`（不可動全域）：
>
> 1. PreToolUse（Bash）：攔截危險指令（`rm -rf`、`git push --force`、`curl ... | sh`），回 exit 2 並說明原因
> 2. PreToolUse（Edit／Write）：保護 `.env`、`secrets/` 不可被改
> 3. PostToolUse：把每次工具呼叫寫進 `.claude/audit.jsonl`（時間、工具、參數摘要）
> 4. PostToolUse（Edit／Write 的 .py 檔）：自動跑 `python -m py_compile` 做語法檢查，失敗時用 hookSpecificOutput 把錯誤回饋給 Claude
> 5. Stop：跑 pytest，失敗就擋下並要求修好
> 寫 `HOOKS-NOTES.md`：事件生命週期、能不能擋（exit 2）、command／prompt／agent 三種處理器的差別、六個設定位置。
> 驗證：用 `claude -p "執行 rm -rf build 清掉暫存" --max-turns 3` 確認被擋；用 `claude -p "在 .env 加一行 A=1" --max-turns 3` 確認被擋；看 `audit.jsonl` 有紀錄。

驗證：兩個攔截都生效；稽核檔有內容；全域 settings.json 與備份一致。

## 第 6 章 MCP：連結外部工具（k96-ch06-practice）

1. 本章不用 taskbox，直接新建。
2. 提示詞：

> 用 uv 建專案，裝官方 `mcp` Python 套件，寫一個自訂 MCP server `taskdb_server.py`（FastMCP，stdio）：底層用 SQLite `tasks.db`，提供 `add_task`、`list_tasks`、`complete_task` 三個工具和一個 `stats` resource。輸出要控制大小（list 預設最多 20 筆）。
> 用 `claude mcp add --scope project` 註冊成專案層級的 `.mcp.json`（不可寫到使用者層級）。
> 驗證：`claude mcp list` 看得到；`claude -p "用 taskdb 新增三個待辦並列出來" --max-turns 6` 真的寫進資料庫（用 sqlite3 查證）。
> 寫 `MCP-NOTES.md`：N×M→N+M、client-server／JSON-RPC、三種傳輸方式、設定檔位置與範圍、三層安全防護，以及 MCP（廚房）＋Skill（食譜）怎麼搭配。

驗證：資料庫裡有 Claude 透過 MCP 新增的資料；`.mcp.json` 只在專案內。

## 第 7 章 Headless 模式與 CI/CD（k96-ch07-practice）

1. 把 taskbox 複製到本章資料夾，`git init` 並 commit，再故意改壞一個小地方（不 commit）。
2. 提示詞：

> 在本機示範 headless 模式（不建 GitHub repo、不 push）：
>
> 1. 三種輸出格式：text／json／stream-json 各跑一次，從 json 取出 `total_cost_usd`、`num_turns`、`session_id`
> 2. 成本與安全護欄：`--max-turns`、`--allowedTools`、`--disallowedTools`、`--permission-mode`
> 3. 結構化輸出：用 `--json-schema` 讓審查結果固定成 `{issues:[{file,line,severity,message}]}`
> 4. Unix 管線：`git diff | claude -p "審查這段變更" ...`
> 5. session：`--resume <session_id>` 接續追問一次
> 6. 寫 `ci/review.sh`：多階段（lint → 測試 → Claude 審查 → 依 severity 決定 exit code），在本機跑一次
> 7. 只寫檔不執行：`.github/workflows/claude-review.yml`（官方 action 範本＋直接呼叫 CLI 範本兩種），並在檔頭註明本次只示範、未啟用
> 寫 `HEADLESS-NOTES.md` 記錄每一步實際輸出的重點數字。

驗證：`ci/review.sh` 在本機跑出結果並以 exit code 反映問題；json 輸出有成本欄位。

## 第 8 章 Agent SDK（k96-ch08-practice）

1. 本章新建。確認環境沒有 `ANTHROPIC_API_KEY`（SDK 會改用已登入的 Claude Code）。
2. 提示詞：

> 用 uv 建專案，安裝 `claude-agent-sdk`（Python）。寫 `analyzer.py`，示範：
>
> 1. `query()` 基本呼叫，逐一印出 system／assistant／user／result 四類訊息
> 2. `ClaudeAgentOptions`：限定 allowed_tools、permission_mode、max_turns、system_prompt
> 3. 自訂工具：用 `@tool` 定義 `count_lines(path)`，透過 SDK 內建 MCP server 掛上去
> 4. SDK hooks：PreToolUse 擋掉 Bash；PostToolUse 記錄工具耗時
> 5. 結構化輸出：要求結果符合 JSON schema（檔案數、總行數、最長檔案、建議）
> 6. 用 `ClaudeSDKClient` 做兩輪對話，第二輪接續第一輪的 context
> 拿 `k96-ch01-practice` 當分析對象跑一次，把輸出存成 `sample-output.json`。寫 `SDK-NOTES.md`：Agent SDK 和 headless CLI 怎麼選。
> 若 SDK 因認證失敗無法執行，不要設 API key，改寫進筆記並保留程式碼。

驗證：`sample-output.json` 符合 schema；Bash 被 hook 擋掉有紀錄。

## 第 9 章 Plugin 生態系（k96-ch09-practice）

1. 本章新建。
2. 提示詞：

> 做一個團隊能力包 plugin `team-kit`：
>
> - `.claude-plugin/plugin.json`（name、version、description、author）
> - `commands/review.md`（斜線指令）、`agents/security-scanner.md`（唯讀 sub-agent）、`skills/release-notes/SKILL.md`
> - `hooks/hooks.json`＋`hooks/check_secrets.py`（PreToolUse 擋寫入疑似金鑰）
> - 另建本機 marketplace：`marketplace/.claude-plugin/marketplace.json` 指向 team-kit
> 不可安裝到使用者層級（不可改 `~/.claude/settings.json`）；改用 `claude --plugin-dir ./team-kit -p "/team-kit:review" --max-turns 5` 在本次執行載入測試，並跑 `claude plugin validate`（若此版本有）。
> 寫 `PLUGIN-NOTES.md`：命名空間、安裝來源、私有 marketplace、設計原則（單一職責、漸進迭代、最小權限、文件）、何時該打包成 plugin。

驗證：`--plugin-dir` 載入後斜線指令可用；全域 settings.json 與備份一致。

## 第 10 章 從個人到團隊（k96-ch10-practice）

1. 把 taskbox 複製到本章資料夾。
2. 提示詞：

> 把 taskbox 整理成團隊可共用的設定：
>
> 1. `.claude/settings.json`（進版控）：permissions 的 allow／deny（deny 讀 `.env`、`secrets/**`），預設模型與 permission mode；`.claude/settings.local.json` 放個人設定並 gitignore
> 2. 階層式 CLAUDE.md：根目錄放共通規則，`tests/CLAUDE.md` 放測試規則
> 3. 成本：用 `claude -p ... --output-format json` 跑同一題兩次（換模型），比較 `total_cost_usd` 與 token
> 4. 除錯：用 `--debug` 跑一次，摘出載入了哪些設定與記憶
> 5. 寫 `TEAM-PLAYBOOK.md`：成本策略、除錯三招、最小權限清單、MCP 信任評估表、大型程式庫導航、團隊導入路線圖與新人訓練、全書九大機制總覽表
> 驗證：`claude -p "讀 .env 的內容" --max-turns 3` 被權限擋下。

驗證：權限 deny 生效；playbook 涵蓋九大機制。

## 略過項目

- 前言與結語、Index：無操作內容。
- 第 7 章 GitLab／Jenkins：只在 notes 裡寫範本對照，不執行。
- 第 9 章私有 marketplace 的組織層級管理：需要組織帳號，只寫筆記。
