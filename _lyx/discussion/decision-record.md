# Decision record — intake-bugs-1012

## Goal

Close the board entry's four intake bugs, one per section of its body (the brief and title name three; the body's `plan-cli-tree-check` section adds the fourth):

- Gate steps never name an external test package as a directory (#571, #577, #580, #587).
- Settle the flaky `internal/lyxtemp.TestSweep/two_concurrent_sweeps_both_succeed` (#584).
- A loop stop on an absent or unreadable `status.json` is diagnosable from its halt note (#581).
- A plan that shapes a CLI command against the `cmd/lyx/clitree_test.go` rules is caught before implementation (#588).

## Scope

In:

- `internal/planparser`: the ref-to-directory mapping behind `CardTargetDirs`, and one new plan check over author-written verify chains.
- `internal/gatecli`: a refusal for a relative package argument that names no directory.
- `internal/shedverbs`: the loop's status read at the top of each iteration, its stop cause, way forward and trace file.
- `PATTERN.md` and `pattern/PATTERN-glyph-conversion-chokepoint.md`: the one sanctioned external-test mapping.
- `pattern/PATTERN-cli-cobra.md`, `docs/overview.md`'s audience paragraph and the Plan-Review rubric `contracts/stencils/loom/loom-rubric-plan-review.md`: the runnable-group audience rule.
- Docs of every touched package (`doc.go`), `contracts/specs/loom-plan-spec.md` for the new check, and `contracts/specs/refusal-spec.md`'s `lyx gate test` table for the gate refusal.

Out:

- Any change to quarry: its glyph grammar, `Glyph.UnitPath` and the external-test pseudo-unit convention stay as they are.
- A mechanical plan check for CLI shapes: card prose carries no `Use`, flags or annotations to check.

## Decisions

### D1 — An external-test glyph unit maps to its package directory

Quarry names a Go external test package's unit `<dir>_test`, a pseudo-path with no directory, and `Glyph.UnitPath` returns it unchanged.
`planparser`'s ref-to-directory mapping passes that on as a directory, so a card targeting `internal/x_test#TestY` yields the gate argument `./internal/x_test` in Webster's derived card and batch gates.

Decision: `internal/planparser` holds one helper that takes a Go unit string, either a glyph's `UnitPath` result or a handle's package half, and, when its last path segment ends in `_test` and has no file extension, returns it with that suffix removed.
The ref-to-directory mapping applies it to every glyph-derived unit, and `vocabname.go`'s existing `strings.TrimSuffix(unit, "_test")` is replaced by a call to it.
Path refs and file units are untouched.

Both production readers of `CardTargetDirs` change with it: Webster's `gateScopeOf` and `internal/batcher`'s read-set estimate, where an `internal/x_test` entry becomes `internal/x` and merges with an existing one.

Bound: the mapping is disk-free and unconditional, so a real Go package directory whose own name ends in `_test` maps to its sibling directory; this repository has none.

Rationale: the derived gate is where every reported bad path came from, and one mapping fixes every card's gate at once.

### D2 — The glyph chokepoint gains one sanctioned site

`PATTERN-glyph-conversion-chokepoint` says loomyard performs no glyph-to-path conversion of its own; D1's helper is one, and `vocabname.go`'s trim was one already without saying so.
Decision: the PATTERN.md one-liner adds the exception (for example "…plus `planparser`'s one external-test unit mapping"), and the background names D1's helper as the single sanctioned adjustment of a Go unit string, for the external-test pseudo-unit only, citing quarry's `docs/glyph.md` corner case.

Rationale: quarry documents the pseudo-unit as having no directory, so some caller must map it, and the inlined one-liner is what spawned agents read.

### D3 — New plan check `verify-package-missing`

Decision: a new `internal/planparser` check reads each card's `**Verify:**` value and the overview's `## verify:` chain, with the chain splitting, env-assignment skipping and `cd`/`-C` tracking `verify-nested-module` uses.
For `lyx gate test` and for `go test`, `go build` and `go vet`, each package argument spelled relative (`.`, `..`, `./…`, `../…`) and not ending in `.go` is joined onto the command's directory with any trailing `/...` removed.
It is a finding when that directory neither exists on disk under the worktree root nor holds a Create target or Rename destination of a card numbered at or below the Verify's card (the whole plan for the overview chain).
The finding names the card and the argument; when the argument ends in `_test` and the trimmed directory exists or is created, the detail names it as the replacement.
Its way forward is to correct the Verify line.
It runs in `Validate` and `ValidateFormat` after `verify-module-wide` and is blocking; dispatch already drops a finding against a completed card.

Bound:

- Static spellings only: an argument behind a variable, `bash -c`, after an unplaceable `cd`, an import path, a module-wide pattern or a `.go` file argument passes.
- A Verify on or after a card with a Move group is not checked, since a Move's destination lives in Intent prose.
- A directory an earlier card deletes is not reported; the card's own verify run catches it.

Rationale: author-written Verify lines and the overview chain are the remaining route for a non-directory argument after D1.
The `go` commands share the parsing and fail the same way, so they are covered by the same rule.

### D4 — `lyx gate test` refuses a relative package argument that names no directory

Decision: after resolving its directory (the `-C` value, or the seam cwd, never `os.Getwd`) and before any slot path, inherited, slotted or outside every hub, and before the prebuild, `lyx gate test` checks each package argument spelled relative and not ending in `.go`, with any trailing `/...` removed.
When one is not an existing directory, it exits 1 with a JSON error naming every such argument.
For an argument ending in `_test` whose trimmed directory exists, the error says it is an external test package whose tests live there and names the corrected argument.
The way forward is to re-run the same command with corrected arguments.
`refusal-spec.md` gains the row, classed like the no-package row.

Bound: import paths, absolute paths, patterns and `.go` file arguments pass to go test unchanged; a refused request runs no package.

Rationale: the issues ask for the missing directory to be named; refusing before the slot also saves the wait and the prebuild of a run that cannot pass.

### D5 — The loop tells an absent status from an unreadable one and points at its own log

Premise: `status-missing` is set by `internal/shedverbs`' loop when `readStatusAt` reports false at the top of an iteration, which it does for an absent file and for any read or decode error alike; no child step runs, so there is no history entry and no trace file.
The #581 halt most likely came from an older-binary loop's strict decode of a `routed_to` history field; `e5b56fe91`, in this branch's base, made `internal/state` reads tolerate unknown fields.

Decision:

- An absent file keeps the cause `status-missing`; a read or decode error stops with `status-unreadable: <error>`, the same `<class>: <detail>` form as `start-failed: <error>`.
- At either cause the loop writes a `logger.Warn` naming the status path and, when unreadable, the error.
- The trace file handed to `Hooks.AfterInterrupt` and to the interrupted record is the loop's own log, `spec.Loop.LogPath`, as `loop-exited` already does.
- Their way forward names the status path and the loop log, and re-running `lyx shed step <run> --until-stop` once the file reads, instead of the generic re-run.
- No re-step: the only writer, `internal/state`, writes by atomic rename under the status lock.

Bound: `shedengine.WriteFailedStop` cannot write a failed stop over an absent or unreadable file, so at these two causes the status is not made non-running, as today; overwriting it would destroy the evidence.
This is a stated departure from `PATTERN-failure-stop-visible` for the case where no status can be read.
The halt note's history stays empty, since the hook re-reads the file.

Rationale: the halt note then says what failed and where to read more, which is what #581 asks for.

### D6 — No lyxtemp change

On current code, `lyx gate test ./internal/lyxtemp -- -race -run TestSweep -count=1000` and the same with `-count=300` and no `-race` pass.
Decision: no change to `internal/lyxtemp`; the commit message closes #584 citing `78e31aa06`'s bounded retry and these two runs.

Rationale: the board entry asks to check reproduction on the bounded-retry code first, and it does not reproduce.

### D7 — The runnable-group audience rule is written where plans read it

Decision:

- `pattern/PATTERN-cli-cobra.md`'s audience bullet states `clihelp.Invocable`'s rule: a command with subcommands is invocable only when it is runnable and its `Use` carries a token after its name or it has a visible local flag.
  Any other group carries no audience, prints help on a bare call and refuses a mistyped subcommand, so behaviour meant for the group name belongs on a leaf such as `show`.
- `docs/overview.md`'s audience paragraph points at that bullet.
- The Plan-Review rubric gains one check that points at that bullet for any card adding or reshaping a CLI command, without restating it.

Rationale: no mechanical plan check can read a command's shape from card prose, and the rule lived only in tests and code.

## Constraints

- `PATTERN-glyph-conversion-chokepoint`: changed by D2.
- `PATTERN-failure-stop-visible`: departed from by D5's bound, for an unreadable or absent status only.
- `PATTERN-cwd-resolution`: D4's base directory comes from gatecli's seam cwd.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: D3's finding, D4's refusal and D5's way forward each name a step an agent may run.
- `PATTERN-producer-pointer-rule`: D7's rubric line points at the background.
- D5's cause keeps the `<class>: <detail>` form; `convergence-plan` classes halt reasons by their leading clause.
- Tests: unit tier, extending existing tables where one fits; a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- Docs in the same commit: `internal/planparser/doc.go`, `internal/batcher/doc.go` if its estimate text names test directories, `internal/gatecli/doc.go`, `internal/shedverbs/doc.go`, `contracts/specs/loom-plan-spec.md`.

## Auto-mode assumptions

- D1's unconditional mapping suffices, since no Go package directory in this repository ends in `_test`.
- D5's root-cause attribution is inferred from timing and the `routed_to` field, not from the lost run's files; D5's change holds whatever the cause was.
- D6 treats a green 1000-iteration race-detector loop as sufficient evidence.

## Open risks

- A `TestSweep` flake that needs heavier machine load than the loop ran under may remain.
- D4 turns a hand-written gate line with one wrong argument from a partial run into a refusal.

## Acceptance criteria

- A card whose target is a member glyph on `internal/x_test` yields card and batch gate steps naming `./internal/x`, never `./internal/x_test`; a card targeting both `internal/x#…` and `internal/x_test#…` names `./internal/x` once, in the gates and in the batcher's estimate.
- `vocabname.go`'s owner check behaves as before for an external-test unit.
- `verify-package-missing` reports: a card Verify `lyx gate test ./internal/x_test` with the `./internal/x` hint; `go test ./nope` in the overview chain; an argument under `-C <module>` resolved against that module; a directory created only by a later card.
  It reports nothing for: an existing directory; a directory the same or an earlier card creates a target in; `./...`; an import path; a `.go` file argument; an argument after an unplaceable `cd`; a Verify on or after a Move card.
- `lyx gate test ./internal/x_test` refuses with exit 1 before any slot or prebuild, naming `./internal/x`; `./nope ./internal/x` refuses naming `./nope` only; `-C <dir>` arguments resolve against that directory; `./internal/...`, an import path and a `.go` file argument reach go test.
- The shed loop stops with `status-missing` when `status.json` is absent and `status-unreadable: …` when it holds invalid JSON, runs no child, records the loop log path as the trace file and leaves the file as it found it.
- A `status.json` carrying an unknown key is read normally and the loop steps.
- `lyx loom lint-pattern` passes with D2's and D7's PATTERN text.

## Notes for the plan writer

- D3 can reuse `splitChain`, `isEnvAssignment`, `resolveRunDir` and `goCommandDirAndPackages` in `internal/planparser/verifymodule.go`, and the create and rename unions behind `path-missing`, restricted by card number.
- D5's stop is in `internal/shedverbs/loop.go` (`runLoop`, `readStatusAt`, `interruptedError` in `loopenvelope.go`); loom's hook is `loomAfterInterrupt` in `internal/loomcli/looparm.go`.
- `cmd/lyx/constraintchokepoint_test.go` scans only `#`-trimming `TrimSuffix`, so D1's helper needs no exemption there.
