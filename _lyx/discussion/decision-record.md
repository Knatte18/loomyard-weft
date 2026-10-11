# Decision record — dbench-ohsh: Webster stability

## Goal

Close the Webster faults outside the audit that stop or mislead runs:
- Master's background test shells cut off at a turn end.
- Master fixing a verify failure itself.
- A second recovery treating the first recovery's files as foreign.
- `recover-batch` exiting 0 on a failed recovery.
- A recovery prompt that does not name its allowed gates.
- Create, Edit and Delete checks that can never pass for a member declared once per build-constraint set or per GOOS.

## Scope

In:

- Item 1, gate shells (#505, #509, #517):
  - The Master stencil forbids Master from running tests.
  - The ended-shell record is reworded.
- Item 2, Master self-fix:
  - Each gate entry gets its own re-prompt text.
  - Webster's verify gate supplies text that routes the failure to the one fixer fork.
  - The verify-gate findings file opens with the same rule.
- Item 3, recovery attribution (#522): recovery strand sessions count as the batch's own write evidence.
- Item 3, exit code: `recover-batch` exits non-zero on a terminal `stuck` or `dead` digest.
- Item 4, recovery gates: the shared implementer body names the always-allowed gates and the tmux/llm compile-only rule.
- Item 5, Create refusal text (#511): every resolve-backed finding names the lookup quarry made and its answer.
- Item 6, build variants (#500, #507): an ambiguous answer whose candidates are mutually exclusive build variants of one declaration counts as one declaration, everywhere `planglyph` reads quarry, `lyx quarry resolve` included.

Out:

- A change to quarry itself (external module `github.com/Knatte18/quarry`).
- Letting `create-already-exists` pass when the same card deletes every declaration of the name (#507, second half).
  This stays a documented residual (D9).
- Reviving a dead batch through `lyx loom goto` (#511, point 3).
  `lyx webster reset --to batch-start --batch NN` already clears the batch's recovery count.
- Any hard deny of `lyx gate test` for Master's session.
- Stopping a Claude background shell from lyx, beyond ending the strand.
- Code moves in `cmd/testtiming` or `tmuxkit`, already done in the tree.

## Decisions

### D1 — Master never runs tests

Decision:
- `webster-template-master.md`'s "What you never do" section states that Master never runs tests, `lyx gate test` or any `go test`, in the foreground or the background.
  The verify gate that runs after Master's turn is Master's only test evidence.
- A `contracts/stencils/claims_test.go` row pins the sentence.

Rationale:
- Master started the cut-off `lyx gate test --tags integration` shells on its own, before any gate message.
- Shuttle's `countsAtOnce` correctly counts a fresh gated run's first turn end at once when every outstanding task is an unawaited shell.
- A webster `done` already requires the plan-level verify gate to pass at HEAD, and that gate includes the integration tier.
  So Master's shell can never change the outcome.

### D2 — The ended-shell record says the result was discarded

Decision: wherever an ended shell is recorded, the text says that the shell ended with Master's strand, that its result was never read, and that the run's test evidence is the verify gate's result.
That covers the `webster-background-shell` friction note, the `RunResult.Warnings` entry and summary.md's "Background shells at the run's end" section.
- The text never says lyx stopped the shell.
- The prose that states the current wording is updated with it: `internal/websterengine/doc.go` "Background shells in Master's wait", and the `shuttleengine` doc and config comment near `background_shell_wait_min`.

Rationale:
- The issues asked that the outcome never read as "tests passed".
- lyx holds no pid for a Bash-tool child (`logEndedShells` documents that lyx kills no shell), so ending the strand is the only stop there is, and the record must say exactly that.

### D3 — Each gate entry has its own re-prompt text; webster's routes to the fixer fork

Decision:
- `shuttleengine.GateEntry` gains an optional re-prompt text, which takes the findings path.
  When it is empty, the current generic text (`gateRepromptText`) is sent, so every other gated caller is unchanged.
- Webster's verify-gate entry supplies text that says three things, on one line: do not edit or fix anything yourself; spawn exactly one verify-fix fork as the stencil's "A gate failure" section says; end your turn.
  The text covers the dirty-tree failure too, because `webster-body-verify-fix.md` already handles it ("commit or remove exactly those paths").
- `renderVerifyGateFindings` opens the findings file with the same rule.
  Its dirty-tree branch no longer addresses Master with "Commit or remove these paths"; it names the paths for the fork.

Bound:
- The new field carries text only.
- It must pass `validateSendText`'s one-line rule.
- It changes no gate verdict, attempt count or cap.
- It names no command the agents' settings deny (`PATTERN-no-denied-recovery`) and never tells the agent to ask the operator (`PATTERN-parent-directive`).
- The text is provider-neutral (`PATTERN-shuttle-provider-seam`).

Rationale: Master's stencil already says "Do NOT localize or fix the failure yourself", but the message and the findings file Master acts on say the opposite, so the stencil's rule is the hard path to follow.

### D4 — Recovery strand sessions are the batch's own write evidence

Decision:
- `BatchState` gains an `omitempty` list of the Claude session ids of the batch's recovery strands.
- `recoverSpawn` records the id it already reads through `FindRun` (`runState.SessionID`).
  It carries the list across respawns the way `Recoveries` and `SuspectPaths` are carried.
- `recordedSessions`, or the loop in `loadRunWrites`, includes these sessions.
  Their `ParentWriteEvents` go into `RunWrites.Forks`, never `RunWrites.Master`.
- A recovery session whose transcript cannot be audited, for example a missing transcript on another machine, is skipped and logged rather than failing `loadRunWrites`.
  Its paths then fall under "Not written by this run", as today.

Bound:
- Only sessions lyx itself spawned as a recovery of this run's batches are added.
- They widen exactly the three `loadRunWrites` consumers: `uncommittedPathsBlock`, `reset`'s own-path set and `contractevidence.go`.
- They never feed the run-exit parent-write audit or `countBegunForkBatches`.
- An unattributable path still counts as foreign.

Rationale:
- In #522 the first recovery's 19 uncommitted files were listed as "Not written by this run", because `recordedSessions` holds only `MasterSessionID` and the fork batches' `SessionID`, which is empty for a recovery.
- Routing them to Forks keeps `contractevidence.go`'s "a fork wrote after Master's last write" reading correct.

### D5 — `recover-batch` exits non-zero on a terminal `stuck` or `dead` digest

Decision:
- A terminal recovery digest whose status is not `done` returns an `ok: false` envelope with a non-zero exit.
- The envelope carries every field the `ok` envelope carries today: `batch`, `status`, `dead_reason`, `recovery_retry`, `recoveries` and `warnings`.
- Its error names the next rung: re-run `lyx webster recover-batch NN` once more when `recovery_retry` is true, otherwise end the run stuck naming the batch.
- `refusal-spec.md` gains the row.
  The row states the difference from `record-batch`: a stuck digest from `record-batch` is a ladder signal toward recovery and stays `ok`, while a non-done recovery ends the batch's automatic route.
- The existing `noteRefusals` tee writes the friction note.
- The Master stencil's recover-batch rungs stay keyed on the same fields, reworded so they cover the non-zero exit.

Rationale:
- An exit-0 call whose batch is now stuck or dead reads as success to any caller that checks the exit.
- Only `batch_failed` and the other refusals exit non-zero today.

### D6 — The implementer body names its allowed gates and the tmux/llm rule

Decision: `webster-body-implementer.md`, shared by forks and the recovery strand, gains two statements.
- Next to the sandbox rule: the card gates and batch gate, `lyx gate test` and `lyx loom lint-comments`, are always allowed.
  The ban covers only mutating `lyx fabric` verbs.
- Next to step 5 (a card's `verify:`) and the "Slot-busy exits" section: a tmux- or llm-tagged step is compile-checked with `go vet -tags <tag> <package>` (a named package, never `./...`) and never run here, because Publish's `publish_verify` runs it.
  No report is `FAILED` over such a step.

`claims_test.go` rows pin both statements.

Rationale:
- The ambiguity lives in the shared body, whose only fabric ban is the sandbox rule.
- The plan spec makes the tmux tier compile-only in plans and run at Publish.
- `go vet -tags tmux <pkg>` is not matched by the raw-go deny, so the sentence names no denied command.

### D7 — Resolve-backed findings name the lookup and quarry's answer

Decision:
- The Detail text is built where the finding is built, in `doneCheckVerdicts` and the plan-time `statusFindings`, so the `batch_failed` envelope and the plan gate carry the same text.
  It covers `create-not-done`, `rename-not-done`, `glyph-not-found` and `glyph-ambiguous`.
- The text names:
  - the glyph resolved through quarry against the post-batch or plan-time worktree;
  - quarry's status;
  - for `not_found`, whether the unit (file or package) was found;
  - for `ambiguous`, each candidate file and why the candidates are not one build-variant set under D8.
- The `refusal-spec.md` "card not done" row is updated.
  For an ambiguous Create, the way forward is a plan edit, in the same shape as the existing plan-edit refusals.

Rationale:
- In #511 the operator could not tell a missing symbol from a resolver limit.
- Quarry resolves unexported members and `_test.go` members (`cmd/testtiming#rusageStats` and `internal/planglyph#TestDoneCheckVerdicts_Rules` answer `found`), so resolution itself needs no change.
- The #511 targets were build-variant ambiguities that the text reported as "still does not resolve".

### D8 — A build-variant set resolves as one declaration

Decision:
- One predicate in `internal/planglyph` decides whether an `ambiguous` answer is one build-variant set, all three conditions holding:
  - every candidate sits in one package directory;
  - every candidate has the same kind;
  - no build context matches two candidates' files.
- Matching uses the standard library (`go/build` `Context.MatchFile`, which reads both `//go:build` lines and `_GOOS`/`_GOARCH` filename suffixes), never a hand-written constraint parser.
- It evaluates every combination of a fixed set of platform values (the platforms this repo targets, plus one value outside them) and every assignment of the custom tags the candidates' constraints name.
- It is applied once, at the resolve chokepoint:
  - `resolveTargets`, the package's only `(*quarry.Repo).Resolve` call, rewrites a variant-set `ambiguous` answer to `multipart`, with `Symbols` set to the candidates.
  - The exported `Resolve`, which `lyx quarry resolve` uses, goes through the same rewrite.
    So the plan author's check, the plan gates, `begin-batch`, record-batch's DoneChecks and the implementer see one answer.
- `resolveTargets` is told the worktree root alongside the repo.
- `handle.go` keeps rejecting `multipart` for a rename's signature source, since a signature needs exactly one declaration.
- The `lyx quarry resolve` help and `planglyph/doc.go` say that a build-variant set answers `multipart`.

Bound: the guard that blocked every `ambiguous` answer now passes only a variant set.
- A candidate set fails the predicate, and the answer stays `ambiguous`, in any of these cases:
  - the candidates span directories or kinds;
  - a candidate file cannot be read;
  - the constraints name more custom tags than a fixed cap;
  - two candidate files match one evaluated context.
- A false "disjoint" can only hide a genuine duplicate, which the compiler rejects in the verify gate.
- A false "not disjoint" keeps today's behaviour.
- Delete's done-check is unchanged: any remaining candidate still counts as present.

Rationale:
- `tmuxAllowed` (#500), `StartTime` and `runChild` (#507), and #511's `rusageStats` are idiomatic per-tag or per-GOOS declarations.
  Quarry deliberately ignores build constraints, so their Create could never pass and their Edit or Delete could never be planned by glyph.
- Rewriting at the chokepoint reuses every consumer's existing `multipart` handling, so the status-enforcement and status-completeness tests see no new branch.
- The plan-time half of the board's expectation is met because the plan gate and record-batch share the predicate:
  - an ambiguous answer that is not a variant set blocks at Plan time as `glyph-ambiguous`, with D7's reason;
  - a variant set passes in both places.

### D9 — The delete-and-recreate residual is documented

Decision: `planglyph/doc.go` records the residual:
- A plan cannot delete every declaration of a name and create the same name in one card; `create-already-exists` still fires.
- The plan edits or re-signs the variant set (now resolvable under D8), or renames.

Rationale: under D8 an Edit, including a re-sign, replaces a variant set in place, so the delete-then-recreate path #507 hit is no longer needed.

## Constraints

- `PATTERN-refusal-way-forward`: D5's new non-zero exit and D7's changed Detail land with their `refusal-spec.md` rows in the same commit, each name its way forward, and a test reaches each.
- `PATTERN-friction-capture`: D5's refusal writes its friction note through the existing `noteRefusals` tee; no new note path.
- `PATTERN-no-denied-recovery`: no stencil, re-prompt or refusal text names a denied command. D6 names `go vet -tags <tag> <package>` and `lyx gate test`, never a raw `go test -tags tmux` or `./...`.
- `PATTERN-parent-directive`: no new text tells an agent to ask the operator.
- `PATTERN-stencil-ownership`: stencils stay read from the told directory at call time. New stencil claims are pinned in `contracts/stencils/claims_test.go`.
- `PATTERN-shuttle-provider-seam`: D3's field and text live in `shuttleengine` and are provider-neutral.
- `PATTERN-told-geometry`: D8's root is told to `resolveTargets` by its callers (`openRepo` already holds it); `planglyph` derives no path.
- `PATTERN-sole-parsers`: nothing parses plan or discussion files outside their parsers.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-test-isolation`:
  - New cases are rows in existing table tests where a test already covers the surface.
  - Every new test is unit tier, or hermetic-git tier for `recoverbatch_git_test`.
  - Nothing needs the integration, tmux or llm tier.
- Test budget:
  - `planglyph`, `websterengine`, `shuttleengine` and `webstercli` sit at or near their rows.
  - A card adding a top-level test lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`, per CLAUDE.md.
- Docs in the same commit, per CLAUDE.md: `internal/websterengine/doc.go`, `internal/shuttleengine/doc.go`, `internal/planglyph/doc.go` and the `lyx quarry resolve` help.
- `state.json` stays readable by older runs: D4's field is `omitempty`.
- Go comments use semantic line breaks (`PATTERN-comment-line-breaks`).

## Auto-mode assumptions

- The board's "`recover-batch 1` exited 0 while the batch ended failed" is read as a terminal `stuck` or `dead` digest.
  The `failed` status is set only by `failBatch`, whose path already exits non-zero.
- #511's `rusageStats` failure was a per-GOOS ambiguity: `cmd/testtiming` has `rusage_linux.go` and `rusage_other.go`, and the symbol resolves `found` today.
- Item 1 needs no wait change. `countsAtOnce` is right to end a fresh gated run on an unawaited shell, because the verify gate supplies the test evidence.
- The fixed platform set for D8 is the platforms the repo builds for (linux, windows, darwin, with their common architectures) plus one value outside them. The plan writer fixes the exact list and the tag cap.
- The rewrite to `multipart` is an acceptable answer shape for `lyx quarry resolve`, documented in its help, rather than a new status.

## Open risks

- `gate-test-hygiene` (running in Webster) changes `cmd/lyx/testdata/test-budget.yaml`'s collision behaviour. A row raised here may collide at Publish's merge of main.
- D1 is a stencil rule with no hard deny. A Master that still starts a test shell is caught only by D2's record.
- D8 changes `lyx quarry resolve`'s answer for variant sets from `ambiguous` to `multipart`. A plan author or tool that read `multipart` as "split declaration of one body" now also sees variant sets.
- D5 changes the Master ladder's exit-code contract. A Master on an older stencil reading a non-zero exit as a refusal would still find the same fields in the envelope.

## Acceptance criteria

- A Master stencil claim forbids Master from running tests or `lyx gate test`, and names the verify gate as its test evidence.
- An ended shell's warning, friction note and summary section say that it ended with Master's strand, that its result was never read, and that the verify gate's result is the run's test evidence; none says lyx stopped it.
- A failed webster verify gate sends Master the webster re-prompt (one line, fixer-fork route, no self-fix), for a test failure and for a dirty tree. A gated run with no per-entry text still gets the generic text.
- The verify-gate findings file opens with the fixer-fork rule, and its dirty-tree branch no longer tells Master to commit or remove paths.
- A second recovery of a batch lists the first recovery's uncommitted files under "Written by this run".
- A recovery session with no auditable transcript leaves its paths under "Not written by this run" without failing `recover-batch`.
- Recovery writes never appear as Master writes to `contractevidence.go` and never count in `countBegunForkBatches` or the run-exit parent-write audit.
- A `state.json` without the new field loads unchanged.
- `recover-batch` with a terminal `stuck` or `dead` digest exits non-zero with the digest fields and a way forward that matches `recovery_retry`.
  A `done` digest still exits 0.
  The refusal-spec row exists and a test reaches it.
- Implementer-body claims name the always-allowed gates beside the sandbox rule, and the tmux/llm compile-only, never-`FAILED` rule beside step 5.
- `create-not-done`, `rename-not-done`, `glyph-not-found` and `glyph-ambiguous` details name the glyph, quarry's status, and either the unit result or the candidate files with the non-variant reason.
- A member declared once per mutually exclusive `//go:build` set, and one declared per GOOS by filename suffix:
  - resolves `multipart` through `planglyph` and through `lyx quarry resolve`;
  - passes a Create done-check;
  - is accepted as an Edit or Delete target at the plan gate.
- Two declarations of one name whose constraints overlap, or that sit in different directories or kinds, stay `ambiguous` and block at plan time and at record-batch.
- A Delete of a variant set fails its done-check while any variant remains.
- A rename whose old side is a variant set still refuses at `handle.go`.

## Notes for the plan writer

- Item 1's mechanism is `countsAtOnce` in `internal/shuttleengine/wait.go`. The only awaited Master prefix is `masterAwaitedShellPrefix`, set in `websterengine/runlevel.go`.
- The re-prompt is sent at the `gateRepromptText` call in `wait.go`, where `run.gate[failed]` is in scope. `renderVerifyGateFindings` is in `websterengine/verifygatereport.go`.
- `recoverSpawn` already calls `FindRun` and discards `runState.SessionID`. `contractevidence.go`, `recoverbatch.go` and `reset.go` are `loadRunWrites`' three callers.
- `resolveTargets` and the exported `Resolve` are in `internal/planglyph/repo.go`. `status_enforcement_test.go` and `status_completeness_test.go` pin status consumption.
- Variant fixture: `internal/testkit/tmuxkit` (`allow_tmux.go`/`allow_none.go`) and `internal/proc` (`proc_linux.go`/`proc_windows.go`) are real in-tree shapes, but a temp-dir fixture keeps the test independent of them.
