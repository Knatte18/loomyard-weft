# Support log: dbench-solo

## Interview

Autonomous run: no operator answered, so each turn below is a question the board entry raised, the evidence read, and the auto-pick.

1. Item 1, gate shells.
   Read `internal/shuttleengine/wait.go`: shells no longer expire (fixed in 5a8be0274), but `countsAtOnce` still counts a fresh gated run's turn end at once when every outstanding task is an unawaited shell, which is how a `done` run ends over a gate shell.
   Master's spec awaits only `lyx webster recover-batch`.
   Pick: add `lyx gate ` as an awaited prefix (D1), mark ended gate shells as discarded (D2), and tell Master not to run gates (D3).
2. Item 2, Master self-fix.
   Read the Master stencil's gate-failure section and `gateRepromptText`: the message Master receives says "fix every finding", contradicting the stencil.
   Pick: a per-entry re-prompt text naming the fixer fork (D4).
3. Item 3, recovery attribution.
   Read `uncommittedPathsBlock`, `loadRunWrites`, `recordedSessions`: write evidence covers Master and fork sessions only; recovery strands, separate sessions, are absent.
   Pick: record every recovery session per batch and include it (D5); exit code non-zero on terminal-failed digest (D6).
4. Item 4, recovery gates.
   Read `webster-prefix-recovery.md` and `webster-body-implementer.md`; the plan stencil already says the tmux tier is compiled by `go vet -tags tmux` and run at Publish.
   Pick: state allowed gate commands and the tmux rule in the shared body (D7).
5. Item 5, unexported targets.
   Probed `lyx quarry resolve`: `cmd/testtiming#rusageStats` and `internal/testkit/tmuxkit#tmuxAllowed` resolve `found` now; an in-package test-file function resolves `found`; an external test package's function resolves as `<dir>_test#Name`.
   Unexported and test-file identifiers are not the fault; the batches died on build-constraint ambiguity.
   Pick: make the detail name status, candidates and the lookup's coverage (D8).
6. Item 6, build-tagged members.
   `internal/proc#StartTime` resolves `ambiguous` with `proc_linux.go` and `proc_windows.go`.
   quarry is the external module `github.com/Knatte18/quarry`, outside this worktree.
   Pick: a disjoint-constraint predicate in `internal/planglyph`, used at Plan time and record-batch (D9).

## Rejected alternatives

- Item 1: hold every background shell, awaited or not, at a gated turn end. Rejected: unawaited shells finishing at once is deliberate for left-over shells that would otherwise stall Master's arrival until its deadline.
- Item 1: lyx kills outstanding gate shells at run end and records it. Rejected as the main fix: the result is still lost; D1 keeps it, D2 covers the remaining exits.
- Item 2: a `lyx webster` verb that spawns the verify fixer. Rejected: a fork is an in-session Agent call, which a CLI verb cannot start; a cold strand would change Webster's fixer model.
- Item 2: only restate the rule in the stencil. Rejected: the stencil already states it, and Master followed the contradicting message.
- Item 3: start the second recovery from a clean tree with the first attempt's diff as reference. Rejected: it needs a destructive reset of uncommitted work and throws away a valid start the prompt already tells the strand to check.
- Item 6: change quarry to honour build constraints. Rejected for this task: separate module and release; lyx's policy over quarry's answers is in `internal/planglyph` already.
- Item 6: accept any ambiguous answer whose candidates are all `.go` files. Rejected: an overlapping redeclaration is a real defect the check must still catch.
- #507's option of letting a Create reuse a name the same card deletes. Rejected: D9 lets the plan Edit the per-GOOS symbol in place, so no rename is forced.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): should a gate shell hold Master's turn, or be stopped and recorded? Hold it via awaited prefix, bounded by `master_timeout_min` (D1), and record any that a run ends over (D2).
- Q2 (resolved, auto-pick): stencil text or a verb for the verify fixer? Neither alone; the gate re-prompt carries the rule (D4).
- Q3 (resolved, auto-pick): attribution by batch or a clean tree for the second recovery? Attribution, by adding recovery sessions to the run's write evidence (D5).
- Q4 (resolved, auto-pick): which `recover-batch` digests exit non-zero? `stuck`, and `dead` without a retry (D6).
- Q5 (resolved, auto-pick): what does the recovery strand do with tmux-tagged steps? Compile-check, leave the run to Publish (D7), the plan stencil's existing rule.
- Q6 (resolved, by evidence): does the Create check resolve unexported and test-file identifiers? Yes, probed with `lyx quarry resolve`; D8 makes the detail say so.
- Q7 (resolved, auto-pick): quarry change or lyx-side predicate for build-constrained members? lyx-side in `internal/planglyph` (D9).
- Q8 (resolved, auto-pick): reviving a dead batch through `goto` (#511 point 3)? Out of scope; not on this board entry.
- Q9 (resolved, auto-pick): does D9's predicate also change Delete done checks? No: an ambiguous answer means a declaration is still present.
