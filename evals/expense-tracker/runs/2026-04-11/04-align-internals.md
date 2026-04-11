# Skill Run: ec:align-internals

## Mode: Design Mode (no existing code)

## Simulated Conversation

**Claude (following skill):**
> 我們進入合約設計。這是設計模式（沒有現成程式碼）。
> 
> 先從 Seam B（Domain ↔ Storage）開始，因為 AP1 告訴我們這裡風險最高。Storage 需要提供哪些操作給 Domain？

**User:**
> 就是新增支出、查詢支出（可以過濾）、管理分類（新增、列出）。

**Claude:**
> Storage 如果寫入失敗，Domain 應該收到什麼？

**User:**
> 收到錯誤吧？然後告訴使用者失敗了。

**Claude:**
> 好。那 Storage 的合約應該是：成功時回傳結果，失敗時 raise exception。這條 exception 要有名字嗎？

**User:**
> 有比較好，這樣可以分清楚是 Domain 邏輯失敗還是 Storage 失敗。

**Claude:**
> 現在 Seam A（CLI ↔ Domain）。CLI 傳給 Domain 的是「raw 字串」還是已解析的結構？

**User:**
> Domain 那邊應該收到已經解析過的，CLI 負責 parse。

**Claude:**
> 所以 CLI 解析完之後，傳給 Domain 的應該是什麼形狀的資料？

**User:**
> 就是金額（數字）、分類名稱（字串）、描述、日期之類的。

**Claude:**
> 這就是一個 `AddExpenseRequest`。Domain 回傳什麼？

**User:**
> 成功的話回傳新增的紀錄？或者只回傳 OK 也可以。

**Claude:**
> 如果 CLI 需要顯示「已新增：120 元，餐廳，2026-04-11」，它需要拿到紀錄的細節。所以回傳 `Expense` 結果比較好。

**Claude (idempotency check — following skill):**
> D1（新增支出）有沒有需要保證冪等性的情境？比如使用者按了兩次？

**User:**
> 我沒有特別考慮過，應該不需要，就讓使用者重複輸入就好。

*→ Skill correctly flags this as a gap to watch: AP2 mentions duplicate detection, so idempotency might matter.*

---

## Skill Observations

### ✅ Did well
- Correctly started from SYSTEM_MAP Boundary Map (Seam B first, because AP1 identifies it as highest risk)
- Distinguished domain errors from infrastructure errors in the contract
- Asked about data direction at Seam A (CLI passes parsed objects, not raw strings)
- Raised the idempotency question for D1 — this came from the skill's "idempotency: for boundaries supporting retry/re-run" instruction

### ⚠️ Gaps observed
- **OQ1 (storage format) was not resolved here.** The skill should have surfaced "your contract design affects the storage format choice." A Protocol (interface) abstracts this away, but the skill didn't explain *why* an interface is valuable here — it just produced one.
- **Cross-contract validation was partial.** The skill says to check that producer output shape = consumer input shape. Seam A (CLI produces → Domain consumes) was discussed, but the Seam B shape match (Domain produces → Storage consumes) wasn't cross-validated.
- **Persistence design was lightweight.** The skill has a full "Phase 3: Persistence Design" covering lifecycle, query patterns, and indexing. For a CLI tool, these questions are relevant (e.g., "what indexes do you need for date-range filtering?") but were skipped.
