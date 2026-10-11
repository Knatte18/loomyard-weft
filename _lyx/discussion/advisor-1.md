# Advisor 1 notes — dbench-omsm

Board entry `dbench-omsm` (#335) has six items.
Sibling entries `dbench-ohsh`, `dbench-ohom`, `dbench-ohsm`, `dbench-n0fm`, `dbench-n0oh`, `dbench-n0om` and `dbench-solo` carry the identical title and brief; they are parallel benchmark copies, not distinct work.
`gate-test-hygiene` (#327), `audit-reader-globs` (#319) and `weaver` (#329) border the task without overlapping it.

## Modules per item

| Item | Module | Finding |
|---|---|---|
| 1 background gate shell | `internal/shuttleengine` (`wait.go`, `claudeengine/background.go`) | `countsAtOnce`/`expiredTurnEnd` count a turn end `Done` when every output file exists and every outstanding task is an unawaited shell (gated run, started fresh, no gated arrival yet). `background_shell_wait_min` is log-and-display only (`ShellWaitBound`). `logEndedShells` says "lyx kills no shell itself". Strand removal then ends the shell. The fix is either to hold the turn end for a gate shell (`AwaitedShellPrefixes` on the launch spec exists for this) or to record the discarded shell in the run record. |
| 2 Master fixes verify itself | `contracts/stencils/webster/webster-template-master.md`, `internal/websterengine` | The stencil already has "A gate failure: spawn one fixer fork" with "Do NOT localize or fix the failure yourself". The gap is where Master first reads the failure: the `Gate findings recorded at …` message text, which is Go-authored. A verb that spawns the fixer needs a Go change in `websterengine`. A stencil edit must stay a pointer, not a duplicate (`PATTERN-producer-pointer-rule`). |
| 3 recovery attribution | `internal/websterengine/recoverbatch.go` (`uncommittedPathsBlock`, `loadRunWrites`, `writtenWorktreePaths`), `webster-prefix-recovery.md` | Attribution is by the current run's write evidence, so a first recovery's files are "Not written by this run". Fix: union the writes of every earlier recovery run of the same batch, or start clean and offer the diff. Also `recover-batch` exit code when the batch ends failed and terminal (`webstercli`). |
| 4 recovery gates | `webster-prefix-recovery.md`, `doc.go` | Stencil text only, plus the `doc.go` sentence. |
| 5 Create check (#511) | `internal/planglyph/donecheck.go`, `create.go`, `internal/websterengine/recordbatch.go` | Quarry v0.2.1 (external module `github.com/Knatte18/quarry`) walks all declarations and has no exported-name filter, so unexported targets likely resolve. The cause is unproven: the code does not say whether #511 was a resolver limit or a missing symbol. The refusal text (`Create target %q still does not resolve`) carries no lookup detail, which is the stated gap. |
| 6 ambiguous per build tag/GOOS | `internal/planglyph/donecheck.go`, `create.go`, `resolve.go`; `internal/planparser/validate.go` | Quarry's `statusForMatches` deliberately does not evaluate build constraints ("the engine has no business interpreting"). So quarry-side honouring is an external-repo change plus a `go.mod` bump. The loomyard-side route is `doneCheckVerdicts`: treat `StatusAmbiguous` as resolved for Create/Rename-new when all `Candidates` sit in files with disjoint constraints. This needs a constraint reader for `//go:build` lines and `_GOOS`/`_GOARCH` filename suffixes (stdlib `go/build/constraint` plus `go/build`'s name matching), and the candidates must expose a file path (not verified). Delete/Rename-old must keep blocking on ambiguous (`stillExists`). The Plan-time flag belongs in `planparser` validate or `planglyph`'s resolve findings (`glyph-ambiguous` today). |

## PATTERN entries touched

- `PATTERN-refusal-way-forward`: any new or reworded webster refusal (item 5, item 3 exit code) names the way forward, lands its spec row in the same commit, and has a test reaching it.
- `PATTERN-no-denied-recovery`: item 4's prompt must not name a command the agents' settings deny; check that `lyx gate test` and `lyx loom lint-comments` are not denied before naming them as allowed.
- `PATTERN-producer-pointer-rule`, `PATTERN-stencil-ownership`: stencil edits point, not paraphrase; text is read from the stencil store, not embedded.
- `PATTERN-completion-signal`: item 1 touches the "did this run finish" answer; `allOutputFilesExist` must still be consulted first.
- `PATTERN-shuttle-provider-seam`: Claude-specific shell detection stays under `claudeengine`.
- `PATTERN-wait-mark-display-only`: the wait marker stays display-only; a held turn end must not be decided from `@lyx_wait`.
- `PATTERN-friction-capture`: a refusal in webster writes a Go-authored friction note.
- `PATTERN-sole-parsers`, `PATTERN-ref-shape-registry`: the Plan-time glyph flag (item 6) goes through `planparser`/`planglyph`, not a new parser.
- `PATTERN-shed-producer-seam`, `PATTERN-gate-self-check-parity`: apply if a new gate or verb is added.
- `PATTERN-test-economy`, `PATTERN-test-speed`: new tests extend existing ones; `cmd/lyx/testdata/test-budget.yaml` rows may need an `Edit:`.
- `CLAUDE.md` docs rule: `internal/websterengine/doc.go`, `internal/shuttleengine/doc.go`, `internal/planglyph/doc.go` and any spec row change in the same commit.

## Conflicts and open questions

- No direct PATTERN conflict seen with the six items.
- Item 2's second option ("a verb that spawns the verify fixer") needs the new verb to follow `PATTERN-cli-cobra` and the webster refusal rule; the stencil-only option is cheaper, but the stencil already states the rule, so a stencil-only fix may repeat what failed.
- Item 6 has two valid homes (quarry upstream vs loomyard DoneChecks); the code favours loomyard-side because quarry documents its refusal to read build tags.
- Item 5 cannot be closed without reproducing #511 against quarry v0.2.1; the code alone does not show the cause.
- Item 1's "hold the turn end" option interacts with the liveness check and the run deadline, which already bound the wait for runs with a missing output file.
