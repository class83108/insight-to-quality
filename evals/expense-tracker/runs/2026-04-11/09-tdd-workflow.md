# Skill Run: ec:tdd-workflow

## Phase 0: Verification Ledger

**Claude (follows skill — reads feature plan first):**
> 讀完 `docs/feature-plans/add-expense.md`。在寫任何測試前，先建立 Verification Ledger。
>
> 從 .feature 檔推斷 SHALL 清單（無 OpenSpec）：
> - SHALL-1: 新增支出時，金額、分類、描述、日期必須被持久化
> - SHALL-2: 分類不存在時，系統必須拒絕新增並回傳 CategoryNotFound（AP2）
> - SHALL-3: 金額 ≤ 0 時，系統必須拒絕並回傳 InvalidAmount
> - SHALL-4: Storage 寫入失敗時，系統必須 fail fast，不留部分資料（AP1）
> - SHALL-5: 成功新增後，紀錄可透過查詢取得
> - SHALL-6: 回傳資料的 schema 必須包含 amount、category、description、date 欄位（Output contract）

**Verification Ledger:**

```
# Verification Ledger — add-expense

## Error Handling Strategy (from feature plan)
- Catch boundary: CLI layer only
- Domain errors: CategoryNotFound, InvalidAmount
- Infrastructure errors: fail fast

## Unit Test Coverage
- SHALL-1 → mock boundary: mock ExpenseRepository.add()；驗證 domain 正確建構 Expense 物件並呼叫 repository
- SHALL-2 → mock boundary: mock CategoryRepository.exists()，讓它回傳 False；驗證 domain 在 add 前先查分類
- SHALL-3 → 無需 mock；驗證 domain 的輸入驗證邏輯（純計算）
- SHALL-5 → mock boundary: mock ExpenseRepository.add()；mock ExpenseRepository.list()；驗證新增後 list 可取得紀錄
- SHALL-6 → mock boundary: mock ExpenseRepository.add()；驗證回傳的 Expense object 有正確欄位

## Needs Integration Test
- SHALL-4: Storage 寫入失敗時 fail fast
  - Reason: 需要真實的 Storage 實作（JSON file 或 SQLite）才能驗證「部分寫入」行為
  - Minimum verification: 寫一個 smoke test 用真實 Storage，模擬磁碟滿或權限不足的情況

## Explicitly Not Tested
- (none)
```

**Claude:**
> 請確認 Ledger 的 mock 邊界和 tier 分配是否正確。特別是 SHALL-4——你希望現在寫 integration test，還是先追蹤後補？

**User:**
> 先追蹤後補，目前先跑 unit tests。

**Claude:**
> 好，我把 SHALL-4 記錄到 `docs/feature-plans/add-expense.md` 的 Integration Test Gaps，然後開始寫測試。

---

## Phase 1: Red (Summary)

*(Actual Python code omitted from eval record — this section describes what would happen)*

Claude would:
1. Scan `step_defs/` (empty) and `conftest.py` (doesn't exist yet)
2. Create `tests/step_defs/add_expense_steps.py`
3. Write step definitions organized by scenario order (not by step type)
4. Use `MagicMock(spec=ExpenseRepository)` for the repository mock
5. Run `pytest` → all tests FAIL (ImportError / NotImplementedError)
6. Show red output and wait for user confirmation

---

## Phase 2: Green (Summary)

Claude would implement minimally:
1. `src/expense_tracker/protocols.py` — ExpenseRepository Protocol
2. `src/expense_tracker/domain.py` — ExpenseService.add_expense()
3. Run pytest after each scenario, not all at once

---

## Skill Observations

### ✅ Did well
- Verification Ledger was produced before any test code
- Mock boundary is correctly placed at Storage layer (ExpenseRepository), not inside Domain logic
- SHALL-4 correctly identified as needing integration test (it requires real Storage behavior)
- Skill waited for user confirmation of ledger before proceeding
- MagicMock with spec= is specified (rubric item TDD7)
- Step definitions organized by scenario order would be instructed (not by step type)

### ⚠️ Gaps observed
- **Phase 1 Red confirmation was ambiguous in the skill.** The skill says "Wait for user confirmation: 'Red confirmed. Ready to start implementation?'" — but it doesn't specify what format the red output should be shown in. A real agent might show a truncated output or just say "all failed" without showing the actual error messages.
- **Checkpoint A and B were not explicitly named** in the ledger review. The skill defines Checkpoint A (mock conceals external behavior, not own logic) and Checkpoint B (cross-component shape match). These should appear explicitly in the ledger document so the user can audit them.
- **The "scan existing step_defs" step** (Phase 1, step 1) is important for larger projects but creates noise for new projects. The skill doesn't distinguish "first feature in project" from "Nth feature" — the scanning instruction should have a note for when the directory is empty.
- **Coverage analysis → ledger traceability gap**: The ledger's SHALL statements were inferred from the .feature file. But SHALL-5 ("新增後可查詢") appears in the .feature as a Scenario, not as an explicit SHALL. The skill should note that when there's no OpenSpec, the ledger SHALLs are inferred from Then steps — but this inference can miss behavioral requirements that weren't captured in scenarios.
