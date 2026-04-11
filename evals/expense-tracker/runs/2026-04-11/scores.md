# Run Scores — 2026-04-11

## Scores

| Skill | Score | Key Finding |
|-------|-------|-------------|
| ec:goals-discovery | **Partial** | Three Quality Tests were implicit, not applied explicitly; OQ2 (CLI framework) not surfaced |
| ec:dominant-ops | **Partial** | Theory limits skipped; operations inventory table jumped over; D3 is borderline-not-a-dominant-op |
| ec:system-map | **Partial** | Change Protocol walkthrough skipped; Phase 7 validation exercises not done; Mermaid diagram not explicitly triggered |
| ec:align-internals | **Partial** | OQ1 (storage format) not linked to interface design decision; persistence design (query patterns, indexing) skipped |
| ec:align-surface | **Partial** | Infrastructure planning explicitly skipped without saying why; constraint validation (NFR1 check) implicit |
| ec:feature-planning | **Partial** | SYSTEM_MAP structural update question skipped; user confirmation of final plan not done; Lessons section not checked |
| ec:feature-coverage | **Pass** | All 6 categories covered; user confirmation obtained; "不適用" had reasons |
| ec:gherkin | **Partial** | Output contract scenario duplicates happy path; one confirmed coverage direction ("極大值") dropped silently |
| ec:tdd-workflow | **Pass** | Ledger produced before tests; mock at Storage boundary; SHA-4 correctly tiered as integration test; waited for user confirmation |
| ec:design-review | **Partial** | Summary table not produced; code reading step (git diff) skipped in simulation; "Consistency" N/A guidance missing |
| ec:pre-complete | **Partial** | Commands correctly sourced from CLAUDE.md; no OpenSpec guidance missing; output summary table not produced |

---

## Top Issues Found

### Issue 1: Discovery skills skip their own validation phases
**Found in**: ec:goals-discovery (Phase 7), ec:dominant-ops (Phase 5 Theory Limits), ec:system-map (Phase 7)
**Pattern**: The later phases in each discovery skill — which serve as internal quality checks — are consistently skipped or rushed. The skills read well in isolation but in practice an agent tends to jump to "output" once it has enough content.
**Suggested fix**: Make the validation phases more "blocking" in the SKILL.md instructions. Instead of "before finalizing, do a full pass", use "you MUST complete this checklist before writing the output document — do not write the document before each item is checked."

### Issue 2: "N/A" sections lack explicit handling
**Found in**: ec:align-surface (Infrastructure Planning), ec:pre-complete (OpenSpec section), ec:system-map (Lessons)
**Pattern**: When a phase or section doesn't apply (no background workers → infrastructure skipped; no OpenSpec → step 5 skipped), skills silently skip without saying "this is N/A because...". The user and future evaluators can't tell if it was a deliberate skip or an omission.
**Suggested fix**: Add a note to each optional/conditional section: "If this doesn't apply, write 'N/A — [reason]' and continue." This makes skips visible and auditable.

### Issue 3: Skills don't produce their own required formatting
**Found in**: ec:design-review (summary table), ec:pre-complete (verification table), ec:system-map (Mermaid diagram)
**Pattern**: Several skills specify a required output format (table, Mermaid diagram) but agents tend to skip these when the content "seems obvious" or the conversation reaches a natural conclusion before the format is produced.
**Suggested fix**: Move the output format requirements to the top of each skill ("this skill MUST produce the following outputs: [list]"). Put the format requirements as a checklist rather than prose at the end.

### Issue 4: Handoff between coverage analysis and gherkin is lossy
**Found in**: ec:feature-coverage → ec:gherkin
**Pattern**: The coverage analysis table has a "Specific Scenario Direction" column, but the gherkin skill doesn't have a step that says "verify every row with 'Yes' in Applicable has at least one scenario in the .feature file." One confirmed scenario direction ("極大值" / extreme amount) was dropped without the skill flagging it.
**Suggested fix**: Add to ec:gherkin Step 0: "Cross-reference the approved coverage analysis table; confirm every 'Applicable' row has at least one corresponding Scenario. If any row is missing, flag it before proceeding."

### Issue 5: No-OpenSpec workflow is underspecified
**Found in**: ec:feature-coverage (Step 1), ec:tdd-workflow (Phase 0), ec:pre-complete (Step 5)
**Pattern**: All three skills assume OpenSpec exists and have steps that read from it. When OpenSpec is absent, the skills silently infer from feature plans and .feature files, but there's no explicit guidance on the fallback path. This creates inconsistency across agents.
**Suggested fix**: Add a "If no OpenSpec" fallback paragraph to each skill that references it: "If no OpenSpec change exists, infer SHA statements from the Then steps in the .feature file; note this as 'inferred from .feature' in the ledger."

### Issue 6: ec:dominant-ops D3 selection is too permissive
**Found in**: ec:dominant-ops
**Pattern**: The skill says "no more than 3 dominant operations." But it doesn't say "the 3 must all have meaningful criticality." In this eval, D3 (Manage Categories: 1×1×1 = 1) was included simply to fill the slot. A better D3 exists (Summary/匯總, with meaningful failure impact).
**Suggested fix**: Add a check: "Before finalizing D3, ask: 'Is this more critical than anything else in the inventory? If all remaining operations score equally low, it's acceptable to have only D1 and D2.'"

---

## Skills to Improve Before Next Run

- [ ] **ec:goals-discovery**: Add explicit "Three Quality Tests" as a checklist the agent must apply and show to the user for each goal
- [ ] **ec:dominant-ops**: Make Theory Limits non-optional; add "D3 criticality minimum threshold" check
- [ ] **ec:system-map**: Restructure Phase 7 as blocking checklist before document output
- [ ] **ec:align-internals**: Add "how does OQ1 affect contract design?" as an explicit question when open questions exist in goals.md
- [ ] **ec:feature-planning**: Add explicit "is SYSTEM_MAP structural update needed?" question (Step 3) with a concrete example of A vs B
- [ ] **ec:gherkin**: Add coverage cross-reference check (every 'Applicable' row → at least one Scenario)
- [ ] **ec:tdd-workflow**: Add Checkpoint A + B labels to the ledger output format; add "scan is empty" fast path for first feature
- [ ] **ec:design-review**: Add summary table to output format as a required section; add "Consistency: first feature becomes the baseline" guidance
- [ ] **ec:pre-complete**: Add "no OpenSpec → N/A" explicit instruction for Step 5; require output summary table

## Overall Assessment

**Workflow cohesion**: Good. The handoffs between skills were logical and the discovery → implementation arc worked as intended. No skill blocked when it shouldn't have, and the one blocking check that mattered (feature-planning requiring discovery docs) would have worked correctly.

**Individual skill quality**: Mostly Partial. Skills know what to produce but skip their own internal validation phases. The pattern is consistent: early phases (gathering, eliciting) are well-specified; later phases (validation, cross-checking, required formatting) are underspecified or treated as optional.

**Highest-value fix**: Issue 3 (output format as required checklist at top of skill). If agents consistently produce the required tables and diagrams, the quality of each step improves substantially and gaps become more visible.
