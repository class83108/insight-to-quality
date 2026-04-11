# Goals — Expense Tracker CLI

## System Purpose

一個供個人使用的命令列記帳工具，解決「用記事本記帳容易忘記或找不到」的問題，讓使用者快速新增支出紀錄並依分類或日期查詢。

## Functional Goals

- **G1** [STABLE]: 系統 SHALL 允許使用者新增一筆支出紀錄，包含金額、分類、描述和日期（日期預設為今天）
- **G2** [STABLE]: 系統 SHALL 允許使用者列出支出紀錄，可依分類、日期區間或兩者組合過濾
- **G3** [STABLE]: 系統 SHALL 允許使用者自訂並管理支出分類（新增、列出），不使用固定列表
- **G4** [STABLE]: 系統 SHALL 提供依分類的支出匯總，顯示各分類在指定期間的總額

## Non-Goals

- **NG1**: 不支援多使用者 — 這是單人個人工具，帳號系統帶來不必要的複雜度
- **NG2**: 不支援多貨幣 — 只記台幣，貨幣換算超出工具定位
- **NG3**: 不支援匯出至試算表或 CSV — 個人使用不需要整合外部工具
- **NG4**: 不支援預算設定或超支提醒 — 屬於不同功能領域
- **NG5**: 不提供圖表或視覺化介面 — 純文字輸出即可

## Non-Functional Requirements

- **NFR1** [EVOLVING]: 從輸入指令到顯示結果的時間 < 1 秒（用 `time expense list` 量測）
- **NFR2** [EVOLVING]: 一年使用量（估算 3650 筆紀錄）的資料檔案大小 < 5 MB

## Constraints

- **C1**: 使用 Python 實作（使用者技術偏好）
- **C2**: CLI 介面，無 GUI 需求
- **C3**: 資料儲存在本機，無雲端同步需求

## Open Questions

- **OQ1**: 資料儲存格式尚未決定 — JSON 檔案 vs. SQLite，影響查詢效能與可攜性，需在 ec:align-internals 討論
- **OQ2**: 使用者假設自己會用到查詢功能；若實際使用後發現只用新增，G2/G4 的優先序需重新評估
