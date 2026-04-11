# Start Prompt — 2026-04-11

## Eval Run Info

- Date: 2026-04-11
- Evaluator: Claude (self-evaluation)
- Model: claude-sonnet-4-6
- Plugin version: current (feat/skill-consolidation branch)
- Project setup: fresh directory (no existing code, no OpenSpec)

## User Context Block (verbatim, as sent to Claude)

> 我想建一個用 Python 寫的命令列工具，讓我可以記帳。
>
> 我每天會花錢在一些固定的地方——餐廳、交通、買東西之類的。現在我都用記事本記，但常常忘記或找不到。我想要一個簡單的工具，可以快速把花費加進去，之後也可以查詢。
>
> 技術上我想用 Python，其他的我沒有特別的限制。

## User Role Constraints

When answering skill questions, I (playing the user) will:
- Only reveal information that the skill explicitly asks for
- Reveal hidden truths from scenario.md when the question directly targets them
- Not volunteer information the skill hasn't asked about yet
- Behave as a real user would: positively-biased, underspecifying, not thinking about edge cases
