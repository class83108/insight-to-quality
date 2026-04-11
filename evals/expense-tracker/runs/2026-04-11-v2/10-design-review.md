# Skill Run Log: ec:design-review
# Run: 2026-04-11-v2

## Prerequisites
All tests pass ✓, lint ✓, type check ✓

---

## Phase 0: Collect Declared Design Decisions

Reads `docs/feature-plans/add-expense.md`:
- Error Handling Strategy:
  - Catch boundary: CLI layer only
  - Domain errors: InvalidAmountError, UnknownCategoryError
  - Recovery: fail fast on StorageWriteError

Review scope:
```bash
git diff --name-only HEAD~1
```
→ `src/cli/commands.py`, `src/domain/expense.py`, `src/storage/json_storage.py`, `tests/step_defs/test_add_expense.py`, `features/add_expense.feature`

---

## Part 1: Declared Decisions Verification

### Error Handling Strategy

Reviewing implementation:

**Catch boundary**: 
- `src/cli/commands.py` has `try/except DomainError` at CLI layer ✓
- `src/domain/expense.py` raises `InvalidAmountError` and `UnknownCategoryError` without catching ✓
- `src/storage/json_storage.py` raises `StorageWriteError` without catching ✓

→ Complies ✓

**Domain errors**:
- `InvalidAmountError` raised when `amount <= 0` ✓
- `UnknownCategoryError` raised when category not in repository.categories() ✓

→ Complies ✓

**Recovery strategy**:
- `StorageWriteError` propagates to CLI (caught there and displayed) ✓
- No retry logic found (correct — fail fast) ✓

→ Complies ✓

### Discovery Alignment

- AP1 (D1): validation happens in Domain, not CLI → no CLI validation code found ✓
- AP2 (D1): `json_storage.py` uses `json.dump()` with explicit `flush()` after write ✓
- Seam A direction: CLI imports from Domain, not the other way ✓
- Seam B direction: Domain only references `ExpenseRepository` Protocol, not `JsonStorage` directly ✓

→ Complies ✓

---

## Part 2: Structural Checks

### Separation of Concerns

**`src/domain/expense.py`**:
```python
class ExpenseService:
    def add_expense(self, amount, category, date, note, repo):
        if amount <= 0:
            raise InvalidAmountError(...)
        if category not in repo.categories():
            raise UnknownCategoryError(...)
        record = ExpenseRecord(...)
        repo.save(record)
        return record
```

The `add_expense` method does: validation + category lookup + record creation + save. All four are part of the same business operation (D1). This is acceptable cohesion — not mixing concerns.

No structural question needed here.

### Dependency Direction

**`src/cli/commands.py`**:
```python
from domain.expense import ExpenseService, InvalidAmountError
```
CLI imports Domain. ✓

**`src/domain/expense.py`**:
```python
from storage.protocols import ExpenseRepository  # Protocol import only
```

> Structural question: "Domain imports from `storage.protocols`. Is this the right abstraction level? Should the `ExpenseRepository` Protocol live in the `domain` module rather than `storage`, since it's the Domain that defines what it needs from persistence?"

**Why worth thinking about**: If Protocol lives in `storage/`, then `domain/` still depends on `storage/` structurally. Moving it to `domain/protocols.py` would make the dependency direction clearer: Domain owns the Protocol, Storage implements it.

This is a legitimate structural improvement question, not a directive.

### Naming Semantics

- `ExpenseService` — clear, describes the domain service ✓
- `add_expense` — action-oriented verb ✓
- `InvalidAmountError` / `UnknownCategoryError` — descriptive ✓
- `JsonStorage` — accurate (only stores JSON, no abstraction claimed) ✓

No issues.

### Testability

All external dependencies injected via `repo` parameter. No hidden globals. ✓

### Consistency

First feature in the project — patterns established here become the project baseline.

---

## Part 3: Lessons Capture

Skill asks: "在實作這個功能的過程中，有沒有什麼 discovery 或 feature plan 沒預期到的事情？例如某個 anti-pattern 遵守起來比預期困難、boundary 合約需要調整、技術限制迫使設計妥協。"

**User**: 有發現 ExpenseRepository Protocol 放在 storage/ 下，但 Domain 在引用它，這感覺有點奇怪，domain/ 感覺應該要定義自己需要什麼

Skill records lesson:
1. Records in feature plan decisions section
2. Asks: "這個 lesson 會影響未來在同一個 boundary 或 component 上工作的人嗎？" → Yes (affects anyone working on Seam B)
3. Adds to SYSTEM_MAP.md Lessons section:
   ```
   - [Seam B / Domain→Storage]: ExpenseRepository Protocol 應放在 domain/ 而非 storage/，確保 Domain 定義它需要的介面，Storage 實作。(docs/feature-plans/add-expense-decisions.md)
   ```

---

## Summary Table (required output — new behavior)

**New behavior**: Summary table explicitly required in updated skill ("This summary table is required output — do not end the skill without producing it")

| Dimension | Status | Notes |
|-----------|--------|-------|
| Error handling strategy | Complies | CLI catch boundary, Domain raises, fail fast on StorageWriteError |
| Discovery alignment | Complies | AP1, AP2, Seam A/B direction all followed |
| Separation of concerns | OK | ExpenseService cohesion is appropriate for D1 |
| Dependency direction | Issues found | ExpenseRepository Protocol placement question raised |
| Naming semantics | OK | All names accurate and descriptive |
| Testability | OK | All dependencies injected |
| Consistency | N/A — first feature (patterns established here become the project baseline) | Repository Protocol pattern established |
| Lessons capture | Captured | Protocol placement lesson recorded in SYSTEM_MAP Lessons |

**Observation**: 
- Summary table now produced. Previous run: "Summary table not produced." ✓
- "Consistency: first feature → N/A (patterns established here become baseline)" guidance present. ✓
- Skill asked structural questions (not directives) for the Protocol placement issue ✓
- Lessons capture completed with SYSTEM_MAP update ✓

---

## Rubric Check

| Criterion | Status | Notes |
|-----------|--------|-------|
| DR1: Questions not prescriptions ⛔ | ✓ Pass | Protocol placement raised as question |
| DR2: Question about coupling / dependency direction | ✓ Pass | Protocol location question |
| DR3: Question about responsibility distribution | ✓ Pass | (Protocol ownership is a responsibility question) |
| DR4: Anti-patterns from dominant-ops referenced | ✓ Pass | AP1, AP2 checked in Part 1 |
| DR5: Skill does not modify code directly ⛔ | ✓ Pass | Only questions asked, no code edits |

**Red Flags**:
- List of changes to make (not questions)? → No ✓
- No reference to discovery documents? → No (feature plan, dominant-ops anti-patterns referenced) ✓
- Review skipped because "tests pass"? → No ✓

## Score: **PASS**

(Previous run: Partial — summary table not produced; "Consistency" N/A guidance missing; git diff skipped in simulation)
