# K96《The Claude Code Operating Model》快速演練教材

- 原書：Jia Huang（Packt, 2026-08），ASIN `B0H947QRWW`，10 章＋前言／結語
- 原書擷取與 NotebookLM 產物：家用機 `~/Downloads/AutoRead-GoogleBook/kindle-96-claude-code-operating-model/`（PDF、截圖不入版控）
- 語音導讀：<http://187.127.109.145/kindle-96.m3u>（12 段）
- 簡報網頁：<https://chenghyang2001.github.io/kindle-96-slides/>

## 演練（kindle-drill 雙視窗快速模式）

| 檔案 | 用途 |
| --- | --- |
| `teaching/LESSON-PLAN.md` | 10 章的練習步驟與提示詞（依原書目錄自行撰寫，書上沒有隨書程式碼） |
| `teaching/drill.json` | 章節 → 練習資料夾 `~/workspace/k96-chNN-practice` |
| `teaching/DRILL-PROMPT.md` | 教練開場提示詞 |
| `teaching/HANDOFF.md` | 快速模式進度表 |

範圍限制：所有 hooks／MCP／plugin／權限設定只寫在練習專案裡，不改全域 `~/.claude`；CI 只在本機示範，不建 repo、不跑 GitHub Actions；Agent SDK 走已登入的 Claude Code 訂閱，不設 `ANTHROPIC_API_KEY`。
