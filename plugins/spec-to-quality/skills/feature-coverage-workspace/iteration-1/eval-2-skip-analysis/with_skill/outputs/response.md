需要先做覆蓋率分析，確認要覆蓋哪些類別的 scenario，才能開始撰寫 .feature 檔。

先收集上下文再產出分析表。

---

## 匯率轉換 Feature 覆蓋率分析

在開始撰寫 .feature 之前，先對 6 類 scenario 類別逐一判斷：

| # | 類別 | 適用？ | 具體 Scenario 方向 | 不適用原因 |
|---|------|--------|-------------------|-----------|
| 1 | **Happy path** | ✅ 適用 | 給定來源幣別、目標幣別與金額，回傳正確換算結果 | |
| 2 | **Error / Failure paths** | ✅ 適用 | 外部匯率 API 無法連線、API 回傳非預期格式、匯率資料過期時的 fallback 行為 | |
| 3 | **Input boundary** | ✅ 適用 | 金額為 0、金額為負數、金額超大（overflow）、不支援的幣別代碼、格式錯誤的幣別（小寫、空白） | |
| 4 | **Edge cases** | ✅ 適用 | 來源與目標幣別相同（USD→USD）、金額為小數點極小值（精度問題） | |
| 5 | **State mutation** | 待討論 | 若有匯率 cache（DB 或 in-memory），寫入與讀取是否正確、cache 過期後是否重新拉取 | 若為純函式無副作用則不適用，但需確認是否有 cache 機制 |
| 6 | **Output contract** | ✅ 適用 | 回傳物件包含換算後金額、使用的匯率值、匯率時間戳記；欄位型態與 schema 符合 | |

---

### 需要你確認的問題

1. **State mutation（第 5 類）**：匯率轉換是否有 cache 機制（例如 Redis、in-memory、DB）？
   - 有 → 需要加入 cache 寫入/讀取/過期的 scenario
   - 無（純函式每次呼叫外部 API）→ 標記為不適用

2. 外部匯率 API 失敗時，是否有預期的 fallback 行為（例如使用上一次快取的匯率）？還是直接拋出錯誤？這會影響 Error paths 的 scenario 方向。

3. 是否有既有的同類型 .feature 檔可以作為一致性參考基準？

**在你確認以上問題後，才會開始撰寫 .feature 檔。**
