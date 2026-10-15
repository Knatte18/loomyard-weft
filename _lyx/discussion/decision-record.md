# Decision record — report-timing

## Goal

`lyx report timing` prints, mechanically from the hub's run records, where run time went: per run, per phase, per day or week, and who settled each stop.
It replaces the LLM-and-awk computation of 2026-10-12 with a read-only verb beside `lyx report convergence`, sharing its record reading.
To tell the orch, the LLM driver and the operator apart, the task also records the caller on goto entries and on resumes of a halted run, a writer change in the shed verbs.

## Scope

In: the decisions below, which cover the verb, its engine, the caller label written by the shed verbs, and two small package moves (`internal/durfmt`, `internal/sleepjournal`).

Out:

- Token totals: the board's `engine-usage-readers` entry owns reading usage through the engine.
- Repairing the alignment attempt's unreliable `spawned_at` writer (see Open risks).
- Any change to `lyx report convergence`'s output.
- Hub pauses: the pause record keeps no history, so paused time stays in whichever phase it fell.

## Decisions

### The verb

Decision: `lyx report timing [--since <date>] [--by day|week] [--format json|markdown]`, prime-only like `convergence`, read-only, no lock, no commit.
`--since` is a UTC `YYYY-MM-DD` date defaulting to seven days before today's UTC date; a date after today is accepted and yields an empty report.
`--by` defaults to `day`; `--format` defaults to `json`.
Flag refusals follow `convergence`: a JSON error, exit 1, naming the flag, the value and the accepted form, checked before any cwd resolution.
Rationale: the operator reads recent days; one calling shape for both reports.

### One engine, one selection

Decision: timing is a second compute entry in `internal/reportengine` over the same `Sources` snapshot, reusing its archive-tag run selection and its reads of the loom seed and status, batten snapshots, parent-review rounds, arbiter and alignment attempts and stops.
A record that does not decode is a finding, and a figure a run's records lack is absent, never zero, as in `convergence`.
The arbiter and alignment reads stay one site each, shared by both reports, so the board's `shed-arbiter-all` move of the arbiter into shed edits one reader.
Rationale: the board asks to share convergence's record reading; one selection means both reports agree on which runs exist.

### Run selection and windows

Decision: a landed run is a full run as `convergence` defines it.
A run in flight is a pair the CLI tells the engine is live, whose records branch exists and has no archive tag at its tip.
The snapshot grows a listing of those branches, resolved once with the other tips, through `fabricengine`'s records-branch naming.
A records branch with no tag and no live pair is abandoned: it is left out and adds a finding.
An in-flight run's batten snapshots are those committed after the slug's newest archive tag.
A run enters the report when its span overlaps the window from `--since` to the snapshot time.
Medians cover only landed runs whose tag falls in the window; a run in flight and a partial run never enter a median, and a partial run is not in the run table.
Day and week totals clip each interval to the bucket it falls in; buckets are UTC days and ISO weeks, as in `convergence`.
A bucket's time is run-time summed over runs, so two parallel runs add two hours to an hour.
Rationale: day totals without runs in flight miss the day's work; an abandoned branch must not grow forever.

### Run span and intervals

Decision: a run's span opens at batten's first row, or at the seed's `started_at` when the batten source is absent, or at the first loom row with a finding when both lack a stamp.
It closes at the last stamp (`at`, or `last_at` on a folded entry) of its loom and batten histories, or at the snapshot time for a run in flight.
The loom and batten histories merge in stamp order, and each row's interval runs from the previous merged stamp to its own, since `at` is a completion time.
A stamp earlier than the previous one gives a zero interval and a finding.
Rationale: the 2026-10-12 computation's interval rule; opening at batten's first row adds the time before `Seed-Child` to `setup`, which that computation left out.

### Phases

Decision: one closed table in `reportengine` maps every row name to a phase:

| Phase | Rows |
|---|---|
| setup | batten rows before `Run-Shed`, `Preflight`, `Loom-Preflight` |
| discussion | every `Discussion-*` row |
| plan | every `Plan-*` row, `Batchifier` |
| webster | every `Webster*` row, `Describe` |
| merge | every `Merge-*` row |
| publish-finalize | `Publish`, `PR-Align`, `PR-Gate`, `PR-Rework`, every `Darn-*` row, `Finalize`, `Friction-Reflect` |
| land | batten rows after `Run-Shed` |

A row the table does not know counts under `other` and adds a finding naming it.
Without the batten source, `setup`'s batten part and `land` are absent.
Rework counts in the phase of the producer that re-ran, which the per-row mapping gives.
Rationale: the board's phases, plus `land`, since batten's Land delay after Friction-Reflect is in batten's own history.

### Waits

Decision: a wait is any of:

- `parent_review`: a parent-review round, from its request's `opened_at` to its verdict's `recorded_at`;
- `pr_gate`: an awaiting stop at `PR-Gate`, from batten's change into awaiting to the next row of either history;
- `escalation`: any other awaiting stop, measured the same way;
- `halt`: a blocked or failed stop, from batten's change into it to the next row of either history.

Waits stay inside their phase's time and are also reported by kind.
Sleep is subtracted first; waits are net of sleep, and overlapping waits count once, under the kind listed first above.
Active time is the span minus sleep minus waits, so active plus waits plus sleep equals the span.
An open wait in a run in flight runs to the snapshot time.
Rationale: the board's "waiting on the parent or orch" covers every stop someone else must move on; a halt left overnight otherwise inflates active time.

### Sleep

Decision: sleep intervals come from the systemd journal, read by `reportcli` once per invocation through `journalctl` and handed to the engine as plain intervals in its options; the engine runs no process.
The journal parsing in `tools/tokencount/sleep.go` moves to `internal/sleepjournal`, which `tokencount` and `reportcli` import.
Sleep is subtracted from every interval it overlaps and reported per run; a missing or failing journal subtracts nothing and the output's `sleep_note` says why.
The `journalctl` spawn is logged through `internal/logger`.
Rationale: lyx records no sleep durably; the journal holds every suspend and hibernate with both ends.

### Arbiter and alignment time

Decision: every attempt is one spawn, counted per run and per bucket: arbiter attempts, PR-Gate alignment attempts under `internal/alignment`, and parent-review alignment records under `internal/parentreview`.
An attempt's time runs from its `spawned_at` to the first row of either history stamped after it, or to the snapshot time in a run in flight.
In a landed run, an attempt whose `spawned_at` falls outside the span, or has no row after it, has its time absent and adds a finding; its spawn still counts.
Attempt time lies inside the wait it serves and is reported beside it, never added to it.
Rationale: no attempt record carries an end time, and the next row is the applied verdict's or the hand-back's.

### Stops by settler

Decision: a stop is a halt or awaiting stop as `convergence` lists it, or a Bouncer `Stuck` entry that `OnStuck` bounced, which `convergence` counts as no stop; the report counts the two apart so its halt count matches `convergence`.
Each stop has one settler, the first that holds:

1. `judge`: the stop is a Bouncer bounce;
2. `shed`: a batten automatic step accounts for it, or a Go driver settle record shows a rule acted on it;
3. `tackler`: a settle record shows an applied tackler resume;
4. `arbiter`: an applied arbiter attempt accounts for it;
5. `alignment`: an applied alignment verdict accounts for it;
6. the `by` label of the goto or resume that moved it on: `orch`, `driver`, `operator` or another role;
7. `unknown`: moved on by an entry or resume without a label, as every run before this task;
8. `open`: still open in a run in flight.

Rationale: the board's settlers; `tackler` is added because the Go driver's tackler settles stops without an LLM parent, and folding it into `shed` or `driver` would hide which agent carries the load.

### The caller label

Decision: the goto CLI and the step CLI that resumes a halted run classify the caller from the strand variable through `internal/agentname`, in the CLI layer, and pass the label down explicitly; `shedverbs` and `shedengine` take it as a plain string and read no environment.
Labels: `orch` for the orch's strand, `driver` for the loom LLM run driver's role, the role for any other strand, `operator` for an empty strand.
The Go driver's own goto passes `shed` through its `Deps.Goto` seam, and batten's automatic steps pass `shed`.
A goto writes the label as an optional `by` field on its history entry.
A step on a halted run records its label on the status; the next history entry the run writes carries it as `by` and clears it.
Both fields are tolerant: records without them decode as before.
Bound: the label is for reports only; no Go decision reads it, and an unrecognised strand is recorded as its raw role, never refused.
A later hub-level run driver gets its own label, not `driver`.
Rationale: without the label, the records cannot tell the orch from the operator or the LLM driver; reading the strand in the CLI keeps the shed packages within their import bounds.

### Seconds

Decision: every duration in markdown renders with seconds, `45s`, `12m05s`, `1h05m12s`, through the formatter the status pane uses for closed spans; JSON carries integer seconds.
`FormatSeconds` and `FormatMinutes` move from `internal/statuspane` to `internal/durfmt`, which both import.
Rationale: the operator's 2026-10-15 decision; one formatter prints the same text for the same span.

### Output

Decision: JSON is `{"ok":true}` with `selection`, `runs`, `phases` (medians), `buckets`, `sleep_note` and `findings`.
Markdown is the selection, a per-run table, a phase-median table, a bucket table and a stops-by-settler table per bucket.
Both exit 0 whatever findings hold; a window with no runs prints empty tables.
Rationale: `convergence`'s envelope shape and finding policy.

## Constraints

- `PATTERN-told-geometry`: `reportengine` never imports `lyxcwd`; its allowlist gains `internal/durfmt` only, and live pairs and sleep intervals are told by `reportcli`.
- `PATTERN-cliwire-sole-wiring`, `PATTERN-cwd-resolution`: `reportcli` resolves the prime as `convergence` does.
- `PATTERN-shed-verb-set` and `PATTERN-shed-producer-seam`: neither `shedverbs` nor `shedengine` gains an import; the label arrives as a string.
- `PATTERN-shed-driver`: the Go driver's label is passed through its existing goto seam.
- `PATTERN-tolerant-records`: the `by` field and the status's pending label are optional.
- `PATTERN-agent-name`: strand parsing goes through `internal/agentname`.
- `PATTERN-spawn-observability`: the `journalctl` spawn is logged.
- `PATTERN-leaf-packages`: `internal/durfmt` and `internal/sleepjournal` join its background file as stdlib-only leaves in the same commit.
- `PATTERN-fabric-git`: pair records branches are named through `fabricengine`'s exports and read through `gitrepo`.
- `PATTERN-sandbox-coverage`: the new verb is exercised by the sandbox suite or excluded with a reason.
- Test budget: a package pushed past its `test-budget.yaml` row lists that file.
- Docs land in the same commit: `reportengine` and `reportcli` `doc.go`, the shed docs for the label, `statuspane`'s doc for the moved formatters, `tokencount`'s package doc for the moved parser.

## Auto-mode assumptions

- Default window of seven days (see The verb).
- UTC buckets (see Run selection and windows).
- Runs in flight are in (see Run selection and windows).
- `land` and `tackler` are added to the board's lists (see Phases, Stops by settler).
- Recording the caller label is in scope (see The caller label).

## Open risks

- Alignment `spawned_at` stays unreliable until its writer is fixed; a stamp outside the span shows absent time and a finding, but a late stamp inside the span shortens the attempt's time silently.
- A machine powered off rather than suspended, or a journal whose retention dropped the window, counts that time as run time.
- Every run before this task reads its orch, driver and operator settles as `unknown`.
- `engine-usage-readers` also rewrites `tools/tokencount`; whichever lands second rebases onto the moved sleep parser.

## Acceptance criteria

- For a landed run holding both histories, its phase seconds, `other` included, sum to its span, and active plus waits plus sleep equals the span.
- A row's interval runs from the previous row of either history; a batten row between two loom rows splits the gap.
- A folded entry's `last_at` closes its interval.
- A stamp earlier than its predecessor gives a zero interval and a finding.
- An unknown row name counts under `other` with a finding.
- A run without the batten source shows `land` absent and opens at the seed's `started_at`.
- Overlapping waits count once, under the first kind in the list; an awaiting stop at `PR-Gate` is `pr_gate`, one elsewhere `escalation`, a blocked or failed stop `halt`.
- A sleep inside a wait counts as sleep and not also in the wait; a failed or absent journal subtracts nothing and sets the note.
- An interval spanning UTC midnight splits across both day buckets; `--by week` buckets by ISO week.
- A run in flight appears in the run table and buckets, never in a median; its open wait, open attempt and open stop run to the snapshot time, with no finding.
- A records branch with no tag and no live pair is left out with a finding.
- An in-flight run under a reused slug reads only the batten snapshots after the slug's newest tag.
- Each settler class is reached by a stop, including an entry without `by` reading `unknown`.
- A landed run's attempt whose `spawned_at` lies outside its span has absent time, a finding and a counted spawn.
- A goto and a resumed step write `by` per caller class, the Go driver's goto writes `shed`, and an entry from an older binary decodes and reads `unknown`.
- Each flag refusal names the flag, the value and the accepted form, before any read; a `--since` after today gives an empty report.
- Markdown prints every duration with seconds in the status pane's format.
- `tokencount`'s sleep subtraction and the status pane's text are unchanged after the moves.

## Notes for the plan writer

- `tools/tokencount/shed.go` already splits a run's span by shed history entry; its interval rule is the one this report uses.
- The Go driver's settle records are `shedrun.SettleRecords` (`go-settle.jsonl`), committed with the run directory.
- `internal/hibernateengine` classifies callers by strand the same way the caller label does.
