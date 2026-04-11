# Feature Plan — add-expense

## Source
- Serves: G1
- SYSTEM_MAP gap: G1（新增支出）未實作；同時建立 Seam A 和 Seam B 的合約
- Coverage: Full coverage

## Error Handling Strategy
- Catch boundary: CLI layer only（最外層 catch；Domain 和 Storage 不吸收例外）
- Domain errors: `CategoryNotFound`（AP2：分類不存在）、`InvalidAmount`（金額 <= 0）
- Infrastructure errors: fail fast（Storage 寫入失敗直接 raise，不 retry）

## Constraints

### Anti-patterns (from dominant-ops)
- **AP1** (D1): 不允許寫入操作在部分完成時靜默失敗 — impact: Storage 寫入必須是原子的，任何失敗都要讓 CLI 知道，不可靜默丟失
- **AP2** (D1): 不允許在未驗證分類存在的情況下接受支出新增 — impact: Domain 在寫入前必須先驗證分類存在
- **AP4** (D1): 不允許讀取整個資料檔案後僅覆寫部分內容 — impact: 寫入策略需要原子性（write-then-rename 或 SQLite transaction）

### Boundary Rules (from SYSTEM_MAP)
- CLI 只傳已解析的 `AddExpenseRequest`（非 raw string）給 Domain（Seam A）
- Domain 透過 `ExpenseRepository` Protocol 操作 Storage，不直接碰資料格式（Seam B）
- Storage 可替換（JSON ↔ SQLite），Domain 不應知道實作細節

## Integration Test Gaps
<!-- Filled in after ec:tdd-workflow Verification Ledger is complete; initially empty -->
