# Support log — twb-tw-om-1

## Interview

Autonomous run: no operator answered; every pick is the writer's own and is logged under `## Question ledger`.

1. Read the board entry and the board list.
   The pair's tree already held `1e79825ff` ("Webster stability: gate shells, recovery writes, partitioned members"), which closes every issue on this entry.
   `docs/research/discussion-bench.md` says bench pairs reset their code branch to `800a0e892`; this pair was not reset.
   Before asking, the writer had read that commit's message and file list, not its diff.
2. Asked `ly:orch` which tree to face.
   `ly:orch` answered: write against base `800a0e892`, read through `git show`/`git grep`, treat `1e79825ff` and later as not landed, and do not repair the pair.
   The writer acknowledged and disclosed the earlier read of the commit message.
3. Item 1 at base: `countsAtOnce` lets a fresh gated run with only unawaited shells count at once; `background_shell_wait_min` already expires nothing.
   Picked: drop the exception for every gated run (`D1`).
4. Item 2 at base: the shared re-prompt says "fix every finding", against Master's stencil; picked a per-entry renderer plus a report header (`D2`).
5. Item 3 at base: recovery strands' sessions are recorded nowhere, so `recordedSessions` never audits them; `recover-batch` returns a failed digest on the ok envelope.
   Picked: record recovery sessions per batch with a third write group (`D3`) and non-zero exit for every non-done terminal (`D4`).
6. Item 4 at base: the implementer body bans mutating `lyx fabric` verbs but never names the allowed lyx verbs; `steerRawGoDeny` refuses tagged `go test`, not `go vet`.
   Picked `D5`.
7. Items 5 and 6: probed `lyx quarry resolve` (quarry's `go.mod` pin is the same at base and now): unexported and `_test.go` members answer `found`, per-GOOS `proc#StartTime` answers `ambiguous`.
   #511 is therefore the ambiguity of #500/#507, not a resolver gap.
   Picked richer findings (`D6`) and a `MatchFile` classifier in `planglyph` (`D7`).
8. Lens review: six forks (premises, goal-scope, board-fit, pattern-fit, scenarios, leanness); every fork returned a report.
   Duplicates merged: the pinned-port risk (pattern-fit 3, goal-scope 4, scenarios 7) and the unselectable-candidate rule (pattern-fit 3, scenarios 7).
   - premises, `gc` held true through `Context.Compiler`: fixed, `D7` sets `Compiler` to neither and passes `gc`/`gccgo` as free tags.
   - premises, ended-shell "never read" wording is not existing text: fixed, `D1` and Scope make it a change.
   - premises, hedge on removing `startedFresh`/`gatedArrival`: fixed, removed outright.
   - premises, session id at spawn and fixer reads the report: confirmations, no change.
   - goal-scope, #511's dead-batch-after-goto item unhandled: fixed, new `D8` and a Scope Out line.
   - goal-scope, `D5` runs tmux-tagged gate steps against the board: fixed, every `tmux`/`llm` run compiles only, bounded to those tags.
   - goal-scope, `D6` cause stated as verified: fixed, worded "most likely" with the detail as the safeguard.
   - goal-scope, pinned-list risk names no owner: fixed through the unselectable-candidate rule; the risk line is removed.
   - goal-scope, Plan-time flag boundary only inside `D7`: fixed, Scope line added.
   - board-fit, `burler-multi-llm` edits the neighbouring hold path: fixed, Notes line.
   - board-fit, `pane-inject-safety` send guard: fixed, Notes line.
   - board-fit, `weaver` inherits `D3`/`D5`: rejected, informational, no record change.
   - board-fit, name `audit-reader-globs`: fixed in Scope Out.
   - pattern-fit, `PATTERN-stencil-ownership` misread: fixed, `D2` states the strings are Go-authored runtime messages like the default re-prompt.
   - pattern-fit, the report is YAML: fixed, `D2` targets `renderVerifyGateFindings`, the YAML stays unchanged.
   - pattern-fit, unknown port errs toward partitioned: fixed, a candidate no environment selects stays `ambiguous`.
   - pattern-fit, `...` patterns denied: fixed, `D5` says packages are listed one by one.
   - pattern-fit, producer-pointer departure unnamed: fixed, `D2` names it and points at the stencil section.
   - pattern-fit, gate parity: fixed, `D7` names `PATTERN-gate-self-check-parity`.
   - scenarios, missing recovery transcript: fixed, skip with warning, contract files uncleared, criterion added.
   - scenarios, hold size and unseen completion: fixed in `D1` and Open risks, fork-leftover criterion added.
   - scenarios, `D4` persist order, `batch_failed`, running snapshot: fixed in `D4` and criteria.
   - scenarios, attach/duplicate/failed-start for session ids: fixed in `D3`.
   - scenarios, empty re-prompt render: fixed, falls back to the default.
   - scenarios, fork-outstanding and attached-run regressions: fixed, criterion added.
   - leanness, rationale clauses arguing rejected alternatives (`D1`, `D3`): fixed, cut.
   - leanness, probe history in `D6`: fixed, conclusion kept.
   - leanness, base fact stated twice: fixed, kept in Auto-mode assumptions.
   - leanness, Auto-mode bullets repeating rationale: fixed, one line of self-picks.
   - leanness, docs and test-tier Constraints, test-budget note: fixed, cut.
   - leanness, Scope Out reasons: fixed, trimmed.

## Rejected alternatives

- Item 1, Master-only hold on a `lyx gate` label prefix: a `cd x &&` command defeats the prefix, and other gated producers keep the fault.
- Item 1, stopping the shell and recording its result discarded: lyx has no shell pid to stop, and an earlier end still loses the tier's result.
- Item 1, a Master stencil rule never to run tests: binds Master only and leaves shuttle's at-once count in place.
- Item 2, a verb that spawns the fixer: a CLI process cannot spawn an in-session fork.
- Item 3, a clean tree for the second recovery with the first diff as reference: discards the first attempt's work and still needs attribution for a reset.
- Item 3, auditing recovery sessions as Master sessions: a recovery's later contract-file write would clear a finding as Master's.
- Item 4, `dead` with `recovery_retry: true` exiting 0: defensible, but two rules where one suffices.
- Item 6, quarry honouring build constraints: changes `lyx quarry resolve`'s raw answers and an external module.
- Item 6, rewriting a partitioned answer to `multipart` inside `Resolve`: changes the CLI's answer, which promises no re-shaping.
- Item 6, independent booleans per tag: cannot read `_linux.go` beside `_windows.go` as disjoint; `MatchFile` over enumerated contexts can.
- #511 dead batch, reviving it on `lyx loom goto` into Webster: re-opens a batch whose recoveries are spent without an explicit operator act; the existing `batch-start` reset already clears the counters.
- Item 4, running a tmux-tagged gate step in the strand: the plan spec leaves that tier to Publish, and the recovery could not run it.
- Item 6, a `create-already-exists` waiver for Delete-then-Create of one name: the partitioned-member acceptance removes the case.

## Review rounds

_No rounds yet._

## Question ledger

- Resolved (ly:orch): which tree to face; base `800a0e892`.
- Environment fact, not repaired: the bench pair's code branch was not reset to its base.
- Auto-pick: `D1` scope is every gated producer; the fault is not Master-specific.
- Auto-pick: `D1` deadline outcome is `OutcomeDone` through `classifyDeadlineExpiry`, as `PATTERN-completion-signal` requires.
- Auto-pick: `D3` treats recovery writes like fork writes for contract evidence.
- Auto-pick: `D4` one non-zero rule for `failed`, `stuck` and `dead`.
- Auto-pick: `D5` compile-only command is `go vet -tags <tag> <batch packages>`, which the raw-go deny admits.
- Auto-pick: `D7` free-tag cap of four; genuine ambiguity stays blocked at Plan time.
- Auto-pick: `D7` Rename old side stays found-only, with a Delete-plus-Create way forward.
- Auto-pick: `D3` a missing recovery transcript is skipped with a warning, failing closed only for contract evidence.
- Auto-pick: `D8` the reset route over a goto revival; whether `reset --to batch-start` already clears a dead status is left for Plan to read, and `D8` requires it to.
