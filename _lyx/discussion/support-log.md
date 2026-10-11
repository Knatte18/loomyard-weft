# Support log — dbench-ohsm

## Interview

Autonomous run: no operator turns.
The chair explored the code, then put two question batches to advisor `ly:dbench-ohsm:discussion-advisor-1`.

### Exploration (chair)

- Read board entry `dbench-ohsm` and the linked issues #505, #509, #517, #522, #511, #500, #507; read adjacent entries `gate-test-hygiene`, `proc-child-lifetime`, `audit-reader-globs`, `weaver`, `kick-start-pack`, `enclose-kickstart`.
  The `dbench-*` entries are sibling benchmark variants of this task.
- Item 1: shuttle's `expiredTurnEnd` counts a gated turn end with every output file present as an arrival when every outstanding task is an unawaited shell; Master awaits only `lyx webster recover-batch`; `awaitedLabel` is a plain `HasPrefix`; a shell's `Label` is its command (description only as fallback).
- Item 2: shuttle's `gateRepromptText` sends "read it, fix every finding, and end your turn", which contradicts the Master stencil's fixer-fork rule.
- Item 3: `uncommittedPathsBlock` classes paths by `loadRunWrites`, which audits only Master sessions and their forks; recovery strands are never audited. `recover-batch` returns `output.Ok` for every terminal digest.
- Item 4: the recovery prefix's sandbox rule bans mutating fabric verbs and never names gates as allowed; the plan format leaves the `tmux` tier to Publish's `publish_verify`, compiling it with `go vet -tags tmux`.
- Items 5–6: `lyx quarry resolve` answers `found` for an unexported const and for a `_test.go`-only function; `internal/proc#StartTime` answers `ambiguous` across `proc_linux.go` and `proc_windows.go`. #511's two targets were per-build-constraint members at the time (`rusage_linux.go`/`rusage_other.go`, `allow_tmux.go`/`allow_none.go`).

### Advisor notes file

`ly:dbench-ohsm:discussion-advisor-1` wrote `_lyx/discussion/advisor-1.md` before batch 1: module borders, the PATTERN entries touched, the adjacent board entries, and the finding that #511's unexported premise does not reproduce.

### Batch 1 — items 1, 2, 5, 6

- Q1 gate shell: chair recommended awaiting `lyx gate test` shells and widening `awaitedLabel` to segment starts.
  `ly:dbench-ohsm:discussion-advisor-1` agreed: the only option that cannot report `done` with a tier unfinished; match after `&&`, `||`, `;`, `|`, skip `cd`, env assignments and `(`, never inside quotes, fall back to `HasPrefix`; a hung shell holds until `Spec.Timeout`; asked the chair to confirm `Label` is the full command (confirmed in `claudeengine/background.go`).
  Settled as D1 and D2.
- Q2 Master self-fix: chair recommended per-entry re-prompt text.
  `ly:dbench-ohsm:discussion-advisor-1` agreed: one line, caller data, generic fallback, point at `webster-body-verify-fix.md` rather than restate it, and also state the rule in the Master stencil section.
  Settled as D3 and D4.
- Q3 build constraints: chair recommended one `planglyph` disjointness predicate applied to every ambiguous reader, with item 5 reduced to a better message.
  `ly:dbench-ohsm:discussion-advisor-1` agreed, adding the implication rules (`unix`, `android`→`linux`, `ios`→`darwin`), an enumeration cap, unconstrained and test/non-test pairs never disjoint, and the risk that consumers assume one symbol per glyph.
  Settled as D8 and D9.

### Batch 2 — items 3, 4

- Q4 attribution: chair recommended recording recovery session ids per batch and auditing them in `loadRunWrites`, and widening reset and contract evidence too.
  `ly:dbench-ohsm:discussion-advisor-1` agreed with widening the shared reader, flagged the capture timing of the session id and asked whether `audit.go` treats recovery commits another way.
  The chair checked: `audit.go` does not read `RunWrites`, so the widening changes the recovery prompt, reset and contract evidence only; recovery writes go in their own group, never into `Master`, so contract evidence does not read them as Master's.
  Settled as D5.
- Q5 exit code: chair recommended `output.ErrFields` with the digest fields for `stuck` and `dead` without retry.
  `ly:dbench-ohsm:discussion-advisor-1` agreed: no caller keys on exit 0, Master's rungs stay keyed on `status`, reword "completes with" in the stencil, no `batch_terminal` field, spec row and a test in `verbs_test.go`.
  Settled as D6.
- Q6 allowed gates: chair recommended the allowance in the recovery prefix only and leaving `tmux` steps to verify.
  `ly:dbench-ohsm:discussion-advisor-1` agreed, recovery-only, and found no code reason a recovery cannot run a `tmux` gate, so the prompt states no reason.
  The chair found the plan format leaves the `tmux` tier to Publish's `publish_verify`, so "left to verify" means Publish, and the step is compiled with `go vet -tags tmux`.
  Settled as D7.

## Rejected alternatives

- Item 1, end the run stuck or paused when a gate shell is outstanding at an output-files turn end: stops runs that are usually fine.
- Item 1, only reword the friction note: leaves the shell killed at strand removal and the run reading `done`.
- Item 1, stop the shell explicitly: lyx has no pid for a provider shell.
- Item 2, a `lyx webster` verb that spawns the verify fixer: the fixer is an in-session Agent fork; a verb would be a second recovery mechanism.
- Item 2, stencil rewording only: the re-prompt Master acts on would still tell it to fix.
- Item 3, restore a clean tree before the second recovery and offer the first attempt's diff: a destructive step outside the fabric chokepoint that loses uncommitted work.
- Item 3, class any uncommitted path inside the batch's targets as own: misses deviations and misclassifies another writer's edits.
- Item 3, fold recovery writes into `RunWrites.Master`: contract evidence would read a recovery's writes as Master's own.
- Item 3b, a `batch_terminal: true` field: a second signal for the fact `status` and the exit code already carry.
- Item 4, the allowance in the shared implementer body: forks never misread the sandbox rule, so it would be padding there.
- Items 5–6, change quarry to honour build constraints: an external module outside this repo.
- Item 6, accept every ambiguous Create answer: passes a real duplicate declaration.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 gate shell hold vs stop — resolved, D1; advisor-informed (`ly:dbench-ohsm:discussion-advisor-1`).
- Q1b awaited-prefix matching at segment starts — resolved, D2; advisor-informed (`ly:dbench-ohsm:discussion-advisor-1`).
- Q2 per-entry gate re-prompt — resolved, D3 and D4; advisor-informed (`ly:dbench-ohsm:discussion-advisor-1`).
- Q3 build-constraint disjointness in `planglyph` — resolved, D8 and D9; advisor-informed (`ly:dbench-ohsm:discussion-advisor-1`).
- Q4 recovery attribution by recorded session — resolved, D5; advisor-informed (`ly:dbench-ohsm:discussion-advisor-1`).
- Q4b widen reset and contract evidence — resolved yes, recovery writes in their own group; advisor-informed (`ly:dbench-ohsm:discussion-advisor-1`).
- Q5 non-zero exit on a terminal batch — resolved, D6; advisor-informed (`ly:dbench-ohsm:discussion-advisor-1`).
- Q6 gate allowance in the recovery prefix — resolved, D7; advisor-informed (`ly:dbench-ohsm:discussion-advisor-1`).
- Q6b what "left to verify" means for `tmux` steps — auto-pick: Publish's `publish_verify`, since the plan's `## verify:` only compiles the tier.
- #507 second remedy (Create reusing a name the same card deletes) — auto-pick: out of scope; the board's expected outcome names only the ambiguity fix.
- #511 dead-batch revival through `goto` — auto-pick: out of scope; not in the board entry.
- When a recovery's session id is captured — open for the plan: at spawn if the shuttle API yields it, else from the recorded run directory (D5).
