# Eval Rubric: Expense Tracker CLI

Score each skill as **Pass / Partial / Fail** using the criteria below.
A **Partial** on any item marked ⛔ is automatically a **Fail**.

---

## ec:goals-discovery

### Required Output Criteria

| # | Criterion | Flag |
|---|-----------|------|
| G1 | At least 4 functional goals, each with a Gx ID and STABLE/EVOLVING tag | |
| G2 | At least 3 non-goals; each one explicitly says *why* it is excluded | |
| G3 | At least 2 NFRs that are **quantified** (e.g., "< 500ms startup", not "fast") | |
| G4 | At least 1 open question that is genuinely unresolved | |
| G5 | No goal is an implementation detail (e.g., "use SQLite", "use argparse") | ⛔ |
| G6 | All goals use strong verbs (SHALL/MUST), not "can"/"may" | |

### Red Flags

- Skill writes goals.md itself without asking questions first → **Fail**
- Storage technology appears as a goal (not a constraint or open question) → **Fail**
- Non-goals section has fewer than 3 entries → **Partial**
- NFRs are unquantified ("the tool should be fast") → **Partial**
- Skill does not probe for non-goals after user gives only positive description → **Partial**

### Scoring Guide

- All Pass + no red flags → **Pass**
- 1-2 criteria missing (none ⛔) OR 1 red flag → **Partial**
- Any ⛔ criterion missing OR >1 red flag → **Fail**

---

## ec:dominant-ops

### Required Output Criteria

| # | Criterion | Flag |
|---|-----------|------|
| D1 | Each dominant operation has: frequency estimate, failure cost, failure impact | |
| D2 | Write (add expense) and Read (query/list) are identified as separate ops | |
| D3 | The frequency asymmetry (write >> read in count) is explicitly noted | |
| D4 | The failure asymmetry (broken query is worse than missed write) is explicitly noted | |
| D5 | At least 1 anti-pattern per dominant op, with an ID (AP1, AP2...) | ⛔ |
| D6 | Each Dx ID appears in the output for downstream traceability | ⛔ |

### Red Flags

- Skill lists operations without assessing pressure (frequency × cost × impact) → **Fail**
- Anti-patterns are missing or have no IDs → **Fail**
- Write and Read are collapsed into one operation → **Partial**
- Data corruption risk on write is not mentioned → **Partial**

---

## ec:system-map

### Required Output Criteria

| # | Criterion | Flag |
|---|-----------|------|
| S1 | Component Map identifies at least 3 components (CLI layer, domain logic, storage) | |
| S2 | Boundary Map defines at least 2 seams with direction and contract stub | ⛔ |
| S3 | Change Protocol table covers at least 3 change types | |
| S4 | Current State section exists (even if mostly gaps) | |
| S5 | Lessons section exists (can be empty on first run) | |

### Red Flags

- Only 1 seam defined (CLI ↔ storage, skipping domain logic) → **Partial**
- No contract stubs at boundaries → **Fail**
- Change Protocol is missing → **Partial**
- SYSTEM_MAP is a narrative description rather than a structured document → **Fail**

---

## ec:align-internals

### Required Output Criteria

| # | Criterion | Flag |
|---|-----------|------|
| AI1 | Each seam from SYSTEM_MAP is explicitly assessed | ⛔ |
| AI2 | Output states whether each contract exists, is defined, or is a gap | |
| AI3 | Storage seam contract is assessed (what does the domain layer expect from storage?) | ⛔ |
| AI4 | Alignment report distinguishes design mode from verification mode | |

### Red Flags

- Skill produces generic advice not tied to specific seams → **Fail**
- Storage contract is described in terms of a specific technology (SQLite schema vs. abstract interface) at this stage → **Partial**

---

## ec:align-surface

### Required Output Criteria

| # | Criterion | Flag |
|---|-----------|------|
| AS1 | Each Dx user journey is mapped to at least one CLI command | ⛔ |
| AS2 | Command signatures are assessed (input shape, output shape) | |
| AS3 | Report identifies any Dx journey with no corresponding CLI entry point | |
| AS4 | Infrastructure assessment covers storage location and access pattern | |

### Red Flags

- Skill does not reference Dx IDs from dominant-ops → **Partial**
- CLI commands are designed without reference to user journeys → **Partial**

---

## ec:feature-planning

### Required Output Criteria

| # | Criterion | Flag |
|---|-----------|------|
| FP1 | Skill reads and references all four discovery documents before producing output | ⛔ |
| FP2 | Candidate list is produced with Gx mapping for each candidate | ⛔ |
| FP3 | User is asked to choose; skill does not auto-select | |
| FP4 | Three Decisions (catch boundary, error taxonomy, recovery strategy) are all addressed | ⛔ |
| FP5 | Feature plan document is created at `docs/feature-plans/{name}.md` | |
| FP6 | Anti-patterns from dominant-ops appear in the Constraints section with AP IDs | ⛔ |
| FP7 | SYSTEM_MAP Current State is updated | |

### Red Flags

- Skill proceeds without SYSTEM_MAP.md present → **Fail** (blocking check ⛔)
- Three Decisions are generic ("handle errors appropriately") rather than tied to this feature's boundaries → **Fail**
- Feature plan exists but has no Serves: Gx mapping → **Partial**

---

## ec:feature-coverage

### Required Output Criteria

| # | Criterion | Flag |
|---|-----------|------|
| FC1 | All 6 categories are addressed explicitly (適用/不適用 + reason) | ⛔ |
| FC2 | Skill waits for user confirmation before writing the .feature file | ⛔ |
| FC3 | Storage boundary is mapped to Error/Failure category, not Happy Path | |
| FC4 | Input boundary category includes at least: empty amount, negative amount, unknown category | |

### Red Flags

- Any category is skipped without a "不適用" + reason → **Fail**
- Skill writes Gherkin before confirmation → **Fail**
- Scenarios are listed but not mapped to the 6 categories → **Partial**

---

## ec:gherkin

### Required Output Criteria

| # | Criterion | Flag |
|---|-----------|------|
| GH1 | All keywords (Feature, Scenario, Given, When, Then, And, But, Background) are in English | ⛔ |
| GH2 | All step descriptions and names are in Traditional Chinese | ⛔ |
| GH3 | Feature/Rule/Scenario hierarchy is used correctly | |
| GH4 | Each scenario from the approved coverage analysis appears in the .feature file | |
| GH5 | Steps describe behavior, not implementation (no "click button X", no SQL references) | |

### Red Flags

- Any keyword in Chinese (功能, 情境, 假設, 當, 那麼) → **Fail**
- Any step content in English → **Partial** (unless it's a technical term with no Chinese equivalent)
- Scenarios added that were not in the confirmed coverage analysis → **Partial**

---

## ec:tdd-workflow

### Required Output Criteria

| # | Criterion | Flag |
|---|-----------|------|
| TDD1 | Verification Ledger is produced and shown to user before any test code is written | ⛔ |
| TDD2 | Ledger explicitly places mock boundary at storage layer (not inside domain logic) | ⛔ |
| TDD3 | Skill waits for user confirmation of ledger before proceeding | ⛔ |
| TDD4 | Red phase: tests are written and `pytest` output showing failures is presented | ⛔ |
| TDD5 | Skill waits for user confirmation that Red is seen before writing production code | ⛔ |
| TDD6 | Green phase: `pytest` output showing all tests pass is presented | |
| TDD7 | Step definitions follow scenario order, not step-type grouping | |

### Red Flags

- Production code written before Red is confirmed → **Fail**
- Ledger skipped or merged into test writing step → **Fail**
- Mock placed on domain logic method rather than storage boundary → **Fail**
- `MagicMock()` used without `spec=` → **Partial**

---

## ec:design-review

### Required Output Criteria

| # | Criterion | Flag |
|---|-----------|------|
| DR1 | Skill asks questions rather than prescribing changes | ⛔ |
| DR2 | At least one question about coupling / dependency direction | |
| DR3 | At least one question about responsibility distribution | |
| DR4 | Anti-patterns from dominant-ops are referenced in the review | |
| DR5 | Skill does not modify code directly | ⛔ |

### Red Flags

- Skill produces a list of changes to make (not questions) → **Fail**
- Review makes no reference to the discovery documents → **Partial**
- Skill skips review because "tests pass" → **Fail**

---

## ec:pre-complete

### Required Output Criteria

| # | Criterion | Flag |
|---|-----------|------|
| PC1 | All required commands (pytest, ruff, pyright or equivalents from CLAUDE.md) are run | ⛔ |
| PC2 | Actual command output is shown, not assumed | ⛔ |
| PC3 | Skill does not declare completion if any command fails | ⛔ |
| PC4 | Integration test gaps from the feature plan are reviewed | |
| PC5 | Delta spec sync to openspec/ is prompted | |

### Red Flags

- "Complete" declared without running commands → **Fail**
- Commands assumed to pass ("ruff should be fine since we followed conventions") → **Fail**
- Integration gaps from feature plan are ignored → **Partial**

---

## Overall Run Score

After all skills are scored, summarize in `runs/YYYY-MM-DD/scores.md`:

```markdown
# Run Scores — YYYY-MM-DD

| Skill | Score | Notes |
|-------|-------|-------|
| ec:goals-discovery | Pass/Partial/Fail | ... |
| ec:dominant-ops | Pass/Partial/Fail | ... |
| ec:system-map | Pass/Partial/Fail | ... |
| ec:align-internals | Pass/Partial/Fail | ... |
| ec:align-surface | Pass/Partial/Fail | ... |
| ec:feature-planning | Pass/Partial/Fail | ... |
| ec:feature-coverage | Pass/Partial/Fail | ... |
| ec:gherkin | Pass/Partial/Fail | ... |
| ec:tdd-workflow | Pass/Partial/Fail | ... |
| ec:design-review | Pass/Partial/Fail | ... |
| ec:pre-complete | Pass/Partial/Fail | ... |

## Top Issues Found

1. [Issue] — found in [skill] — [suggested fix]
2. ...

## Skills to Improve Before Next Run

- [ ] [skill]: [what to fix]
```
