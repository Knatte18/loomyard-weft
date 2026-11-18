---
round: 1
focus:
  - "internal/shellguard/rows.go: does every refusal steer open with the clause that nothing in the command ran, and does each steer still name the refused segment, as the board entry's Idea asks?"
  - "internal/shellguard/rows.go and doc.go: does the change stay inside the Idea's scope (refusal text), or does it alter guard decisions, row matching or steer content beyond the opening clause?"
  - "internal/claudeengine/testdata/settings-golden.json: does the golden change follow only from the new steer text, with no unrelated drift?"
  - "internal/shellguard/rows_test.go: TestRows_SteersOpenWithWholeCommandRefusal is reported contained by TestCheckTexts and TestRows_EngineTexts; compare assertions and decide whether to fold, delete or mark it keep."
  - "Commit messages and the change description: do they match the diff, and are any Pattern-Departure trailers present and valid?"
---

Seed focus for round 1, drawn from the board entry under `## Goal` and the committed diff stat.
