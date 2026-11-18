```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to a Goal point of the decision record and each Goal point to the changes that build it.

**Decisions.** The record holds only `## Goal`, with no `Added after Discussion` entries.
- **G1.** The shell guard's refusal says that nothing in the command ran.
- **G2.** The refusal names the refused segment.
- **G3.** First check whether the current text already says so.

**Per change**
- **C1.** `rows.go` adds `wholeCommandRefusal`, adds `openSteersWithWholeCommandRefusal`, and wraps `Rows`'s return in it. It traces to **G1**: the clause opens every plain and engine steer. It also traces to **G2**: "The refused part: " is followed by the row's own text, and every row's steer opens by naming what it refuses (rows.go:79, 94, 134, 171, 182, 201, 234, `refuseRawRow`, `rawGoSteer`).
- **C2.** The `Rows` doc comment gains the opening-clause sentence and its over-long lines are re-split (commits 315046fb and b837f1e0). This is forced by C1: the contract changed, and the comment lint requires the split.
- **C3.** `doc.go` gains a paragraph on the shared clause and on notices staying without it. This is forced by C1, since the package doc now describes the changed steer contract.
- **C4.** In `rows_test.go`, `TestCheckTexts` now expects the Codex python steer to open with `wholeCommandRefusal+pythonRefusal`, and the raw-go notice comparison strips the clause. The file header and the `TestCheckTexts` doc comment are restructured. This is forced by C1: the old assertions would fail.
- **C5.** The new `TestRows_SteersOpenWithWholeCommandRefusal` is the test of C1's behavior, so it is forced by C1. Whether it is redundant is for the code-tests lens.
- **C6.** `settings-golden.json` changes 16 lines. This is forced by C1: after stripping the clause, the HEAD golden is byte-identical to the base golden.

**Per decision**
- **G1** is built by C1, documented by C2 and C3, and pinned by C4, C5 and C6.
- **G2** is built by C1 through each row's own text, which names the refused part by kind rather than quoting the matched command segment. `_lym/landing/summary.md` gives the reason: the `grep -E` match spans the earlier segments and can end partway through a JSON escape. The Idea's wording, "names the refused segment", allows this reading: every row's text identifies which part of a compound command was refused. I raise no finding, but I'm flagging the reading to the handler.
- **G3** is a check, not a code change. The summary records its outcome ("None of the old steers said that nothing had run"), and the base steers I read in rows.go confirm that none carried such a clause.

**Unrequested scope:** none. Every hunk traces to G1 or G2, or is forced by C1. Notices are untouched, guard patterns and row matching are unchanged, and claudeengine's webster deny was left alone because it is not a shell guard row.

**Findings:** none.

Tool calls:
1. Read the publish steer (`steerPublishDeny`) and the headless-claude refusal constant in `rows.go` to confirm G2 for every row; my context lacked those two openings.
