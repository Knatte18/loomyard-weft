# Support log — report-timing

## Interview

Autonomous run: no operator answered; every pick is the writer's own and is recorded in the question ledger.
No advisors were seated.

### Turn 1 — scope and module boundary

Exploration: `internal/reportengine` already selects full runs from archive tags and reads batten snapshots, parent-review rounds, arbiter attempts per verdict word, alignment attempts per point and verdict, halt-note stops and LLM steps.
Its doc says the records cannot tell the loom driver from the orch.
`tools/tokencount` splits a run's span by shed history entry and subtracts sleep it reads from the systemd journal.
`internal/statuspane` holds `FormatSeconds`.
Picks: Q1, Q3; Q2 is a fact.

### Turn 2 — time model

Picks: Q4 to Q7.

### Turn 3 — settlers and data gaps

Picks: Q8 to Q11.

### Turn 4 — lens review

All six forks returned.

- goal-scope 1 (halted stops not counted as waits): fixed, `halt` wait kind added.
- goal-scope 2 (shed and driver vs the Go driver and tackler): fixed, settle records map to `shed` and `tackler`.
- goal-scope 3 (span start differs from the 2026-10-12 computation): fixed, the difference is stated in the rationale.
- goal-scope 4 (attempt time vs active time): fixed, attempt time lies inside its wait and is never added.
- goal-scope 5 (judge double count): fixed, a judge stop is a Bouncer bounce, which convergence counts as no stop, and it is counted apart.
- goal-scope 6 (Goal says read-only but writers change): fixed in Goal.
- pattern-fit 1 (shedverbs and shedengine cannot import agentname or read the env): fixed, the CLI layer classifies and passes a string.
- pattern-fit 2 (leaf-packages is a closed list): fixed, both packages join its background file.
- pattern-fit 3 (Go driver gotos get the wrong strand): fixed, the driver passes `shed` through its goto seam.
- pattern-fit 4 (vague agentname allowlist): fixed, `reportengine` gains `durfmt` only.
- pattern-fit 5 (tokencount outside spawn rule): rejected, no change needed; the rule binds only `reportcli`'s spawn.
- pattern-fit 6 (sandbox coverage of convergence unchecked): fixed, the wording no longer assumes it.
- board-fit 1 (Go driver rule table and tackler missing as settlers): fixed, merged with goal-scope 2.
- board-fit 2 (`driver` meaning under `orch-split`): fixed, pinned to the loom LLM driver role; a hub run driver gets its own label.
- board-fit 3 (`shed-arbiter-all` moves the arbiter): fixed, the arbiter read stays one shared site.
- board-fit 4 (overlap with `engine-usage-readers` on tokencount): fixed, named under Open risks.
- board-fit 5 (`run-verbs-shared`): rejected, labels are written in the namespace-neutral verb bodies and the CLI wrappers that move with them.
- board-fit 6 (`spawn-audit-log`): rejected, no conflict today.
- board-fit 7 (token totals out): no change, the reviewer confirmed it.
- premises 1 (no verb writes a resume entry): fixed, a resumed step records its label on the status, and the next entry carries it.
- premises 2 (reroute entries are producer outcomes): fixed, the label covers goto and resumed steps only.
- premises 3 (judge premise unclear): fixed, merged with goal-scope 5.
- premises 4 (snapshot needs a branch listing): fixed, stated.
- premises 5 (parent-review and seed stamps check out): no change.
- premises 6 (parent-review alignment records live in `parentreview`): fixed, both homes named.
- scenarios 1 (halt not a wait): fixed, merged with goal-scope 1.
- scenarios 2 (sleep and wait overlap): fixed, sleep subtracted first, with an acceptance criterion.
- scenarios 3 (non-monotonic stamps): fixed, zero interval plus finding.
- scenarios 4 (stale in-flight branch): fixed, live pairs only, an abandoned branch is a finding.
- scenarios 5 (open intervals in flight): fixed, they run to the snapshot time with no finding.
- scenarios 6 (batten source absent): fixed, `land` absent and span fallbacks stated.
- scenarios 7 (bucket time ambiguous): fixed, summed over runs.
- scenarios 8 (`--since` in the future): fixed, accepted and yields an empty report.
- scenarios 9 (reused slug in flight): fixed, snapshots after the newest tag.
- scenarios 10 (Go-side resume labels): fixed, merged with pattern-fit 3.
- leanness 1 to 7 and 9 (restated In list, assumptions, duplicates): fixed, the In list and assumptions now point at Decisions and the duplicates are cut.
- leanness 8 (interview turns copy the ledger): fixed, turns now point at the ledger.

## Rejected alternatives

- **A new `reporttiming` engine package.** Rejected: it would duplicate convergence's run selection and record reading, which the board asks to share.
- **One "by hand" class for orch, driver and operator.** Rejected: the board names them separately, and load moving between them is what the figure is for.
- **Reading the strand variable inside `shedverbs` or `shedengine`.** Rejected: a hidden input, and outside both packages' import bounds.
- **Reading sleep from the pause record.** Rejected: the record is ephemeral and holds pauses, not machine sleep.
- **Inferring hibernation from hub-wide quiet gaps.** Rejected: a long producer step writes no row, so a quiet threshold would subtract real work.
- **Reading Claude transcripts for producer activity.** Rejected: provider-specific (`PATTERN-shuttle-provider-seam`); `engine-usage-readers` owns usage reading.
- **Fixing the alignment `spawned_at` writer here.** Rejected: the root cause is unknown and it is a separate bug; the report bounds the bad stamp.
- **Importing `statuspane` from `reportengine`.** Rejected: it brings terminal and self-exec imports into a told-geometry engine.
- **Local-time day buckets.** Rejected: `convergence` uses UTC, and two reports disagreeing on a day's edge would confuse more than the two-hour shift.
- **Counting only awaiting stops as waits.** Rejected after the lens review: a halt left overnight would read as active time.
- **Folding `tackler` into `shed` or `driver`.** Rejected: it would hide which agent carries the load.
- **Importing `internal/sheddriver` from `reportengine` for the settle records.** Rejected: it brings the driver's lock, verb and render imports into a told-geometry engine, and its reader opens a file where the engine reads git objects.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): the report lives in `reportengine` and `reportcli` beside convergence, which shares selection and records.
- Q2 (resolved, fact): convergence counts arbiter outcomes per verdict word and alignment attempts per point and verdict; timing reuses those reads.
- Q3 (resolved, auto-pick): runs in flight are in the run table and buckets but not the medians, since day totals need them.
- Q4 (resolved, auto-pick): a row's interval runs from the previous row of either history, since `at` is a completion time.
- Q5 (resolved, auto-pick): a closed phase table plus `land`, and `other` with a finding.
- Q6 (resolved, auto-pick, revised after lens review): the waits are `parent_review`, `pr_gate`, `escalation` and `halt`, net of sleep, with overlaps counted once.
- Q7 (resolved, auto-pick): sleep comes from the systemd journal via a shared `internal/sleepjournal`.
- Q8 (resolved, auto-pick, revised after lens review): a caller label, classified in the CLI layer, is written on goto entries and on resumed steps.
- Q9 (resolved, auto-pick): arbiter and alignment time runs from `spawned_at` to the next row, and is absent outside the span in a landed run.
- Q10 (resolved, auto-pick): `internal/durfmt` holds the shared formatters.
- Q11 (resolved, auto-pick): a seven-day default window, `--by day`, UTC buckets.
- Q12 (resolved, auto-pick, from lens review): the Go driver's settle records map to `shed` (rule) and `tackler`, and `tackler` is added to the board's settlers.
- Q13 (resolved, auto-pick, from lens review): a records branch with no tag counts as in flight only for a live pair that the CLI names.
