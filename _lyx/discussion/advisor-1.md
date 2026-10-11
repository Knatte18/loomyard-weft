# Advisor 1 notes — dbench-ohsh (Webster stability)

Read from the code at HEAD `800a0e892`; the evidence for each point is the named file or symbol.
The issue texts (#500, #505, #507, #509, #511, #517, #522) were read with `gh issue view`, read-only.

## Module boundaries

| Item | Owner | Neighbours |
| --- | --- | --- |
| 1 gate shell cut off | `internal/shuttleengine` (`wait.go` `expiredTurnEnd`, `countsAtOnce`) | `internal/websterengine/runlevel.go` sets the Master `Spec` (`AwaitedShellPrefixes`); `claudeengine` builds `BackgroundTask` values from the Stop payload |
| 2 Master fixes verify itself | `shuttleengine/gate.go` `gateRepromptText` (generic text), `websterengine/verifygatereport.go` `renderVerifyGateFindings` | `contracts/stencils/webster/webster-template-master.md` |
| 3 recovery attribution | `websterengine/recoverbatch.go` `uncommittedPathsBlock`, `runwrites.go` `loadRunWrites`/`recordedSessions` | `state.go` `BatchState`; `webstercli/recoverbatch.go` for the exit code |
| 4 allowed gates | `contracts/stencils/webster/webster-prefix-recovery.md` + `webster-body-implementer.md` | `renderCardGates`/`renderBatchGate` in `cardgate.go`; deny table in `claudeengine/settings.go` |
| 5 Create check refusal | `internal/planglyph/donecheck.go` (`DoneChecks`, `doneCheckVerdicts`) | caller `websterengine/recordbatch.go` `postBatchChecks`; text surfaces as `batch_failed` |
| 6 build-tagged members | `planglyph` (`donecheck.go`, `resolve.go` `statusFindings`, `create.go`) | quarry v0.2.1 (external module, `internal/engine/resolve.go` `statusForMatches`) |

## What the code says about each item

### 1. Gate shell
- The residual fault is narrower than the issues say.
  Since `5a8be0274` (2026-10-10 16:47, after the three issues were filed) a waiting turn end never expires, and the ended shells are recorded (`Result.EndedShells`, friction note, `RunResult.Warnings`, summary section).
- What remains: Master's run is gated (the verify gate), and `countsAtOnce` counts a turn end at once when the output files exist, the run started fresh, no gated arrival happened yet, and every outstanding task is an unawaited shell.
  `lyx gate test --tags integration` is an unawaited shell; only the `lyx webster recover-batch` prefix (`masterAwaitedShellPrefix`) is awaited.
  So the run arrives at the gate, finishes `done`, and shuttle removes the strand with the shell in it.
- Smallest fix: add a gate prefix to `Spec.AwaitedShellPrefixes` in `runlevel.go`.
  A prefix match is on `BackgroundTask.Label`, which is the command string, so `cd x && lyx gate test` or an env-prefixed form would not match.
  The Master stencil would then have to state the form (a bare `lyx gate test ...` first in the command), or the match would need to be a contains-match.
- "lyx stops the shell explicitly" is not buildable provider-neutrally: `logEndedShells` documents that lyx kills no shell, and shuttle holds no pid for a Bash-tool child.
  `internal/proc` child-lifetime work (board `proc-child-lifetime`) is about lyx's own children, not Claude's.
- Holding is bounded by `master_timeout_min` (480) and the liveness check, so a hung gate shell ends the run `OutcomeTimeout`.
- Stale prose to update with the code: `websterengine/doc.go` "Background shells in Master's wait", `shuttleengine/doc.go` near the `BackgroundShellWaitMin` paragraph, `shuttleengine/config.go` comment.

### 2. Master fixes a verify failure
- The stencil already says "Do NOT localize or fix the failure yourself" (section "A gate failure: spawn one fixer fork").
- The text Master reads at the moment of failure says the opposite: `gateRepromptText` sends `Gate findings recorded at <path> — read it, fix every finding, and end your turn.`
  Its findings file for a dirty tree says "Commit or remove these paths:" (`renderVerifyGateFindings`).
  Both contradict the stencil, so the right path is not the easy one.
- A verb that spawns the fixer is not buildable: the fixer is an in-session `fork` made through Master's Agent tool, which Go cannot call.
  A cold fixer strand would be a different design (recovery-shaped).
- Recommendation: state the rule in the text Master reads.
  Give `shuttleengine.GateEntry` an optional re-prompt text (generic default unchanged for the loom gates), set by the webster verify gate entry in `runlevel.go`; and open `renderVerifyGateFindings` with the same rule.
  The dirty-tree branch needs its own wording, since the worktree-clean failure is not fixable by a fork the same way; decide what Master does there.
- Mind `PATTERN-parent-directive` / `PATTERN-no-denied-recovery`: the new wording must not name a denied command, and no stencil may tell an agent to ask the operator.

### 3. Second recovery sees the first's files as foreign
- `uncommittedPathsBlock` classes a path as "Written by this run" only if some session in `recordedSessions(st)` wrote it.
  That set is `State.MasterSessionID` plus each batch's `SessionID`, which `recoverSpawn` leaves empty ("Empty for a recovery batch").
  So a recovery strand's own writes are never evidence, for the first or any later recovery.
- `recoverSpawn` builds a fresh `BatchState` per spawn; `ShuttleRunDir`/`StrandGUID` of the prior recovery are overwritten, so the first recovery's session id is lost at the respawn.
- Two fixes, both batch-scoped:
  1. Carry the prior recovery's session (or run dir) forward on `BatchState` and add it to `recordedSessions`; `AuditForks(session, worktree)` then yields its `ParentWriteEvents`.
     Needs the recovery session id from the run's `run.json` (`FindRun` returns it as `runState.SessionID`; `recoverSpawn` already calls `FindRun` and discards the id).
  2. Carry the foreign set computed at the first spawn and treat any later-dirty path outside it as the batch's own; no transcript read.
- Recommendation: option 1, since it reuses the existing evidence reader and the "unattributable counts as foreign" rule stays intact for real strays.
  Option 1 needs a state-record field; `state.json` is read by older runs, so the field must be `omitempty`.
- `recover-batch` exit code: a terminal digest of status `stuck`/`dead` returns `output.Ok` (exit 0) by design; Master's ladder reads `status` from that envelope.
  Only `ErrBatchFailed` exits non-zero.
  The code cannot say which of the two the quick-recipe run hit ("ended failed and terminal"); ask the operator for the envelope before changing the contract, because making `stuck`/`dead` non-zero changes the Master ladder wording too.

### 4. Allowed gates in the recovery prompt
- The recovery prompt is `webster-prefix-recovery.md` + `webster-body-implementer.md`.
  The body's "sandbox rule" bans mutating `lyx fabric` verbs and its "slot-busy" section routes module-wide and tmux-tier runs through `lyx gate test`, but nothing says `lyx gate test` and `lyx loom lint-comments` are not fabric commands.
- `lyx gate test` is the sanctioned route (`steerRawGoDeny` in `claudeengine/settings.go` names it), so naming it as always allowed complies with `PATTERN-no-denied-recovery`.
- A tmux-tier card step: the stencil can say it is left to verify; check that this holds for the fork implementer too, since both share the body.
- Stencils are read at call time (`PATTERN-stencil-ownership`); `contracts/stencils/claims_test.go` pins phrases and will need a row for each new claim.

### 5. Create check on unexported targets
- Quarry does not filter by export or by test file: `symbolsOfDir` collects every file in the unit directory, `_test.go` included, and `matchesFor` compares owner and name only.
  So the unexported/test-file hypothesis of #511 is not supported by the resolver.
- The two cited targets are explained by build constraints: `tmuxAllowed` is a pair of `const`s under `tmux || llm` and `!tmux && !llm` (#500), and `rusageStats` now sits in `cmd/testtiming/resources.go` with `rusage_linux.go` / `rusage_other.go` beside it, so a per-GOOS split during the run is plausible.
  I could not confirm the `rusageStats` shape at the time of the run from the tree.
- The refusal text today: `Create target %q still does not resolve` (`donecheck.go`), with no lookup detail.
  Changing it means: name the resolve key, the status quarry answered (`not_found`, `ambiguous` with candidate files, a pre-resolution rejection) and the unit searched.
  That is a webster refusal, so `PATTERN-refusal-way-forward` applies: the message names the way forward, the `refusal-spec.md` row lands in the same commit, a test reaches it.

### 6. Build-tagged and per-GOOS members
- Quarry says on purpose it does not evaluate build constraints (`statusForMatches` doc: the answer would depend on a GOOS/GOARCH the caller did not state).
  `ResolveResult.Candidates` carries each match's `File`, so `planglyph` can decide without a quarry change.
- Recommendation: one predicate in `planglyph` — "every candidate pair sits in files whose build constraints are mutually exclusive" — read from each candidate file's `//go:build` line and `_GOOS`/`_GOARCH` filename suffix, via `go/build/constraint`; disjointness by enumerating assignments of the (small) tag set.
  It must feed every consumer, or the next gate fails instead:
  - `DoneChecks` Create (`create-not-done`), the item named in the task.
  - `statusFindings` (`glyph-ambiguous`), which blocks Edit/Delete by glyph of a per-GOOS symbol at Plan time, and runs again in `begin-batch`'s re-resolution of the plan; a plan that passes DoneChecks but trips begin-batch for the next batch is worse than today.
  - `createFindings` (`create-already-exists` for ambiguous), and the #507 corner where the same card deletes every declaration and creates the name again.
  - `DoneChecks` Delete (`delete-not-done`) stays as is: ambiguous still means declarations exist.
- `BindHandles` already tolerates two declarations (it matches `delta.Created` by ID as a set), so binding is not a blocker.
- "The plan validator flags such a glyph at Plan time": a Create of a member that does not exist yet resolves `not_found`, so the tree cannot reveal that the implementation will split by tag.
  Plan time can flag only what is visible: the card text, or an existing ambiguous glyph whose candidates are not provably disjoint.
  This sentence of the board entry needs the operator's reading before the record fixes it.
- Quarry option (honour constraints) means a new quarry release and a `go.mod` bump, and needs a tag set to evaluate against; not recommended.

## PATTERN entries touched

- `PATTERN-completion-signal`: any change to `wait.go`'s negative-verdict returns is pinned by an AST tripwire; holding a turn end on a shell is not a negative verdict, but a new exit would fail the scan and need the audited list updated.
- `PATTERN-shuttle-provider-seam`: prefix matching on `BackgroundTask.Label` stays provider-neutral; a Claude-specific label shape must stay under `claudeengine`.
- `PATTERN-refusal-way-forward`: items 3 (exit/refusal wording) and 5 (create-not-done text).
- `PATTERN-no-denied-recovery`, `PATTERN-parent-directive`, `PATTERN-stencil-ownership`: items 2 and 4 (stencil and gate-message wording).
- `PATTERN-test-economy`: add cases to existing tests where possible.
- `PATTERN-documentation-lifecycle` and CLAUDE.md "Docs land in the same commit": `websterengine/doc.go`, `shuttleengine/doc.go`, `planglyph/doc.go`; a cross-cutting invariant goes into `PATTERN.md` in the same commit (none is clearly new here).
- `PATTERN-leaf-packages`/`PATTERN-told-geometry`: the build-constraint reader reads files under the worktree root `DoneChecks` is already handed; it does not import `lyxcwd`.

## Board entries touching this task

- `gate-test-hygiene` (running, Webster): edits `cmd/lyx/testdata/test-budget.yaml` and the `PATTERN.md` overview size cap.
  This task should add no `PATTERN-` line, and should not raise a budget row that task also raises (the collision it is fixing).
- `per-role-models` (running, Plan-Write): config and loom; no file overlap seen.
- `audit-reader-globs` (Ready): webster audit classifier; no overlap.
- `proc-child-lifetime` (Ready): relevant only if the operator wants lyx to stop the gate shell; it covers lyx's own children, not Claude's.
- `kick-start-pack`, `enclose-kickstart` (Ready, quarry): adjacent to quarry resolution, no overlap.
- Test-budget rows are close: `planglyph` 83, `websterengine` 199, `shuttleengine` 161, `webstercli` 81 against my count of top-level tests of 84, 200, 159, 82 (my count differs from the row, so use the budget test itself).
  A card adding top-level tests to these packages lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.

## Conflicts between the task and PATTERN

None found.
Two task sentences are not buildable as written: "lyx stops the shell explicitly" (item 1) and "a verb that spawns the verify fixer" (item 2).
One is unclear: "plan validator flags such a glyph at Plan time" (item 6).
