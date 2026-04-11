# Skill Run: ec:pre-complete

## Simulated Conversation

**Claude:**
> 在宣告完成之前，執行驗證清單。請先告訴我這個專案的 Commands 設定（CLAUDE.md 的 Commands 區段）。

**User:**
> 測試用 `pytest`，lint 用 `ruff check . --fix && ruff format .`，type check 用 `pyright`。

**Claude:**
> 好，開始執行：

*(Simulated — actual command outputs not run since project code was not written in this eval)*

**Claude (following skill — shows that it would NOT claim completion without output):**
> 注意：以下是「應該要發生的事」的描述，因為這是 eval 模擬，不是真實專案。在真實執行中，我會顯示每個指令的完整輸出後才繼續。

---

## What Would Happen (Real Run)

### 1. Tests
```bash
$ pytest
# Expected: all N tests PASS, show full output
```

### 2. Lint & Format
```bash
$ ruff check . --fix && ruff format .
# Expected: 0 errors
```

### 3. Type Check
```bash
$ pyright
# Expected: 0 errors
```

### 4. Change Confirmation
```bash
$ git diff --stat && git status
# Show changed files, confirm no .env or credentials
```

### 5. OpenSpec Status
- No OpenSpec used in this project → N/A
- Delta Spec Sync → N/A

### 5b. Discovery Document Sync
- Was SYSTEM_MAP updated during implementation? **Yes** — Lessons section was updated with Windows rename issue
- Does dominant-ops need updating? **No** — AP4 was already there, just implementation detail surfaced
- Does goals.md need updating? **No**

### 6. Integration Test Gaps
Read `docs/feature-plans/add-expense.md` Integration Test Gaps:
- **SHALL-4** (Storage fail fast): needs integration test, reason = requires real Storage behavior
- Not yet addressed → remind user, do not block commit

**Claude:**
> Integration Test Gap 未解決：SHALL-4 需要 integration test 驗證真實 Storage 的 fail fast 行為。建議在下一個 spec 完成後，或在 PR merge 到 main 前補上。是否現在寫，還是追蹤到 issue？

**User:**
> 追蹤到 issue，先繼續。

---

## Skill Observations

### ✅ Did well
- Skill asked for Commands from CLAUDE.md before running — does not assume toolchain
- Correctly would NOT declare completion without running commands (the principle is in the skill)
- Integration Test Gaps were reviewed and the SHALL-4 gap was surfaced
- Discovery Document Sync was checked (5b) — SYSTEM_MAP lesson update was part of completion

### ⚠️ Gaps observed
- **The skill cannot force real command execution** when operating in a simulated/eval context. In a real run this would work, but the skill has no mechanism to distinguish "ran and passed" from "assumed would pass." A real agent could write "all checks pass" without actually running them — the skill relies on the agent's integrity.
- **No guidance for "project has no OpenSpec."** The skill's step 5 (OpenSpec Status) is always shown, but for projects without OpenSpec this creates confusion. The skill should say "if project does not use OpenSpec, skip this step and mark as N/A."
- **The output summary table** (| Verification Item | Status | Notes |) was not produced. The skill says to present results in a table — this step was described but not formatted.
- **Step 5b (Discovery sync) check was rushed.** For a real project where dominant-ops gained a new observation during implementation (the Windows behavior gap), should dominant-ops.md be updated? The skill says yes if "hit an anti-pattern that is not documented" — but AP4 was already there. The skill could be clearer about the threshold for updating discovery docs.
