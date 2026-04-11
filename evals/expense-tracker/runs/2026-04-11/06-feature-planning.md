# Skill Run: ec:feature-planning

## Simulated Conversation

**Claude (following skill — reads all 4 discovery docs first):**
> 讀完了 goals.md、dominant-ops.md、SYSTEM_MAP.md、align-internals report 和 align-surface report。
>
> 目前 SYSTEM_MAP 的 Gaps 列表：
> - Seam A 合約未實作（CLI ↔ Domain）
> - Seam B 合約未實作（Domain ↔ Storage）
> - G1（新增支出）未實作
> - G2（查詢支出）未實作
> - G3（分類管理）未實作
> - G4（匯總）未實作
>
> `docs/feature-plans/` 目前是空的。我整理出以下候選功能：
>
> | # | Feature | 關閉 Gap | Serves |
> |---|---------|---------|--------|
> | 1 | add-expense（新增支出） | G1 + Seam A + Seam B | G1 |
> | 2 | query-expenses（查詢支出） | G2 | G2 |
> | 3 | expense-summary（匯總） | G4 | G4 |
> | 4 | category-management（分類管理） | G3 | G3 |
>
> *Note: add-expense 應該先做，因為 D1 壓力最高，且 Seam A 和 Seam B 的合約會在這個 feature 中首次建立，後續 feature 都依賴它。*
>
> 你要從哪個開始？

**User:**
> 先做新增支出。

**Claude (Three Decisions — follows skill, uses discovery context):**
>
> **Decision 1 — Catch Boundary**
> `add-expense` 碰到兩個邊界：Seam A（CLI ↔ Domain）和 Seam B（Domain ↔ Storage）。當 Storage 寫入失敗，Domain 應該讓 exception 傳到 CLI，還是在 Domain 內部吸收？

**User:**
> 傳到 CLI，CLI 決定怎麼顯示給使用者。

**Claude:**
> 所以 catch boundary = CLI layer only（最外層）。Domain 不吸收 Storage 例外。
>
> **Decision 2 — Error Taxonomy**
> 根據 AP2（分類不存在時不允許接受支出），`CategoryNotFound` 是一個預期的 domain error。還有什麼 domain error？

**User:**
> 金額是負數或零的話也要報錯。

**Claude:**
> `InvalidAmount`。AP1 告訴我們 Storage 寫入失敗是 infrastructure error（fail fast），不是 domain error。
>
> **Decision 3 — Recovery Strategy**
> Infrastructure error（Storage 失敗）：fail fast 還是 retry？

**User:**
> Fail fast。本地寫入失敗通常不會靠重試解決，可能是磁碟空間問題或權限問題。

---

## Skill Observations

### ✅ Did well
- Read all 4 discovery documents before producing candidate list (prerequisite check passed)
- Candidate list showed Gx mapping for each item
- Asked user to choose instead of auto-selecting
- Three Decisions were all addressed and tied to discovery context (D1, AP1, AP2)
- Correctly identified that `add-expense` should establish Seam A + Seam B contracts (these are needed by subsequent features)

### ⚠️ Gaps observed
- **SYSTEM_MAP Lessons section was not checked.** The skill says to "extract entries related to the boundaries or components this feature will touch." For a new project this is empty, but the skill should have explicitly checked and said "Lessons section is empty — proceeding."
- **SYSTEM_MAP structural update question was skipped.** The skill (Step 3) requires asking whether the gap requires a SYSTEM_MAP structural update (A) or is just staged engineering (B). This question wasn't asked — it was implicitly B.
- **Candidate ordering rationale was volunteered by Claude** rather than asking the user first. The skill says to present candidates and ask "which one first?" — it's OK to note dependencies, but the skill shouldn't pre-determine the order.
- **Feature plan was not explicitly confirmed by user** before the handoff message. The skill (Step 8) says to present the plan for user confirmation on traceability, error handling strategy, and constraints.
