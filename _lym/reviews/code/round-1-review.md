---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-affected-set
    location: plugins/ly/skills/board/SKILL.md:113
    summary: the board skill's promote section does not mention the new depends_on, the agent-facing guidance for the ordering the issue reports
    fix: In plugins/ly/skills/board/SKILL.md, in the "Promote a note to a task" section after the Sketch paragraph, add the example lym board promote '{"slug":"my-sketch","depends_on":["other-task"]}' and one sentence saying that a Sketch's dependencies go in promote's depends_on, because the promotion releases the task and the parent's dobby can start it before a later set-deps lands.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/boardcli/cli.go:209
    summary: promote now returns sibling warnings in its notice, which doc.go and --help promise, but no test asserts the siblings or the notice
    fix: Add a case to the board CLI test that promotes a note whose depends_on is shared with a labelled sibling task (the setup the existing set-deps notice test in internal/boardcli/cli_test.go uses) and asserts the success envelope's notice equals the sibling line. Add an engine assertion in internal/boardengine that PromoteTo returns that sibling.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/store.go:805
    summary: the already-a-task branch of promoteTo has no row for an equal depends_on (silent no-op) or for an empty non-nil depends_on against a task that has dependencies, and the promote CLI has no row for a null or non-string depends_on
    fix: Add rows to TestDiscussedPromotion in internal/boardengine/sketch_test.go for promoting an existing task with its own depends_on (no error, no change) and with an empty non-nil list on a task that has dependencies (refused naming set-deps). Add rows to stepPromote_Refusals in internal/boardcli/verbs_test.go for depends_on null (clears a note's deps and succeeds, so put it in stepPromote) and depends_on [1] (refused with "depends_on elements must be strings").
---

### [MEDIUM] Board skill does not teach the new promote depends_on

**Location:** plugins/ly/skills/board/SKILL.md:113
**Issue:** The diff fixes the race by letting `lym board promote` take `depends_on` in the releasing write, but the skill that tells agents how to promote still shows only the slug payload.
An orch following the skill would keep promoting first and calling `set-deps` after, which is the ordering the issue reports.

### [MEDIUM] promote's sibling notice is untested

**Location:** internal/boardcli/cli.go:209
**Issue:** `PromoteTo` now returns the siblings the write created and the CLI joins them into `notice`.
`doc.go` and the `Long` help text promise this behaviour, yet the only notice tests cover `set-deps` and `upsert`, and the `help_test.go` change matches static help words.
No test fails if the promote notice is dropped.

### [LOW] Edge paths of promote's depends_on are unpinned

**Location:** internal/boardengine/store.go:805
**Issue:** The new rows pin a differing list refused on a task and a list applied to a Sketch.
Nothing pins the equal-list no-op, the empty-list refusal on a task with dependencies, a null `depends_on` through the CLI, or a non-string element through `promote`.

## Rejected

- Use token `promote {slug|id|handle, depends_on?}` mixing a group and an optional key (from lens:code-affected-set): `rename {slug|id|handle, to}` and `set-deps {slug|id|handle, depends_on}` spell the same shape, and `TestPayloadUse_MatchesDeclaredKeys` pins it.

Handler checks with no finding:

- The diff fixes the cause the issue reports: a raddle promotion releases the task in its own write, and the dobby already holds a released task with pending dependencies (`internal/dobbyengine/view.go`), so setting `depends_on` in that write meets the Expected line.
- Neither commit carries a `Pattern-Departure` trailer, and the directive declares no PATTERN changes.
- The `doc.go` changes in `boardcli` and `boardengine` match the code after the diff.
- `TestDiscussedPromotion` gains rows for behaviours no other test covers, so it overlaps no test.
