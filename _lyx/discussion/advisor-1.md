# Advisor 1 notes — Webster stability (twb-seat-lead-1)

All code facts below are read at base `800a0e892` (`git show 800a0e892:<path>`), per the board entry's bench-base section.
No other board entry owns this task's items.
`fork-wait-noise` (356) and `audit-reader-globs` (319) border it and are not part of it.

## Findings that change the task's framing

- **Item 1 is already half-fixed at base.**
  `internal/shuttleengine` states "a waiting turn end never expires": `background_shell_wait_min` is a log and display threshold only (`wait.go` `ShellWaitBound`, `expiredTurnEnd`; `template.yaml`; `doc.go`).
  The stop-after-N-minutes behaviour the entry describes is the old code, which the corpus case `claudeengine/testdata/corpus/master-fork-running/case.yaml` records as `old_reading`.
  What survives at base is `countsAtOnce` in `wait.go`:
  a gated run (Master is gated by the verify gate) whose output files all exist, started fresh, with no gated arrival yet and only unawaited shells outstanding, finishes Done at once.
  An ungated run with the files present finishes Done at once whatever is outstanding.
  A shell counts as awaited only when its label starts with one of `Spec.AwaitedShellPrefixes`, and Master declares exactly one, `masterAwaitedShellPrefix = "lyx webster recover-batch"` (`websterengine/runlevel.go`).
  A background `lyx gate test --tags integration` shell therefore reads as unawaited, the turn end counts, `Result.EndedShells` records the shell, and strand removal kills it.
- **Item 5's two named targets are probably item 6's cause, not a resolver gap for unexported or test-file names.**
  Quarry v0.2.1 (`internal/engine/resolve.go`, `symbolsOfDir`, `units.go`) reads every `.go` file in the unit directory, `_test.go` included, and keeps unexported names; an external `_test` package file belongs to the `<unit>_test` unit.
  `rusageStats` is declared per platform (`cmd/testtiming/rusage_linux.go`, `rusage_other.go` with `//go:build !linux`), `tmuxAllowed` in `allow_tmux.go` (`tmux || llm`) and `allow_none.go`.
  Both resolve `ambiguous`, which `doneCheckVerdicts` does not accept for Create.
  The chair should ask whether the entry's "find whether the check resolves unexported identifiers" can be answered "yes, it does" and the real fix is item 6 plus a clearer refusal.
  The code does not show that every instance of #511 is this cause; I have not seen the failing runs.
- **Quarry is an external module** (`github.com/Knatte18/quarry v0.2.1` in `go.mod`, no `replace`).
  Its `statusForMatches` doc says it deliberately does not evaluate build constraints, because the answer would depend on a GOOS the caller did not state.
  "Quarry honours build constraints" is therefore outside this repo and against quarry's stated design.
  The in-repo route is a lyx-side predicate over `ResolveResult.Candidates`, each of which carries `File`.
- **The verify-gate findings text itself tells Master to fix the failure.**
  `shuttleengine/gate.go` `gateRepromptText` sends "Gate findings recorded at <path> — read it, fix every finding, and end your turn." to every gated producer.
  `websterengine/verifygatereport.go` `renderVerifyGateFindings` opens with "Verify gate failed (attempt N of M)." and, for a dirty tree, says "Commit or remove these paths:".
  The Master stencil says "Do NOT localize or fix the failure yourself" and spawn one fixer fork, but only in `webster-template-master.md` under "A gate failure: spawn one fixer fork" and in the NEVER-edit rules list.
  Both texts Master reads at the moment of failure push the other way.
- **A recovery strand's own writes are never attributed.**
  `websterengine/runwrites.go` `loadRunWrites` audits `State.MasterSessionID` plus each batch record's `SessionID`.
  `recoverSpawn` builds a fresh `BatchState` with no `SessionID` ("Empty for a recovery batch"), and the recovery runs as a separate shuttle session.
  So a second recovery cannot see the first recovery's writes; `uncommittedPathsBlock` puts all of them under "Not written by this run".
  The fix shape is evidence-based attribution by batch: record each recovery spawn's session id on the batch record and carry the list across respawns, as `Recoveries` and `ForkTranscripts` are carried, then let `recordedSessions` include them.
  Risk: `AuditForks` errors on a missing parent transcript, so a recovery whose session left no transcript would fail `recoverSpawn` ("transient") unless that case is tolerated.

## Modules the task falls under

| Item | Module (boundary) | Neighbours |
|---|---|---|
| 1 | `internal/websterengine/runlevel.go` (Master spec), `internal/shuttleengine` (`wait.go`, `spec.go`, `doc.go`) | `claudeengine/background.go` holds all Claude payload shape (`background_tasks`, transcript fallback) |
| 2 | `contracts/stencils/webster/webster-template-master.md`, `websterengine/verifygatereport.go`, `shuttleengine/gate.go` | `contracts/stencils/claims_test.go` pins stencil sentences (it quotes ``Gate findings recorded at …``) |
| 3 | `websterengine/recoverbatch.go`, `runwrites.go`, `state.go`; `webstercli/recoverbatch.go` | `output` (`Ok` exits 0, `ErrFields` exits 1), `clihelp.SetExit` |
| 4 | `contracts/stencils/webster/webster-prefix-recovery.md` over `webster-body-implementer.md` | `websterengine/cardgate.go` renders the card and batch gates; the body already says tmux-tier runs go through `lyx gate test` |
| 5 | `internal/planglyph/donecheck.go`, `websterengine/recordbatch.go` (`failCardNotDone`) | `planglyph/create.go`, `resolve.go` read the same statuses at Plan time |
| 6 | `planglyph` (`doneCheckVerdicts`, `statusFindings`, `createFindings`), `planparser` validator | quarry (external), `planglyph/handle.go` `BindHandles` |

Item 6 spans three readers of one status, so one shared predicate is needed:
`doneCheckVerdicts` treats `ambiguous` as not-done for Create and still-exists for Delete and Rename-old;
`statusFindings` blocks any `ambiguous` as `glyph-ambiguous` at Plan time (so an Edit of a per-GOOS member is already blocked at Plan, not only at record-batch);
`createFindings` reports `ambiguous` as `create-already-exists`.
A Create of a not-yet-existing per-GOOS member cannot be detected at Plan time from quarry; only the plan's own file targets could show it.
I found no plan-validator rule about build constraints (`planparser`, `planglyph`, `loom-plan-spec.md` have none), so "the plan validator flags such a glyph at Plan time" is new design, not a tweak.

## PATTERN entries touched

- `PATTERN-stencil-ownership`, `PATTERN-producer-pointer-rule`: items 2 and 4 edit stencils; text is read at call time through `stencilstore`.
  A findings sentence authored in Go (item 2) is data in a Go-rendered file, not a stencil, but it sits near the line this entry draws; the chair should say which.
- `PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-no-denied-recovery`: the recovery prefix is a spawned role's opening stencil.
  Naming `lyx gate test` and `lyx loom lint-comments` as always allowed must match what the recovery strand's settings allow; I did not verify the settings, and no `gate` or `lint-comments` string appears in `fabricengine`, `webster/audit.go` or `claudeengine/settings.go` at base.
- `PATTERN-shuttle-provider-seam`: item 1 must stay provider-neutral; the awaited prefix is caller data on `Spec`, so adding a prefix is within the seam.
  "lyx stops the shell and records the discard" would need pid or task-stop knowledge the provider does not report ("lyx kills no shell, since the provider reports no pid"), so it would add a new seam capability.
- `PATTERN-completion-signal`: governs negative finalizations; item 1 is a positive Done, so it is not violated, but any change to `countsAtOnce` or `expiredTurnEnd` must keep the output-files-first order.
- `PATTERN-shuttle-stop`: a webster-side stop of a Master shell would go through shuttle's stop verb only.
- `PATTERN-refusal-way-forward`: item 3's exit code and item 5's refusal text are webster refusals; each needs a way forward, a `contracts/specs/webster-spec.md` row and a test.
  The Master stencil's failure ladder reads `recover-batch` output keys (`status`, `recovery_retry`, `batch_failed`, `needs_fresh`, ...), so any envelope change must keep those keys.
- `PATTERN-friction-capture`: a refusal that halts a run in webster writes a Go-authored friction note; changed refusal paths must keep that.
- `PATTERN-fabric-destruction-chokepoint`, `PATTERN-batten-bookend`: the "start the second recovery from a clean tree" option of item 3 deletes uncommitted work; any such deletion in a pair goes through the existing reset machinery, so attribution by batch is the cheaper route.
- `PATTERN-glyph-conversion-chokepoint`, `PATTERN-sole-parsers`, `PATTERN-ref-shape-registry`: a build-constraint predicate reads candidate file paths and `//go:build` lines; it must not convert glyphs or classify plan refs itself.
- `PATTERN-leaf-packages`: I did not read the background; if `planglyph` or `planparser` is a leaf, a new import (`go/build/constraint` is stdlib) needs checking against its closed set.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-testkit`, and `CLAUDE.md`'s test budget: new tests extend existing tables (`donecheck_test.go` has a status table with "create ambiguous is not landed"), and a package pushed past its row in `cmd/lyx/testdata/test-budget.yaml` puts that file under `Edit:`.
- Docs-in-same-commit (`CLAUDE.md`, `PATTERN-documentation-lifecycle`): `websterengine/doc.go` (awaited prefix, recovery attribution, exit code), `shuttleengine/doc.go` if the wait changes, `planglyph/doc.go`, `contracts/specs/webster-spec.md`, `contracts/specs/loom-plan-spec.md`.

No conflict between a PATTERN entry and the task text so far.
One tension to flag: item 2's second option, "Master has a verb that spawns the verify fixer", cannot be a Go-side spawn of an in-session fork, because forks are Claude Agent-tool calls Master makes; a verb could only print the exact call or start a cold strand like `recover-batch`, which is a different design.

## Open points for the chair

1. Item 1: hold versus discard.
   Holding means adding a `lyx gate ` prefix to Master's `AwaitedShellPrefixes`, bounded by `MasterTimeoutMin`.
   Prefix matching is brittle: the label is the raw command, so `cd <path> && lyx gate test …` is unawaited.
   Options: match a contained token instead of a prefix (a shuttle `Spec` semantics change), or require bare `lyx gate …` calls in the Master stencil.
2. Item 3's exit code: `recover-batch` already exits 1 on `batch_failed` (`output.ErrFields`), and exits 0 through `output.Ok(digestFields…)` for a terminal digest whose status is `stuck` or `dead`.
   The entry's "ended failed and terminal" most likely means those digests, but a non-zero exit there needs the ladder's `status`, `recovery_retry` and `recoveries` fields kept on the envelope (`ErrFields` keeps fields).
3. Item 6: which statuses an ambiguous-but-disjoint answer should be accepted for (Create done, Delete, Rename-old and new, plan-time `glyph-ambiguous`), and how disjointness is decided: `//go:build` expressions, the `_GOOS` and `_GOARCH` filename suffixes (`proc_linux.go`, `proc_windows.go` rely on the suffix alone), and tests files.
   `go/build/constraint` parses the expressions; disjointness is an unsatisfiability check over the tags both expressions mention.
   Not answered by the code: whether quarry's delta (`delta.Created`) reports one symbol or two for a per-GOOS Create, which `BindHandles` needs to bind the handle.
