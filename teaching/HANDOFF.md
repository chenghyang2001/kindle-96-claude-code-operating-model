# K96 演練交接

## 原機器結果

本書沒有在其他機器演練過，沒有可比對的原機器結果。每章的差異以「和書上原做法的差異」記錄。

## 快速模式進度表

| 章 | 完成日 | 實際耗時 | 和書上做法的差異 |
| --- | --- | --- | --- |
| 01 | 2026-10-17 | 約 8 分（預估 12） | 書上無隨書程式碼，taskbox 依課表自寫；stream-json 中有 16 則全域 SessionStart hook 事件（書上乾淨環境沒有），正好示範「事件」觸發；Windows 跑 pytest 需 PYTHONUTF8=1 |
| 02 | 2026-10-17 | 約 4 分（預估 10） | CLAUDE.md 49 行；claude -p 答對測試指令與命名規則，但因 CLAUDE.md 有一行指向 .claude/rules/tests.md，規則是被「順著讀到」而非依 paths 條件載入——要嚴格示範條件載入需移除該指引行 |
| 03 | 2026-10-17 | 約 5 分（預估 15） | 三個 skill 皆驗證觸發：/smart-commit cli 產生 docs(cli) commit、完成率問句觸發 task-report（40.0%）；全域 Stop hook 每次 claude -p 結束會自動 commit「更新 session-state」，git log 多 4 筆雜訊（書上乾淨環境沒有） |
