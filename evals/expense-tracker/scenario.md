# Eval Scenario: Expense Tracker CLI

## What This Scenario Tests

A deliberately small Python CLI project with enough moving parts to stress-test every skill:

- **Discovery skills**: goals aren't obvious, dominant operation tension exists (write-heavy vs read-heavy usage patterns differ), non-goals are easy to get wrong
- **Boundary skills**: three natural seams (CLI ↔ logic ↔ storage) with real contract questions
- **Implementation skills**: full TDD cycle is feasible within one session, design review has genuine material to work with

---

## User Context Block

> **Paste this verbatim as your first message in the eval session. Do not add, remove, or explain anything.**

---

我想建一個用 Python 寫的命令列工具，讓我可以記帳。

我每天會花錢在一些固定的地方——餐廳、交通、買東西之類的。現在我都用記事本記，但常常忘記或找不到。我想要一個簡單的工具，可以快速把花費加進去，之後也可以查詢。

技術上我想用 Python，其他的我沒有特別的限制。

---

## Eval Constraints (Do Not Share With Claude)

These are the hidden truths of the scenario — things the skills should elicit, not assume:

| Topic | Hidden Answer |
|-------|--------------|
| Storage | The user has no preference; SQLite is fine but so is a JSON file — this is a real design decision to surface |
| Categories | The user wants to define their own categories, not pick from a fixed list |
| Date | The user usually wants "today" but occasionally enters past expenses |
| Query needs | Listing by category AND by date range — user hasn't thought about this yet |
| Non-goals | Multi-currency (no), export to spreadsheet (no), budget alerts (no), multi-user (no) |
| Dominant operation | Adding expenses is 10× more frequent than querying; but querying is the one that fails painfully if broken |
| CLI style | User doesn't care about TUI; plain text output is fine |

## What Good Discovery Looks Like

By the end of `ec:goals-discovery`, the output should have surfaced:
- At least 4 functional goals (add, list, summarize, manage categories)
- At least 3 non-goals that meaningfully constrain design
- NFRs that are quantified (startup time, not "fast")
- An open question about storage that is not pre-answered

By the end of `ec:dominant-ops`, the output should have identified:
- The write/read frequency asymmetry
- The failure consequence asymmetry (forgotten write = mild annoyance; broken query = can't trust data)
- An anti-pattern around data corruption on write

## Known Difficulty Points

These are places where skills are expected to work harder — if they don't, it's a signal:

1. **Goals abstraction level** — Users often say things like "I want to use SQLite" or "I want a CSV file." `ec:goals-discovery` must catch this and redirect to the goal behind it.
2. **Non-goal elicitation** — The user's description is positive-only ("I want X"). The skill must actively probe for what the user will NOT build.
3. **Dominant ops pressure** — The write/read asymmetry is not obvious from the user's description. `ec:dominant-ops` must ask about frequency and failure cost explicitly.
4. **Boundary discipline in system-map** — There's a temptation to put "everything in one file." The skill must surface the seam between CLI parsing and business logic.
5. **Feature-planning blocking** — If the evaluator tries to jump straight to `ec:feature-planning` without running discovery first, the skill must block.
6. **Verification Ledger granularity** — The storage boundary (file I/O or SQLite) is a real external dependency; the ledger must place the mock there, not inside the business logic.
