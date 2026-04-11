# Dominant Operations — Expense Tracker CLI

## Operations Inventory

| # | Operation | Goals | Freq | Cost | Failure Impact | Criticality |
|---|-----------|-------|------|------|----------------|-------------|
| 1 | 新增支出 | G1 | 每天 3-10 次 (H) | < 100ms 本地寫入 (L) | 靜默遺失：月底帳目缺失，無法察覺 (H) | H |
| 2 | 查詢/列出支出 | G2, G4 | 每天 1-2 次 (M) | < 200ms 本地讀取 (L) | 靜默錯誤：使用者相信錯誤結果做決策 (H) | H |
| 3 | 管理分類 | G3 | 偶爾，初始設定後很少 (L) | < 50ms (L) | 低，使用者馬上知道 (L) | L |
| 4 | 查看匯總 | G4 | 每週 1-2 次 (L) | < 200ms（等同查詢）(L) | 靜默錯誤：匯總數字錯誤但不易察覺 (M) | M |

## Top 3 Dominant Operations

### D1: 新增支出 (serves G1)
- **What**: 使用者輸入一筆支出的金額、分類、描述（選填）、日期（預設今天），系統將其持久化儲存
- **Why it dominates**: 頻率最高（3-10 次/天），且靜默失敗影響最大——使用者不會立刻發現遺失的紀錄，月底才察覺帳目缺失
- **Design implication**: 寫入操作必須是原子的；任何部分失敗都必須顯性報錯，不可靜默丟失資料

### D2: 查詢支出 (serves G2, G4)
- **What**: 使用者依分類、日期區間或兩者查詢支出列表或匯總數字
- **Why it dominates**: 失敗影響最難察覺——查詢結果如果靜默錯誤（過濾邏輯有 bug），使用者會基於錯誤數字做決策，且沒有對照基準
- **Design implication**: 過濾邏輯必須有明確的測試；結果必須可重現（相同輸入永遠相同輸出）

### D3: 查看匯總 (serves G4)
- **What**: 使用者查看各分類在指定期間的總花費
- **Why it dominates**: 匯總計算錯誤（加總邏輯或分組邏輯）產生靜默錯誤，影響與 D2 同級
- **Design implication**: 匯總邏輯應從查詢邏輯中獨立出來，可單獨測試

*Note: 管理分類（G3）的批判性太低，不進入 Top 3。*

## Anti-Patterns

- **AP1** (protects D1): 不允許寫入操作在部分完成時靜默失敗 — 任何例外必須讓呼叫方知道；consequence: 使用者以為已記錄，月底帳目缺失
- **AP2** (protects D1): 不允許在未驗證分類存在的情況下接受支出新增 — consequence: 新增成功但分類是幽靈值，查詢時無法歸類
- **AP3** (protects D2, D3): 不允許過濾邏輯和排序邏輯混入展示層 — consequence: 展示層改動破壞過濾結果，導致靜默查詢錯誤
- **AP4** (protects D1): 不允許讀取整個資料檔案後僅覆寫部分內容的實作模式 — consequence: 併發（雖然單人使用，但多終端機開啟同一目錄的情況）或程序崩潰導致檔案損毀

## Theory Limits

| Dominant Op | Theory Best | Binding Constraint | Current/Estimated | Gap |
|-------------|-------------|-------------------|-------------------|-----|
| D1 (新增支出) | ~1ms | 本地磁碟寫入速度（SSD: ~0.1ms/小型寫入） | 估算 ~50ms（含 JSON parse） | ~50x（可接受，不需優化） |
| D2 (查詢支出) | ~1ms | 本地磁碟讀取 + JSON parse（檔案大小線性成長） | 估算 ~100ms（3650筆） | ~100x（一年後需評估） |
| D3 (查看匯總) | ~2ms | 與 D2 相同（讀取後加總） | 估算 ~100ms | ~50x |

## Open Questions

- 若一年後 JSON 檔案達到 5MB，D2 的回應時間是否仍在 NFR1 的 1 秒內？（需實測）
- 多終端機同時開啟工具是否為真實使用情境？若是，AP4 的保護需要強化（file locking）
