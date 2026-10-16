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
