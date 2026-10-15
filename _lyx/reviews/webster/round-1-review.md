---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/loomcli/sharedbootstrap.go:352
    summary: buildLoomShed doc comment still says it holds the fabric-open and landing-assembly block verbatim, but card 5 moved that block into wireLanding
    fix: In the doc comment of buildLoomShed in internal/loomcli/sharedbootstrap.go, drop the claim that the shed-construction block is held here verbatim and the list "open the fabric, read the current branch and origin URL, resolve the recorded origin and the landing parent, assemble Env.Landing"; say instead that it assembles Env.Landing through wireLanding and then constructs the shed via loomrecipe.New, keeping the closing sentence about the return values.
---

### [LOW] buildLoomShed doc comment describes a block it no longer holds

**Location:** internal/loomcli/sharedbootstrap.go:352
**Issue:** Card 5 moved the fabric-open, branch and origin-URL reads and the `Env.Landing` assembly out of `buildLoomShed` into `wireLanding`, and `buildLoomShed` now calls it.
Its doc comment was left as it was: it still says the block is "held here verbatim and in today's order" and lists those steps as its own.
A reader of the comment looks for code that is no longer in the function.

## Verified without findings

- `AppendSkills` keeps `own` first, drops repeats, returns a fresh slice, and `TestAppendSkills` pins the empty, overlapping and spare-capacity cases.
- The darn writer, Webster master, recovery strand and the conflict session at Merge-In and at Finalize all merge the told list after their own skills.
  `lyx loom` loads the module once in `wire`, and `lyx webster` once in `wireHub`.
- The Burler `Profile.LanguageSkills` key reaches review, fix, probe and resume through `seatSkills`.
  `decodeProfile` and `burlerRoundProfile`/`burlerFixProfile` accept it, and `lyx burler run`'s standalone wiring tells an empty list.
- Exactly the Webster, Merge and Darn Burler review and fix rows carry `language-skills: true`, and `TestShippedRecipes_LanguageSkillsOnCodeRounds` pins that set.
- The `language` module is registered with the empty template default and a Guard `skills` key; the code hard-codes no language.
- New tests extend existing tables where one existed, and the package test budgets pass; the targeted packages' `lyx gate test` run passed.
