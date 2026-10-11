# Support log — twb-so-om-1

## Interview

The session ran autonomously: no operator answered, and every decision below is an auto-pick recorded in the question ledger.
The codebase was read at base 800a0e892 through `git show` and `git grep`, as the board entry's bench rule requires.

1. Gate shells (#505, #509, #517).
   At base, `background_shell_wait_min` no longer expires a shell, but a gated, fresh Master turn end with its output files present counts Done at once when only unawaited shells are outstanding, and strand removal then kills them.
   The awaited-prefix mechanism exists with one prefix, `lyx webster recover-batch`.
   Picked: add `lyx gate test` and match per command segment, so `cd … &&` and piped labels match.
2. Master self-fix.
   Master's template already says "Do NOT localize or fix the failure yourself", but shuttle's generic gate re-prompt reads "read it, fix every finding, and end your turn".
   The contradiction at the moment of the failure is the cause.
   Picked: a per-entry re-prompt text for webster's verify entry that carries the fixer-fork rule.
3. Recovery attribution (#522).
   `loadRunWrites` audits Master sessions and their forks only; recovery strands are separate sessions and never enter the evidence.
   Picked: record recovery session ids on the batch record and audit them as the run's own writes.
4. `recover-batch` exit code.
   A terminal `stuck` or `dead` digest returns `output.Ok`, so exit 0.
   Picked: exit non-zero with the same fields.
5. Recovery gates.
   The shared implementer body names only bans (no mutating `lyx fabric`, no `lyx webster`).
   `lyx gate test --tags tmux` is not denied by any setting; only raw tagged `go test` is.
   Picked: name the allowed gates in the shared body and run tmux-tagged steps through `lyx gate test`.
6. Create checks (#511) and build-tagged members (#500, #507).
   Probing with `lyx quarry resolve`: unexported identifiers and `_test.go` declarations resolve `found`; per-GOOS `internal/proc#StartTime` resolves `ambiguous`.
   At base `cmd/testtiming` holds `rusage_linux.go` and `rusage_other.go`, so #511's `rusageStats` failed through the same ambiguity, which the `create-not-done` message hid.
   Picked: a build-variant classifier in `planglyph` and lookup detail in every resolve finding.
7. Related board work.
   `fork-wait-noise` covers forks ending their turn on gate waits and the notices that wakes Master; D1 does not change fork behaviour and fits either outcome of that task.

## Rejected alternatives

- D1, kill outstanding shells explicitly and record the discard: lyx has no pid for a provider shell, and killing a gate run discards the very result the run needs.
- D1, hold every outstanding shell: a stray `sleep` or watcher would hold Master to its deadline; the gate class is the one whose result matters.
- D1, a substring match on `lyx gate test` anywhere in the label: matches a `grep 'lyx gate test'` or an `echo`; per-segment start is tighter.
- D2, only restate the rule in Master's template: the template already states it, and the conflicting message still won.
- D2, a verb that spawns the fixer: forks are in-session Agent calls; no CLI verb can make one.
- D3, start the second recovery from a clean tree with the first attempt's diff as reference: discards committed-worthy work, needs a reset path and a diff artifact the run does not otherwise have.
- D3, attribute by batch only: the run-level evidence already exists and a recovery runs only after earlier batches are terminal, so batch scoping adds state without changing an answer.
- D5, leave tmux-tagged steps to verify: nothing guarantees the plan-level verify runs the tmux tier, and `lyx gate test --tags tmux` is allowed from every strand.
- D6, make quarry honour build constraints: quarry is a separate module outside this repo, and picking one build would hide the other variants from Edit and Delete.
- D6, accept every ambiguous answer at record-batch: would pass a genuine duplicate declaration.
- #507's first option, let a Create reuse a name the same card deletes: with build-variant Edit and Delete working, an in-place Edit covers that case.
- #511's goto-revives-dead-batch question: a separate halt and recovery policy, not a cause of the faults here.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): should gate shells hold or be killed? Hold via the awaited-prefix set, bounded by the run's Timeout; non-gate shells keep today's record with "result never read" wording.
- Q2 (resolved, auto-pick): how to stop Master's self-fix? Webster's verify gate entry supplies its own re-prompt text naming the fixer fork and prompt path.
- Q3 (resolved, auto-pick): attribution by batch or a clean restart? Recovery sessions join the run's write evidence.
- Q4 (resolved, auto-pick): which `recover-batch` outcomes exit non-zero? Terminal `stuck` and `dead`; `done` and a running snapshot exit 0.
- Q5 (resolved, auto-pick): tmux-tagged steps in recovery — run or leave to verify? Run through `lyx gate test --tags tmux`; departs from the board entry's "left to verify" wording, for the reason in Rejected alternatives.
- Q6 (resolved, auto-pick): does the Create check resolve unexported and test-file identifiers? Yes, probed with `lyx quarry resolve`; the #511 failures were per-GOOS ambiguity.
- Q7 (resolved, auto-pick): where does the build-constraint check live? `internal/planglyph`, failing closed on anything it cannot read.
- Q8 (resolved, auto-pick): what does Plan time flag? An ambiguous Edit, Uses or Delete glyph whose candidates are not build-disjoint, with the overlapping pair named.
- Q9 (open, for the plan): where webster reads a recovery strand's provider session id (assumption A2).
- Environment note: the quarry probes ran against the pair's current tree, not base; quarry's version is unchanged since base (`v0.2.1` in `go.mod`, no `go.mod` diff), and the probed files are unchanged where it matters.
