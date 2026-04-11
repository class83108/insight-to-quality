# Skill Run: ec:align-surface

## Mode: Design Mode (no existing code)

## Simulated Conversation

**Claude (following skill — starts from user journeys, not endpoint list):**
> 我們從 D1（新增支出）的使用者旅程開始。使用者想新增一筆支出時，他打什麼指令？

**User:**
> 我想像中是類似 `expense add 120 餐廳 --description 午餐` 這樣。

**Claude:**
> 新增後你希望看到什麼回應？

**User:**
> 就顯示一行「已新增：120 元，餐廳，2026-04-11，午餐」這樣。

**Claude:**
> 這個指令的類型是 Trigger（寫入 + 回應確認），錯誤處理：驗證錯誤同步回傳（分類不存在、金額非正數），不需要非同步模式。

**Claude:**
> D2（查詢）的旅程：使用者怎麼過濾？

**User:**
> `expense list --category 餐廳` 或 `expense list --from 2026-04-01 --to 2026-04-30`。

**Claude:**
> 這是 Query 類型：安全重試，結果不暫存。輸出格式你希望是表格還是每筆一行？

**User:**
> 每筆一行就好，不需要花俏的表格。

**Claude:**
> D3（匯總）：

**User:**
> `expense summary --from 2026-04-01` 這樣，按分類顯示總額。

**Claude:**
> 也是 Query 類型。最後，分類管理：

**User:**
> `expense category add 餐廳` 和 `expense category list`。

**Claude (applies non-goals as design tool):**
> 根據 NG3（不匯出 CSV），NG5（不做圖表）——這些排除了任何 `--output json` 或 `--format table` 的選項。CLI 輸出就是純文字，不需要格式切換。你同意這樣限縮嗎？

**User:**
> 對，完全同意。

---

## Skill Observations

### ✅ Did well
- Started from user journeys (D1, D2, D3) not from endpoint list — this is the skill's core value
- Correctly classified each CLI command as Trigger or Query
- Applied non-goals (NG3, NG5) to eliminate design options — exactly what the skill instructs
- Kept the abstraction level right: "expense add 120 餐廳" is interface-level, not implementation-level

### ⚠️ Gaps observed
- **Infrastructure planning was skipped entirely.** The skill has a Phase 3 (Infrastructure Planning) covering process topology, concurrency model, and worker sizing. For a CLI tool this is mostly N/A, but the skill should have explicitly said "this is a single-process CLI, no worker or daemon needed" rather than silently skipping it.
- **Constraint validation (Phase 5) was not done.** The skill says to cross-check against NFR1 (< 1s response). The design should verify that the planned CLI commands (argparse + local file I/O) can meet this threshold. This was implicit but not stated.
- **No mention of the storage format decision** (OQ1 from goals.md). The surface design (CLI commands) is independent of storage format, but the skill should have flagged "your interface design is ready; when OQ1 is resolved, the storage can be plugged in without changing the CLI."
