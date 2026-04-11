# Run Scores — 2026-04-11-v2

## Comparison with Previous Run

| Skill | v1 Score | v2 Score | Key Change |
|-------|----------|----------|------------|
| ec:goals-discovery | Partial | **Pass** | Three Quality Tests now shown explicitly; OQ2 surfaced |
| ec:dominant-ops | Partial | **Pass** | Operations inventory produced first; D3 threshold check fired (redirected to Op3 score=3); Theory Limits produced with [estimate] |
| ec:system-map | Partial | **Pass** | Mermaid diagram required and produced; Phase 7 blocking checklist executed; Change Protocol 4 types covered; Lessons N/A comment present — *note: Required Outputs was misplaced at file bottom; fixed during eval* |
| ec:align-internals | Partial | **Pass** | OQ1 explicitly linked to Seam B contract design; ExpenseRepository Protocol defined abstractly |
| ec:align-surface | Partial | **Pass** | "Infrastructure N/A — single-process CLI" explicitly stated; NFR1 constraint validation explicit |
| ec:feature-planning | Partial | **Pass** | SYSTEM_MAP structural update question asked; Three Decisions tied to specific boundaries; Lessons section checked (empty) |
| ec:feature-coverage | Pass | **Pass** | No regression; still blocking on confirmation |
| ec:gherkin | Partial | **Pass** | Step 0 cross-reference check: all 6 coverage rows verified against scenarios; 極大金額 scenario explicitly present |
| ec:tdd-workflow | Pass | **Pass** | No regression; new behaviors confirmed: "SHALLs inferred from .feature" header present; Checkpoint A+B labels in ledger |
| ec:design-review | Partial | **Pass** | Summary table produced; "Consistency: first feature → baseline" N/A note present; Lessons captured |
| ec:pre-complete | Partial | **Pass** | "N/A — no OpenSpec" explicitly stated; verification table produced |

**v1: 2 Pass / 9 Partial / 0 Fail**
**v2: 11 Pass / 0 Partial / 0 Fail**

---

## Issues Found and Resolved

### Issue 1: Discovery skills skip validation phases
**Status**: ✅ Resolved

**What changed**:
- goals-discovery: Phase 7 now blocking; Three Quality Tests shown explicitly to user with results
- dominant-ops: Phase 5 (Theory Limits) now explicitly marked "required — do not skip"; [estimate] markers enforced
- system-map: Phase 7 now "MUST complete all three checks before writing"; each check result shown to user

**Evidence from v2**: goals-discovery Phase 7 ran all four checks before writing; dominant-ops produced Theory Limits table with [estimate]; system-map Phase 7 ran navigation, change simulation, and newcomer tests.

---

### Issue 2: N/A sections silently omitted
**Status**: ✅ Resolved

**What changed**:
- align-surface: Single-process CLI N/A guidance added; "state explicitly... do not silently omit"
- pre-complete: "If no OpenSpec → mark as N/A — no OpenSpec, proceed to 5b"
- system-map: Lessons section must contain `<!-- Empty at initial creation — ... -->` comment

**Evidence from v2**: align-surface stated "Infrastructure planning N/A — single-process CLI" explicitly; pre-complete stated "N/A — no OpenSpec" explicitly; SYSTEM_MAP Lessons section present with comment.

---

### Issue 3: Required output formats not produced
**Status**: ✅ Resolved (with one positioning bug caught and fixed)

**What changed**:
- Required Outputs sections added to: goals-discovery, dominant-ops, system-map, design-review, pre-complete
- Mermaid diagram marked "required, not optional" in system-map Required Outputs

**Bug found during v2**: system-map Required Outputs was placed at the END of the file (line 278, after Key Rules), not before ## Workflow. This means an agent would start the workflow without seeing it. **Fixed during this eval run** — moved to after Working Style, before Prerequisites.

**Evidence from v2 (post-fix)**: System-map produced Mermaid diagram; design-review produced summary table; pre-complete produced verification table.

---

### Issue 4: Coverage→Gherkin handoff lossy
**Status**: ✅ Resolved

**What changed**:
- ec:gherkin Step 0 (Coverage Cross-Reference Check) added: "For every row marked Applicable = Yes, verify that at least one Scenario corresponds to the Specific Scenario Direction. If any Applicable row has no planned Scenario → flag it before writing."

**Evidence from v2**: Step 0 built an explicit table of all 6 coverage rows vs planned scenarios. "極大金額 (999999999)" scenario explicitly present — was silently dropped in v1.

---

### Issue 5: No-OpenSpec fallback paths undefined
**Status**: ✅ Resolved

**What changed**:
- ec:tdd-workflow Step 1: "If there is no OpenSpec: infer behaviors from Then steps in .feature file; note as 'SHALLs inferred from .feature file (no OpenSpec)'"
- ec:pre-complete Step 5: "If no OpenSpec → mark as N/A — no OpenSpec, proceed to 5b. Do not skip silently."

**Evidence from v2**: tdd-workflow ledger header shows "SHALLs inferred from .feature file (no OpenSpec)"; pre-complete verification table shows "N/A — no OpenSpec" in both OpenSpec rows.

---

### Issue 6: ec:dominant-ops D3 selection too permissive
**Status**: ✅ Resolved

**What changed**:
- Required Outputs: "Before finalizing D3, check its criticality score. If ≤ 2, ask: 'Is this genuinely more critical than any unselected operation?'"
- Phase 3 Hard rules: "No fewer than necessary. Before finalizing D3, verify: does its criticality score exceed 2? If not, ask. A two-operation list is better than a padded three."

**Evidence from v2**: D3 = 管理類別 (score=1) was rejected. Skill explicitly asked which operation was more critical. D3 reassigned to 查看摘要 (score=3, genuine Failure Impact — incorrect summary leads to bad financial decisions).

---

## New Issue Found During v2

### Issue 7: system-map Required Outputs misplaced (positioning bug)
**Found in**: ec:system-map
**Pattern**: Required Outputs section was added at the END of the file (after Key Rules at line 278), not before ## Workflow. An agent reading top-to-bottom would process all workflow phases before seeing the Required Outputs checklist.
**Fix**: Already applied — moved to after Working Style, before Prerequisites. Consistent with goals-discovery and dominant-ops placement.

---

## Remaining Gaps

### Minor: ec:align-internals has no Required Outputs section
Not blocking in v2 (the skill performed well), but for consistency all skills should have a Required Outputs section before ## Workflow. Current skill relies only on workflow steps without an explicit pre-check list.

### Minor: ec:feature-coverage has no Required Outputs section
Same pattern. The skill works correctly (it has blocking language in Step 5 and Step 6), but lacks the unified Required Outputs format seen in other skills. Lower priority since it was Pass in both runs.

---

## Overall Assessment

**v2 vs v1**: All 9 Partial scores converted to Pass. No regressions.

**Root cause of v1 failures**: Skills had validation phases at the end of the workflow ("before finalizing..."), which are easy to skip once the agent "feels done." Moving to Required Outputs at the top + blocking language ("MUST complete before writing") turns passive guidelines into gates.

**Highest-impact fix confirmed**: Issue 3 (Required Outputs as blocking checklist at top of skill). Once agents see the checklist before starting, they produce the required outputs. The format also makes it easy to verify: "Did the skill produce the Mermaid diagram? Check Required Outputs — it says required, not optional."

**Skills to monitor in future runs**:
- ec:align-internals: add Required Outputs section (minor, low priority)
- ec:feature-coverage: add Required Outputs section (minor, already blocking via Step 5)
- ec:system-map Required Outputs: verify positioning holds across future edits

## Skills to Improve Before Next Run

- [ ] **ec:align-internals**: Add Required Outputs section before ## Workflow (for consistency)
- [ ] **ec:feature-coverage**: Add Required Outputs section before ## Workflow (for consistency)
- [ ] Consider adding a "difficult user persona" to scenario.md to test whether skills handle uncooperative users (e.g., user says "都可以，你決定就好" or tries to skip steps)
