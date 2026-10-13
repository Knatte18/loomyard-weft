---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/06-loom-fmt-verb.md
    summary: lyx loom fmt's first file argument is taken as a run-id by loom's pre-run, so any path with a separator is refused before RunE runs
    fix: In card 6, add `internal/loomcli#loomCLI.resolveRunID` and `internal/loomcli#TestResolveRunID` under **Edit:** (validate-plan --overlay admits both). In **Intent:**, say that resolveRunID in internal/loomcli/arm.go adds fmt to the verbs whose positional arguments are not a run-id (beside reject, insert-card and remove-card), and that its comment names fmt's file arguments. Add a Tests bullet saying TestResolveRunID gains a row like {name - fmt's file arguments are not a run-id, verb - fmt, args - internal/x/y.go, wantRunID - shedrun.SelfRunID}, and that TestFmtCmd skips the pre-run, so this row is what pins it.
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/06-loom-fmt-verb.md
    summary: planGoFormat lists a refusal for an unreadable file but names no reason for a path that does not exist, and no TestFmtCmd row reaches either case
    fix: In card 6's **Intent:**, add a nonexistent path to planGoFormat's refused reasons, with its reason text (for example `<path as named>` followed by no such file), and say it comes from the EvalSymlinks failure before the inside-root check. Add a TestFmtCmd row where a nonexistent .go path, named beside a misformatted file, refuses the whole call, with `refused` naming the missing path and the misformatted file left unwritten. No target changes.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/07-worktree-leftover-retry.md
    summary: removeWorktreeLeftover is a new recording executor in destroy.go, but card 7 leaves it out of the mutation-record guard's executor table
    fix: In card 7, add `cmd/lyx#destructiveGuardRecordingExecutors` under **Edit:** (validate-plan --overlay admits it). In **Intent:**, say the table gains the row {removeWorktreeLeftover, "func removeWorktreeLeftover(rec *Mutations, "}, so TestMutationRecord_FabricengineProductionSource pins its leading recorder per PATTERN-mutation-record.
---

### [BLOCKING] `lyx loom fmt`'s file arguments are parsed as a run-id

**Location:** `_lyx/plan/06-loom-fmt-verb.md` (Intent and Tests); `internal/loomcli/arm.go:88-93`, `internal/loomcli/arm.go:179-184`, `internal/shedrun/seed.go:99-101`, `internal/shedrun/runid.go:32-33`.

**Issue:** Every loom verb goes through `armAt`, which calls `resolveRunID` and then `resolveRecipe` before any wiring.
`resolveRunID` sets `c.runID = args[0]` for every verb except `reject`, `insert-card` and `remove-card`.
`resolveRecipe` then calls `shedrun.ReadSeed(location, c.runID)`, and `ReadSeed` starts with `ValidateRunID`, which refuses a run-id containing `/` or `\`.
So `lyx loom fmt internal/x/y.go`, the normal way an agent names a file, fails in the pre-run with `shedrun: run-id "internal/x/y.go" must not contain a path separator`, and `fmtCmd`'s `RunE` never runs.
Running `lyx loom status internal/x.go` here shows that same ValidateRunID refusal.
A bare `x.go` gets through only because `x.go` happens to be a valid run-id.
Card 6 never mentions `resolveRunID`.
Its `TestFmtCmd` calls `c.fmtCmd()` directly through `clihelp.Execute`, which skips the loom pre-run, so the planned tests pass while the real verb refuses.
D6 requires `lyx loom fmt <file>...` to format the named files, and the acceptance criteria say it rewrites a misformatted `.go` file.
As planned, the shipped verb fails both for any file outside the worktree root directory.

### [LOW] Card 6 says nothing about a nonexistent file, and no test reaches the unreadable-file refusal

**Location:** `_lyx/plan/06-loom-fmt-verb.md`, the `planGoFormat` paragraph and the TestFmtCmd rows.

**Issue:** `planGoFormat` runs `filepath.EvalSymlinks` on each path before the inside-root and `.go` checks.
EvalSymlinks returns an error for a path that does not exist, and a mistyped path is the most likely bad input from an agent.
The card's list of refused reasons covers outside the worktree, a symlink outside it, non-`.go`, unreadable and unparsable files, but not a missing path.
The card does list an unreadable file as a refusal, yet none of the TestFmtCmd rows reaches it.
So one refusal branch has no test, and the missing-path case has no stated reason.
The rubric's reachability rule requires a named input and test for each prescribed refusal branch.

### [LOW] The new recording executor is missing from the mutation-record guard table

**Location:** `_lyx/plan/07-worktree-leftover-retry.md`; `cmd/lyx/destructiveguard_test.go:119-140`.

**Issue:** `destructiveGuardRecordingExecutors` is documented as the table of every executor in `internal/fabricengine/destroy.go` that the Mutation Record Invariant requires to take a leading `rec *Mutations`.
`TestMutationRecord_FabricengineProductionSource` checks each row's declaration prefix.
Card 7 adds `removeWorktreeLeftover(rec *Mutations, ...)` to `destroy.go`, and it appends `KindPathRemoved`, so it is a recording executor.
The card does not add it to the table, so nothing pins its recorder parameter.
The decision record's `PATTERN-mutation-record` constraint names this executor's entry.

## Verdicts on the focus directive

- Fidelity: each of D1 to D7 and each Constraint is carried by a card.
  The decision record licenses card 6 (D6, scope item 6) and card 7 (D7, scope item 7).
  No card does an Out-of-scope item.
  The same-commit docs are covered: card 1 does `shuttleengine`, `claudeengine`, `PATTERN.md` and `docs/overview.md`; card 2 does `shuttleengine`; card 3 `statuspane`; card 4 the `pushretry.go` doc; card 6 `loomcli`, `docs/overview.md` and the operator index through the operator audience annotation that `lyx help index` renders; card 7 `fabricengine` and its pattern background.
  The defect is F1, where card 6 does not deliver what D6 requires.
- Attack surface and reachability: the Intents of cards 1, 2, 6 and 7 each state what they let through and what bounds them.
  Every timeout and refusal branch in cards 1, 2 and 7 has a named input and test.
  Card 7's gate-refused branch is pinned by the counting seam on `TestRemove_RefusesForeignWorktreeWithoutDeletingIt`, where the pre-git ownership gate refuses.
  F1 and F2 are the gaps.
- Card order and shared files: cards 1 and 2 edit different paragraphs of `shuttleengine/doc.go`, and cards 6 and 7 edit different rows of `test-budget.yaml`.
  Card 2 relies on card 1's launch-carried first turn and comes after it.
  No card depends on a later one.
- Edit glyphs: card 1's two re-signed members (`loadSkillsThenPrompt`, `loadSkillTurn`) carry the arrow.
  `settleLoadTurn`, `Claude.Prepare` and `Runner.start` keep their signatures.
  Card 3 changes only struct fields, function bodies and constants, so it needs no arrow.
  No non-test Go file is listed by path while its Intent names symbols; card 5's `tmuxkit.go` path is listed for its package doc.
- Test economy and granularity: card 1 adds no new top-level test.
  It reworks rows of `TestStartup_SkillLoadTurns`, `TestRunner_Start_TypesTheSegmentColorAfterStartup` and `TestStartup_SendFailuresTearDown`, and the fake changes (`playLoad`, `LateEnds`, `EndsAtColor`) serve those rows.
  `markStarted` and `turnStartedAfterSessionStart` each have a stated reason to exist.
  This is not a defect.
- CLI shape: `lyx loom fmt` carries an audience from the closed set.
  The operator audience is what the decision record's operator-command-index constraint asks for.
  The shape is not ruled out by `pattern/PATTERN-cli-cobra.md`.
- Verify coverage: `## verify:` runs `go test ./...` and `go test -tags integration ./...`.
  That covers every targeted package, including `tmuxkit` (untagged) and card 7's integration-tagged test.
  The `tmux` and `llm` tiers are only compiled, and the order matches the template.
