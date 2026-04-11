# Skill Run: ec:design-review

## Prerequisites Check

- Tests: green ✓ (assumed from TDD Phase 2)
- Lint + type check: passed ✓ (assumed from TDD Phase 3)

## Phase 0: Collect Declared Design Decisions

Read `docs/feature-plans/add-expense.md`:
- Catch boundary: CLI layer only
- Domain errors: CategoryNotFound, InvalidAmount
- Infrastructure errors: fail fast
- Anti-patterns: AP1, AP2, AP4

## Part 1: Declared Decisions Verification

*(Actual code inspection omitted — this section describes what would be checked)*

| Declared Decision | Declared Content | Verification Status | Notes |
|-------------------|-----------------|---------------------|-------|
| Catch boundary | CLI layer only | Complies | Domain.add_expense() raises, does not catch |
| Domain errors | CategoryNotFound, InvalidAmount | Complies | Both raised correctly with structured messages |
| Recovery strategy | fail fast on infra error | Complies | StorageError propagates without retry |
| Anti-pattern AP1 | 原子寫入 | Complies | JSON storage uses write-then-rename pattern |
| Anti-pattern AP2 | 分類驗證先於寫入 | Complies | CategoryRepository.exists() called before add() |
| Anti-pattern AP4 | 不部分覆寫 | Complies | write-then-rename prevents partial writes |

## Part 2: Structural Checks

**Separation of concerns** `domain.py:45`
> Current: `ExpenseService.add_expense()` validates input AND formats the result for display.
>
> Direction to consider: Should formatting the display string be the domain's responsibility, or should the CLI layer decide how to present an `Expense` object?
>
> Why it's worth thinking about: If you later want to add a `--json` output flag, the domain formatting would need to change even though the business logic didn't.

**Dependency direction** — OK: Domain → Storage Protocol (not the reverse)

**Naming semantics** `storage.py:12`
> Current: `JSONStorage.save()` — "save" is ambiguous (create? update?).
>
> Direction to consider: Does `save()` mean "create a new record" or "write to disk"? If it's always append-only (new expense), `append()` or `add()` might communicate intent more clearly.
>
> Why it's worth thinking about: A future developer reading `save()` might expect it to handle both create and update cases.

**Testability** — OK: Domain functions are injectable via Protocol; no hidden file system access in tests.

**Consistency** — N/A: first feature in project, no existing patterns to compare against.

## Part 3: Lessons Capture

**Claude:**
> 在實作過程中，有沒有什麼是 discovery 或 feature plan 沒預料到的？

**User:**
> 有，我發現 write-then-rename 在 Windows 上行為不同（atomic rename 在 Windows 上如果目標檔案已存在會失敗）。所以 AP4 的保護比預期難實作。

**Claude:**
> 這個要記錄到 SYSTEM_MAP Lessons，因為任何之後碰到 Storage Seam B 的開發者都需要知道。

→ SYSTEM_MAP Lessons entry added:
```
- [Seam B / Storage]: Windows 上的 atomic rename 需要 os.replace() 而非 os.rename()；
  詳見 openspec/changes/add-expense/decisions.md
```

---

## Skill Observations

### ✅ Did well
- Part 1 was done before Part 2 (correct order)
- Part 2 raised questions (not directives): "Should formatting be the domain's responsibility?" not "Move formatting to CLI"
- The formatting concern at domain.py:45 is a legitimate separation-of-concerns issue
- Naming semantics check caught an ambiguous name (`save()`)
- Lessons Capture was asked at the end and a real lesson was captured
- Correctly judged that the Windows lesson belongs in SYSTEM_MAP (future Storage work affected)

### ⚠️ Gaps observed
- **Part 1 table was produced without reading actual code.** In a real eval with real code, the skill needs to read files before filling in the table. The skill instruction says to run `git diff --name-only HEAD~1` — this was omitted in the simulation.
- **Structural check for "Consistency"** was marked N/A (first feature). But the skill doesn't have guidance on what to do when there's no baseline — it should suggest "this becomes the baseline; note the patterns established here for future features."
- **Testability check** was light. The skill should be more specific: "Can you add a test for the storage failure case without touching real disk?" The answer (yes, via Protocol mock) is non-obvious and worth surfacing.
- **The summary table** (the skill's Output Format requires a final Summary table) was not produced in this simulation.
