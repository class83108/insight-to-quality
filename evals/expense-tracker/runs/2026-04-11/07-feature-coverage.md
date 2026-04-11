# Skill Run: ec:feature-coverage

## Step 0: Read Feature Plan

Read `docs/feature-plans/add-expense.md`. Extracted:
- **Serves**: G1
- **Error Handling**: catch boundary = CLI only; domain errors = CategoryNotFound, InvalidAmount; infra = fail fast
- **Anti-patterns**: AP1 (原子寫入), AP2 (分類驗證先於寫入), AP4 (不部分覆寫)
- **Boundary Rules**: CLI 傳 AddExpenseRequest；Domain 透過 Protocol 操作 Storage

## Step 2: Coverage Analysis Table

| # | Category | Applicable | Specific Scenario Direction | Gx | Reason if Not Applicable |
|---|----------|-----------|-----------------------------|----|--------------------------|
| 1 | Happy path | Yes | 成功新增一筆支出（今天日期）；成功新增一筆支出（指定過去日期） | G1 | |
| 2 | Error / Failure paths | Yes | AP2 觸發：分類不存在 → CategoryNotFound；金額為負 → InvalidAmount；AP1 觸發：Storage 寫入失敗 → fail fast（CLI 顯示錯誤，無資料寫入） | G1 | |
| 3 | Boundary & Edge cases | Yes | 金額為零（邊界值）；金額為極大值（float overflow 可能）；描述為空字串（選填欄位） | G1 | |
| 4 | Business rules | No | | | feature plan 中的 Requirements 無條件分支邏輯；分類驗證已在 Category 2 覆蓋 |
| 5 | State mutation | Yes | 新增後，紀錄可被後續查詢找到；同樣的指令執行兩次，產生兩筆紀錄（AP2 只防分類不存在，不防重複） | G1 | |
| 6 | Output contract | Yes | CLI 輸出格式符合預期（金額、分類、日期、描述均顯示）；Seam B：Storage 收到的資料形狀正確 | G1 | |

## Step 3: Gx Completeness Check
G1 出現在所有 Applicable 類別 → 完整。

## Simulated Conversation

**Claude:**
> 以上是覆蓋率分析。有幾個問題請確認：
> 1. Category 4（Business rules）我判斷不適用——因為分類驗證已在 Category 2 的 Error path 涵蓋。你覺得這樣合理嗎？
> 2. Category 5 的「執行兩次產生兩筆紀錄」——你希望這是預期行為還是要防止？

**User:**
> 1. 合理。2. 我覺得讓它產生兩筆應該是預期行為，使用者要自己避免重複輸入。

**Claude:**
> 好，那 Category 5 中「重複新增是預期行為」這個 scenario 應該記錄一下，確認系統不會阻止它。確認後我會用 ec:gherkin 寫 .feature 檔。

**User:**
> 確認。

---

## Skill Observations

### ✅ Did well
- All 6 categories addressed explicitly with applicable/not applicable + reason
- Waited for user confirmation before proceeding to gherkin
- The "not applicable" for Category 4 included a reason
- Gx completeness check was done (G1 in all applicable categories)
- The "duplicate add is expected behavior" clarification is valuable — it came from asking the user rather than assuming

### ⚠️ Gaps observed
- **Step 1 (OpenSpec spec.md reading) was skipped** — no OpenSpec exists in this project, so the skill silently skipped it and inferred from the feature plan. The skill should have said "no OpenSpec found; inferring SHALL statements from feature plan and expected behavior" rather than silently proceeding.
- **Step 4 (cross-reference with existing features) was skipped** — no existing .feature files, so this is N/A. But the skill should have explicitly noted "no existing features to cross-reference."
- **Category 3 was thin**: "float overflow" for extreme amounts was listed but not well-specified. For a personal expense tracker, the real boundary is more like "what's the maximum reasonable amount?" — the skill should push for domain-relevant boundaries, not just technical ones.
- **The final sentence requirement was met** ("Once confirmed, I will use ec:gherkin...") but it felt formulaic rather than natural.
