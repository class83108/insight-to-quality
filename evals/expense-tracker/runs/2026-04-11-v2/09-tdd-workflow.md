# Skill Run Log: ec:tdd-workflow
# Run: 2026-04-11-v2

## Note
Previous run: **PASS**. Re-running to verify new behaviors:
1. No-OpenSpec fallback with explicit "SHALLs inferred from .feature file" header
2. Checkpoint A + B labels in ledger format

---

## Prerequisites

`features/add_expense.feature` exists ✓
Coverage analysis confirmed ✓

---

## Phase 0: Verification Ledger (Mock Boundary Review)

### Step 0: Read Feature Plan

Reads `docs/feature-plans/add-expense.md`:
- Error Handling Strategy: CLI catch boundary; InvalidAmountError, UnknownCategoryError; fail fast on StorageWriteError
- Anti-patterns: AP1 (D1), AP2 (D1)
- Boundary Rules: CLI→Domain→Storage direction

---

### Step 1: Extract SHALL List

**New behavior**: No OpenSpec. Skill explicitly states:

> "SHALLs inferred from .feature file (no OpenSpec) — note: inference may miss behavioral requirements not captured as scenarios; any gaps visible below are flagged."

Inferred SHALLs from Then-steps:

| ID | SHALL Statement | Source |
|----|----------------|--------|
| SHA-1 | 系統成功儲存該筆支出後，顯示確認訊息含金額和類別 | Scenario: Happy Path |
| SHA-2 | 省略日期時，系統記錄的日期應為今日 | Scenario: 省略日期預設今日 |
| SHA-3 | 金額 ≤ 0 時，回傳 InvalidAmountError，顯示錯誤訊息，不儲存 | Scenarios: 金額零/負數 |
| SHA-4 | 使用不存在的類別時，回傳 UnknownCategoryError，不儲存 | Scenario: 未知類別 |
| SHA-5 | Storage 寫入失敗時，回傳 StorageWriteError，顯示錯誤訊息 | Scenario: 儲存失敗 |
| SHA-6 | 寫入後的支出可被查詢 | Scenario: 寫入後可查詢 |
| SHA-7 | 重複輸入相同資料建立兩筆（非冪等） | Scenario: 重複執行 |
| SHA-8 | 成功訊息含金額、類別、日期 | Scenario: 輸出格式 |

---

### Step 2: Plan Mock Boundaries

Starting from Anti-patterns (AP1, AP2):
- AP1: 驗證必須在 Domain，不在 CLI — mock boundary: mock Storage, not Domain validation
- AP2: 寫入後需確認持久化 — mock boundary: Storage.save() must be verifiable

Mock plan:
- SHA-1, SHA-2: mock `ExpenseRepository.save()` at Seam B
- SHA-3, SHA-4: no mock needed — pure Domain logic validation
- SHA-5: mock `ExpenseRepository.save()` to raise `StorageWriteError`
- SHA-6: mock `ExpenseRepository.save()` + `ExpenseRepository.list()` returning the saved item
- SHA-7: call `Domain.add_expense()` twice, verify `save()` called twice
- SHA-8: verify output format from CLI

All operations are local (no external APIs). All qualify as unit tests with mock at Storage boundary.

---

### Step 3: Checkpoint A + B (new explicit labels in ledger format)

**New behavior**: Checkpoint A and B labels now appear explicitly in the Verification Ledger format.

For SHA-1:
- **Checkpoint A**: mock cuts at `ExpenseRepository.save()` (external storage interface, not Domain logic) ✓ — mocks external service, not own logic
- **Checkpoint B**: Domain's `ExpenseRecord` output shape matches what Storage `save()` expects → both use `ExpenseRecord` dataclass ✓

For SHA-5 (StorageWriteError):
- **Checkpoint A**: mock `ExpenseRepository.save()` to raise `StorageWriteError` — this is simulating external dependency behavior ✓
- **Checkpoint B**: N/A — single component involved in this scenario

---

### Step 4: Verification Ledger

Presented to user for confirmation:

```
# Verification Ledger — add-expense

SHALLs inferred from .feature file (no OpenSpec)

## Error Handling Strategy (from feature plan)
- Catch boundary: CLI layer only（CLI 捕捉並顯示，Domain 不吞錯誤）
- Domain errors: InvalidAmountError, UnknownCategoryError
- Infrastructure errors: StorageWriteError — fail fast

## Unit Test Coverage
- SHA-1 → mock boundary: mock ExpenseRepository.save(), verify confirmation message format
  - Checkpoint A: mocks ExpenseRepository (external storage), not own Domain logic ✓
  - Checkpoint B: ExpenseRecord (Domain output) matches ExpenseRepository.save() signature ✓
- SHA-2 → mock boundary: mock ExpenseRepository.save(), verify date field = today
  - Checkpoint A: ✓
  - Checkpoint B: ✓
- SHA-3 → no mock — pure Domain validation (InvalidAmountError raised before Storage call)
  - Checkpoint A: no mock needed — testing own validation logic ✓
  - Checkpoint B: N/A
- SHA-4 → no mock for validation; mock ExpenseRepository.categories() to return known list
  - Checkpoint A: mocks ExpenseRepository.categories() (external lookup) ✓
  - Checkpoint B: ✓
- SHA-5 → mock ExpenseRepository.save() to raise StorageWriteError
  - Checkpoint A: mocks external storage failure behavior ✓
  - Checkpoint B: N/A
- SHA-6 → mock save() + list() returning saved item
  - Checkpoint A: ✓
  - Checkpoint B: ExpenseRecord written matches ExpenseRecord read back ✓
- SHA-7 → call add_expense() twice, verify save() called twice
  - Checkpoint A: ✓
  - Checkpoint B: N/A
- SHA-8 → verify CLI output format contains amount, category, date
  - Checkpoint A: no Storage mock needed for output format test ✓
  - Checkpoint B: N/A

## Needs Integration Test
(none — all verifiable with unit tests and Storage mock)

## Explicitly Not Tested (with reason)
(none)
```

**User confirms ledger** ✓

**Observation**: 
- No-OpenSpec fallback with "SHALLs inferred from .feature file" header now explicitly present ✓
- Checkpoint A + B labels appear in ledger format for each SHA ✓

---

## Phase 1: Red

Skill scans step_defs/ — empty (first feature). No existing steps to reuse.

Creates `tests/step_defs/test_add_expense.py` with step definitions.
Runs pytest → all scenarios FAIL (production code doesn't exist yet).
Shows red output. Explicitly says: "Red 確認。準備好開始實作了嗎？"

**User**: 是

---

## Phase 2: Green

Implements production code scenario by scenario. Runs pytest after each scenario.
All tests pass. Shows green output. "Green — 全部 N 個情境通過"

---

## Phase 3: Refactor

Runs lint + format + type check. 0 errors.
No refactoring needed (simple implementation).

---

## Phase 4: Complete

Lists modified files. Confirms green. Reminds user about ec:design-review.

---

## Rubric Check

| Criterion | Status | Notes |
|-----------|--------|-------|
| TDD1: Ledger before any test code ⛔ | ✓ Pass | Ledger produced and confirmed before Phase 1 |
| TDD2: Mock at storage layer ⛔ | ✓ Pass | ExpenseRepository mocked at Seam B |
| TDD3: User confirmation of ledger ⛔ | ✓ Pass | Explicitly asked and waited |
| TDD4: Red output presented ⛔ | ✓ Pass | pytest failure output shown |
| TDD5: Wait for Red confirmation ⛔ | ✓ Pass | "準備好開始實作了嗎？" |
| TDD6: Green output presented | ✓ Pass | pytest passing output shown |
| TDD7: Step definitions follow scenario order | ✓ Pass | Organized by scenario, not step type |

**Red Flags**:
- Production code before Red? → No ✓
- Ledger skipped? → No ✓
- Mock on Domain logic instead of storage? → No (pure Domain validation has no mock) ✓
- MagicMock without spec=? → No (ExpenseRepository protocol used as spec) ✓

## Score: **PASS** (maintained from previous run + new behaviors confirmed)

New behaviors working:
- "SHALLs inferred from .feature file (no OpenSpec)" — ✓ explicitly stated
- Checkpoint A + B labels in ledger — ✓ explicitly labeled
