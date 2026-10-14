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
- Docs of every touched package (`doc.go`), `contracts/specs/loom-plan-spec.md` for the new check, and `contracts/specs/refusal-spec.md`'s `lyx gate test` table for the gate refusal and its shed section's `step interrupted` row and new `loop status missing or undecodable` row for D5's stop.

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
It is a finding when that directory neither exists on disk under the worktree root nor is, or is an ancestor of, a created directory.
The created directories are the entries `CardTargetDirs` yields with `DeleteOnly` false for each card numbered at or below the Verify's card (the whole plan for the overview chain), so a path or glyph target of any non-Delete group, member glyphs and Custom groups included, and a Rename destination each count.
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

Decision:

- An absent file keeps the cause `status-missing`; a decode error stops with `status-unreadable: <error>`, the same `<class>: <detail>` form as `start-failed: <error>`.
- A read-lock or raw read error is transient: the loop re-reads once under its existing single re-step, and stops with `status-unreadable: <error>` only when the re-read fails too.
- At either cause the loop writes a `logger.Warn` naming the status path and, when unreadable, the error.
- The trace file handed to `Hooks.AfterInterrupt` and to the interrupted record is the loop's own log, `spec.Loop.LogPath`, as `loop-exited` already does.
- Their way forward names the status path and the loop log, instead of the generic re-run, and is split by cause.
  At `status-missing` it names the recipe's own reseed, as refusal-spec's `status file missing` row gives it: `lyx loom start` for loom, `lyx batten run <slug>` for batten, `lyx shed seed` otherwise.
  At `status-unreadable` from a read-lock or raw read error that failed its re-read, it names re-running `lyx shed step <run> --until-stop`.
  At `status-unreadable` from a decode error, it names re-stepping with a binary that decodes the file, as after a version skew where an older loop binary cannot decode a file a current binary reads;
  otherwise it names reporting the stop to the run's parent with the status path and the loop log, as refusal-spec's `arbiter: status or envelope unreadable` row does.
- No re-step on a decode error: the only writer, `internal/state`, writes by atomic rename under the status lock, so a file that fails to decode stays undecodable to the binary that failed.
- `contracts/specs/refusal-spec.md`'s shed section changes in two rows, each stating that at its causes the status file is left as found rather than written `failed`, and giving its cause's way forward above.
  The `step interrupted` row, transient, names `status-unreadable` from a read error that failed its re-read.
  A new `loop status missing or undecodable` row, a correctness halt like the `status file missing` row because neither cause clears by itself or by a retry, names `status-missing` and `status-unreadable` from a decode error.

Bound: `shedengine.WriteFailedStop` cannot write a failed stop over an absent or unreadable file, so at these two causes the status is not made non-running, as today; overwriting it would destroy the evidence.
This is a stated departure from `PATTERN-failure-stop-visible` for the case where no status can be read.
The halt note's history stays empty, since the hook re-reads the file.
A file that later reads `running` is resumed by the way forward's re-step, not by `lyx loom resume`, which answers that the run is already running while a driver is live; `lyx loom arbiter` refuses either stop under its `status or envelope unreadable` row.

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
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: D3's finding, D4's refusal and D5's way forward each name a step an agent may run, and refusal-spec's gate table row and shed `step interrupted` and `loop status missing or undecodable` rows change in the same commit.
- `PATTERN-producer-pointer-rule`: D7's rubric line points at the background.
- D5's cause keeps the `<class>: <detail>` form; `convergence-plan` classes halt reasons by their leading clause.
- Tests: unit tier, extending existing tables where one fits; a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- Docs in the same commit: `internal/planparser/doc.go`, `internal/batcher/doc.go` if its estimate text names test directories, `internal/gatecli/doc.go`, `internal/shedverbs/doc.go`, `contracts/specs/loom-plan-spec.md`.

## Auto-mode assumptions

- D1's unconditional mapping suffices, since no Go package directory in this repository ends in `_test`.
- The #581 root cause recorded in support-log.md Q5 is inferred from timing and the `routed_to` field, not from the lost run's files; D5's change holds whatever the cause was.
- D6 treats a green 1000-iteration race-detector loop as sufficient evidence.

## Open risks

- A `TestSweep` flake that needs heavier machine load than the loop ran under may remain.
- D4 turns a hand-written gate line with one wrong argument from a partial run into a refusal.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d5-decode-error-way-forward-waits-on-unhealing-file (class decision, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A card whose target is a member glyph on `internal/x_test` yields card and batch gate steps naming `./internal/x`, never `./internal/x_test`; a card targeting both `internal/x#…` and `internal/x_test#…` names `./internal/x` once, in the gates and in the batcher's estimate.
- `vocabname.go`'s owner check behaves as before for an external-test unit.
- `verify-package-missing` reports: a card Verify `lyx gate test ./internal/x_test` with the `./internal/x` hint; `go test ./nope` in the overview chain; an argument under `-C <module>` resolved against that module; a directory created only by a later card.
  It reports nothing for: an existing directory; a directory the same or an earlier card creates a target in; a Create card targeting `internal/new/x.go#Foo` followed by a Verify on `./internal/new`; a Custom card targeting `internal/new/x.go` followed by a Verify on `./internal/new`; a Rename card whose pair's New side lies in `internal/new` followed by a Verify on `./internal/new`; `./...`; an import path; a `.go` file argument; an argument after an unplaceable `cd`; a Verify on or after a Move card.
- `lyx gate test ./internal/x_test` refuses with exit 1 before any slot or prebuild, naming `./internal/x`; `./nope ./internal/x` refuses naming `./nope` only; `-C <dir>` arguments resolve against that directory; `./internal/...`, an import path and a `.go` file argument reach go test.
- The shed loop stops with `status-missing` when `status.json` is absent and `status-unreadable: …` when it holds invalid JSON, runs no child, records the loop log path as the trace file and leaves the file as it found it; the `status-missing` way forward names the reseed and the decode-error `status-unreadable` one a re-step with a binary that decodes the file, or else reporting the stop to the run's parent with the status path and loop log.
  A read error that clears on the re-read lets the loop step on; one that fails the re-read stops with `status-unreadable: …` and a way forward naming the re-step.
  refusal-spec's `step interrupted` row, transient, carries the read-error cause, and its `loop status missing or undecodable` row, a correctness halt, carries `status-missing` and the decode-error cause.
- `PATTERN.md`'s `PATTERN-glyph-conversion-chokepoint` line names `planparser`'s external-test unit mapping, and its background names D1's helper as the single sanctioned adjustment.
- `pattern/PATTERN-cli-cobra.md`'s audience bullet states the runnable-group rule, `docs/overview.md`'s audience paragraph points at it, and the Plan-Review rubric carries a CLI-shape check pointing at it.
- `lyx loom lint-pattern` passes with D2's and D7's PATTERN text.

## Notes for the plan writer

- D3 can reuse `splitChain`, `isEnvAssignment`, `resolveRunDir` and `goCommandDirAndPackages` in `internal/planparser/verifymodule.go`.
- D5's stop is in `internal/shedverbs/loop.go` (`runLoop`, `readStatusAt`, `interruptedError` in `loopenvelope.go`); loom's hook is `loomAfterInterrupt` in `internal/loomcli/looparm.go`.
- `cmd/lyx/constraintchokepoint_test.go` scans only `#`-trimming `TrimSuffix`, so D1's helper needs no exemption there.
