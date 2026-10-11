# Advisor 1 notes — twb-combo-2 (Webster stability)

Code read at bench base `800a0e892` (`git show 800a0e892:<path>`), not HEAD.

## Bench hygiene

Before reading the board entry's "Bench base" paragraph I had not seen `1e79825ff`; after reading it I still ran `git show 1e79825ff --stat`, which printed that commit's message and file list.
I have not read its diff.
The chair should treat anything of mine that echoes a landed design as suspect; every claim below cites base code.

## Module boundaries the task falls under

| Part | Modules (base paths) |
|---|---|
| 1 gate shell cut off | `internal/shuttleengine` (`wait.go` `pollEventsTick`, `expiredTurnEnd`, `countsAtOnce`, `endedShells`; `spec.go` `AwaitedShellPrefixes`; `claudeengine/background.go` label/signal), `internal/websterengine` (`runlevel.go` `masterAwaitedShellPrefix`, `finishMasterDone`; `triage.go` shell warning/friction note; `summary.go` `AppendBackgroundShells`) |
| 2 Master self-fixes | `internal/shuttleengine/gate.go` (`gateRepromptText`), `internal/websterengine/verifygatereport.go` (`renderVerifyGateFindings`), `contracts/stencils/webster/webster-template-master.md`, `webster-body-verify-fix.md` |
| 3 recovery attribution, exit code | `internal/websterengine/runwrites.go` (`loadRunWrites`, `recordedSessions`), `recoverbatch.go` (`uncommittedPathsBlock`, `recoverSpawn`), `reset.go`, `contractevidence.go`, `state.go` (`BatchState`), `internal/webstercli/recoverbatch.go` |
| 4 recovery prompt gates | `contracts/stencils/webster/webster-prefix-recovery.md` + shared `webster-body-implementer.md`, `contracts/stencils/claims_test.go` |
| 5 Create check | `internal/planglyph/donecheck.go` (`DoneChecks`, `doneCheckVerdicts`), `create.go`; resolver is external `github.com/Knatte18/quarry` v0.2.1 |
| 6 ambiguous members | `planglyph` (`donecheck.go`, `create.go`, `resolve.go` `statusFindings`, `deleteorder.go`, `callercoverage.go`), plan validator `plangate.go`; quarry unchanged unless bumped |

## Findings from the code

### Part 1

- The "expires after `background_shell_wait_min`" behaviour in #505/#509/#517 is already gone at base: `expiredTurnEnd` only logs a shell past the bound (`logPayloadShellPastBound`), and `doc.go` says no shell expires.
- What remains at base: a gated run (Master always is, via the `verify` `GateEntry` in `runlevel.go` ~l.730) counts a waiting turn end Done at once when every output file exists, the run started fresh, no gated arrival happened yet, and every outstanding task is an unawaited shell (`countsAtOnce`).
  A `lyx gate test --tags integration` shell Master started itself is exactly that case; shuttle then finalizes, removes the strand, and the shell dies with it.
  `endedShells` lists it, `finishMasterDone` warns, `writeBackgroundShellFrictionNote` records it, and `AppendBackgroundShells` writes the summary section, but the outcome stays `done`.
- The existing hold mechanism is `Spec.AwaitedShellPrefixes`; Master declares only `lyx webster recover-batch` (`masterAwaitedShellPrefix`).
  The match is a label prefix, and the label is the full command text (`claudeengine/background.go` `payloadTask`, `launchKindAndLabel`), so `cd x && lyx gate test` or a leading `timeout` does not match.
- `PATTERN-completion-signal`: finalizing a negative answer over present output files is forbidden, so a held shell that outlives the deadline ends `done` through `classifyDeadlineExpiry`, never timeout.
  `completionsignal_enforcement_test.go` pins audited call sites in `wait.go`; touching `expiredTurnEnd`/`countsAtOnce` may need that test's pin updated in the same change.
- `PATTERN-shuttle-provider-seam`: label/signal knowledge stays in `claudeengine`; any new matching rule lives in `shuttleengine` as caller data or in `websterengine`.

### Part 2

- The cause is textual.
  `gateRepromptText` sends `Gate findings recorded at <path> — read it, fix every finding, and end your turn.`, one generic line for every gated producer.
  Master's template says "Do NOT localize or fix the failure yourself" only in its late "A gate failure: spawn one fixer fork" section and in its rules, and the generic re-prompt contradicts it at the moment Master acts.
- `renderVerifyGateFindings` (webster-owned) begins `Verify gate failed (attempt N of M).` and has no way-forward line.
- A verb that spawns the fixer is not available cheaply: forks are in-session Agent-tool calls, which Go cannot make; a Go-spawned strand would be cold and a different design (it is `recover-batch`'s model).

### Part 3 and the exit code

- `recoverSpawn` records a recovery strand's `StrandGUID`, `ShuttleRunDir`, `EventsPath`, but not its Claude session id; `BatchState.SessionID` is documented "Empty for a recovery batch".
- `loadRunWrites` audits only `recordedSessions`: `State.MasterSessionID` plus each batch's `SessionID`.
  A recovery's own session is never audited, so its writes are "unattributable" and `uncommittedPathsBlock` classes them `Not written by this run`.
  The same evidence feeds `reset.go` (own tracked paths) and `contractevidence.go` (last writer), so the gap is wider than the prompt.
- Session id is obtainable: `shuttleengine.FindRun(cfg, anchorRoot, guid)` returns `RunState.SessionID`, already used in `recoverSpawn` and `runlevel.go`.
- `webstercli/recoverbatch.go`: a terminal `stuck`/`dead` digest returns `output.Ok` (exit 0, envelope `ok:true`); only `ErrBatchFailed` from `PersistRecoveryTerminal` returns `ErrFields` with `batch_failed`.
  Master's failure ladder reads `status: stuck|dead` off a completed call, so the envelope fields must survive a non-zero exit.
- `recover-batch`'s `Long` help is near `maxLongBytes = 2000` (`cmd/lyx/clitree_test.go`); any help-text addition must be paid for in the same edit.

### Part 4

- `webster-body-implementer.md` is shared by fork and recovery.
  Its sandbox rule says "never run a mutating `lyx fabric` verb"; it never names `lyx gate test`, `lyx loom lint-comments` or `lyx quarry resolve` as allowed beyond card/batch gates, and says "a `tmux`-tier run goes through `lyx gate test`" without saying what to do for tmux-tagged card steps.
- `PATTERN-no-denied-recovery` and the raw-`go` deny: new text may not offer denied commands or spell the slot/strand/prebuilt-binary variables.
- `claims_test.go` pins stencil sentences (e.g. l.391 the "Not written by this run" line); stencil edits change it in the same commit.

### Part 5

- Quarry v0.2.1 (`internal/engine/resolve.go`) resolves exported and unexported names alike and puts same-package `_test.go` members in the package's own unit; only external `_test` packages are a separate unit (`foo_test`).
  Verified with `lyx quarry resolve`: `cmd/testtiming#rusageStats` found; `internal/testkit/tmuxkit#tmuxAllowed` found at HEAD.
- So the "unexported or test-file identifiers do not resolve" hypothesis in #511 is refuted by the code.
  The likely cause is part 6's ambiguity (the implementation declared the member in per-tag/per-GOOS files), but the issue's notes could not confirm it; the code cannot say.
- `doneCheckVerdicts` `create-not-done` detail is only `Create target %q still does not resolve`: no resolve key, no status, no unit.
  The `plan:` handle is stripped to its glyph by `resolveKeyFor`, so the displayed ref and the looked-up key differ.

### Part 6

- Quarry deliberately does not evaluate build constraints (`statusForMatches` comment); candidates carry `File`.
  `create_test`/`planglyph` see only the status and candidates, so classification must be in lyx (`go/build.Context.MatchFile`, `go/build/constraint`, filename GOOS/GOARCH suffixes) or via a quarry change (external module, version bump).
- Accept sites: `DoneChecks` create (found|multipart only), delete (anything but not_found blocks), rename; `statusFindings` `glyph-ambiguous` blocks Edit/Delete/Uses at Plan time for partitioned members too (#507's first half); `createFindings` treats an ambiguous Create target as `create-already-exists` at Plan time, which is what makes #507's "delete every declaration and re-create the name" plan fail.
- A genuinely ambiguous glyph (two declarations selectable by one build environment) must stay blocked; only candidates whose files can never be selected together may collapse.

## PATTERN entries touched

- `PATTERN-completion-signal` (part 1), `PATTERN-shuttle-provider-seam` (part 1 matching), `PATTERN-refusal-way-forward` (parts 3, 5: any refusal text/row change lands with its `refusal-spec.md` row and a test), `PATTERN-no-denied-recovery` (parts 2, 4 stencil text), `PATTERN-stencil-ownership` and `PATTERN-producer-pointer-rule` (stencils read at call time; do not paraphrase another producer's format), `PATTERN-sole-parsers` (plan parsing stays in `planparser`), `PATTERN-leaf-packages` (no listed leaf involved), `PATTERN-test-economy`/`PATTERN-test-speed` and the per-package rows in `cmd/lyx/testdata/test-budget.yaml` (`internal/planglyph`, `internal/websterengine`, `internal/webstercli`, `internal/shuttleengine`, `contracts/stencils` all have rows; cards that add top-level tests list the file under `Edit:`), `PATTERN-markdown-link-integrity`/`PATTERN-documentation-lifecycle` (module docs move into `doc.go`; board entry bodies are the unbuilt design).
- CLAUDE.md "Docs land in the same commit": `doc.go` of `shuttleengine`, `websterengine`, `webstercli`, `planglyph`; `contracts/specs/refusal-spec.md` webster rows; `contracts/specs/loom-plan-spec.md` for the Plan-time rule.

## Board entries touching this task

- `twb-combo-1`, `twb-twp-1`, `twb-twp-2`, `twb-twc-1`, `twb-twc-2`: sibling bench entries with the same title; same scope, not separate work.
- `fork-wait-noise` (#356): Webster forks end turns during gate waits, waking Merriam; touches the same stencils (`webster-template-master.md`, fork directive) and the same gate-wait text as part 1/4; both edit the Master template, so the record should keep part 1's stencil wording from colliding with it.
- `discussion-premise-cites` (#377): resolves glyphs in Discussion premises through the gate; shares the `planglyph` resolver and would inherit the ambiguity classifier of part 6.
- `glyph-practice-audit`, `kick-start-pack`, `enclose-kickstart`: consumers of quarry resolve answers; a classifier or quarry bump changes what they see for ambiguous members.
- `convergence-plan`: logs hub interventions per run; the "result discarded" record of part 1 is the kind of fact it wants.

## Conflicts between the task and PATTERN

- None outright.
- Tension: part 1's "stops the shell explicitly" alternative means lyx killing a pane process; lyx kills no shell today (`logEndedShells`), and `PATTERN-spawn-observability` would require logging it. Holding is the lower-risk route.
- Tension: part 3's non-zero exit for `stuck`/`dead` changes the ladder's reading of `recover-batch` output; the Master template and `refusal-spec.md` rows ("terminal stuck") must change together.
