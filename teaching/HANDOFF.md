# K96 演練交接

## 原機器結果

本書沒有在其他機器演練過，沒有可比對的原機器結果。每章的差異以「和書上原做法的差異」記錄。

## 快速模式進度表

| 章 | 完成日 | 實際耗時 | 和書上做法的差異 |
| --- | --- | --- | --- |
| 01 | 2026-10-17 | 約 8 分（預估 12） | 書上無隨書程式碼，taskbox 依課表自寫；stream-json 中有 16 則全域 SessionStart hook 事件（書上乾淨環境沒有），正好示範「事件」觸發；Windows 跑 pytest 需 PYTHONUTF8=1 |
| 02 | 2026-10-17 | 約 4 分（預估 10） | CLAUDE.md 49 行；claude -p 答對測試指令與命名規則，但因 CLAUDE.md 有一行指向 .claude/rules/tests.md，規則是被「順著讀到」而非依 paths 條件載入——要嚴格示範條件載入需移除該指引行 |
| 03 | 2026-10-17 | 約 5 分（預估 15） | 三個 skill 皆驗證觸發：/smart-commit cli 產生 docs(cli) commit、完成率問句觸發 task-report（40.0%）；全域 Stop hook 每次 claude -p 結束會自動 commit「更新 session-state」，git log 多 4 筆雜訊（書上乾淨環境沒有） |
| 04 | 2026-10-17 | 約 7 分（預估 15） | agent 檔於同 session 中途建立，Agent 工具尚未載入，改用 general-purpose＋角色提示詞（tools 限制只是約定、非系統強制）；平行＋管線兩模式都跑完，pytest 13 passed；子 agent 合計 ~425k tokens、主對話只收 ~4k，但每個子 agent 固定載入成本 ~9 萬 tokens，總成本變高 |
| 05 | 2026-10-17 | 約 8 分（預估 15） | isolated（--setting-sources project,local）開窗時跳「allow external imports」確認，看門狗刻意不代按，由教練選 Yes（視窗名稱要確認後才生效）；5 個 hook 都寫在專案 .claude/settings.json，rm -rf 與寫 .env 皆被擋、audit.jsonl 13 筆；練習 Claude 被自己的 Bash hook 擋建 .env 後改用 PowerShell 繞過，正好示範 matcher 範圍；全域 settings.json 被他程序轉成 LF（JSON 內容相同），已從備份還原 |
| 06 | 2026-10-17 | 約 5 分（預估 15） | FastMCP＋SQLite server 以 --scope project 註冊 .mcp.json，專案 settings.local.json 用 enabledMcpjsonServers 核准；claude -p 經 MCP 寫入 3 筆（sqlite3 查證）；~/.claude.json 無 taskdb；claude mcp add 產生的 .mcp.json 含本機絕對路徑（練習資料夾保留，筆記附 ${VAR} 可攜寫法） |
| 07 | 2026-10-17 | 約 11 分（預估 15） | text/json/stream-json、護欄、--json-schema、管線、--resume 全在本機示範；ci/review.sh 實跑 exit 2（抓到故意改壞的第 68 行為 critical、pytest 2 failed）；只給 diff 時模型誤判「沒測試會抓到」，故 review.sh 改為把 pytest 結果一併交給 Claude；yml 只寫不啟用（if: false）；全域 Stop hook 對故意保留的未 commit 改動發提醒 |
| 08 | 2026-10-17 | 約 7 分（預估 15） | 未設 ANTHROPIC_API_KEY，SDK 直接走已登入訂閱成功；sample-output.json 符合 schema；Bash 刻意放進 allowed_tools 以證明是 PreToolUse hook（deny）擋下，共擋 2 次、模型改用 Glob；模型起初自行排除 .md 致最長檔案判錯，提示詞講清楚範圍後正確 |
| 09 | 2026-10-17 | 約 8 分（預估 12） | 未安裝到使用者層級（known_marketplaces／installed_plugins 皆無 team-kit），改用 --plugin-dir 單次載入，/team-kit:review 與金鑰 hook 皆生效；validate --strict 通過；marketplace 不接受 ../team-kit，改用 Windows junction（換機需重建、實際安裝未驗證）；用 plugin 審查自己抓出 3 個 hook 漏洞與 python3||python 放行陷阱並修正 |
| 10 | 2026-10-17 | 約 14 分（預估 12） | 專案 settings.json（allow 9／deny 11）＋settings.local.json gitignore＋階層式 CLAUDE.md；實測 Read(./.env) deny 擋不住 python -c open(.env)，補 block_secrets hook 才擋住；haiku 只載入專案設定時成本 $0.0182→$0.0034（全域 149 工具＋159 skill 約 8.3 萬 token 背景）；-p 模式 --debug 不印畫面需改 --debug-file；settings 寫 defaultMode default 但實際為 auto（未查證原因） |

**全部 10 章完成：2026-10-17 05:01–06:21，約 80 分鐘（課表預估 136 分鐘）。** 全域 `~/.claude/settings.json` 每章比對；第 05 章曾被他程序轉成 LF（內容相同）已還原，其餘一致。
