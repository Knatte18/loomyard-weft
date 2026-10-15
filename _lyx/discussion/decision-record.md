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
Medians cover only landed runs whose tag falls in the window; a run in flight and a partial run never enter a median, and a partial run is in neither the run table nor any bucket.
Day and week totals clip each interval to the window and to the bucket it falls in, so no bucket before `--since` is printed; buckets are UTC days and ISO weeks, as in `convergence`.
A bucket's time is run-time summed over runs, so two parallel runs add two hours to an hour.
Rationale: day totals without runs in flight miss the day's work; an abandoned branch must not grow forever.

### Run span and intervals

Decision: a run's span opens at batten's first row, or at the seed's `started_at` when the batten source is absent, or at the first loom row with a finding when both lack a stamp.
It closes at the last stamp (`at`, or `last_at` on a folded entry) of its loom and batten histories, or at the snapshot time for a run in flight.
The loom and batten histories merge in stamp order, and each row's interval runs from the latest merged stamp seen so far to its own, since `at` is a completion time.
A stamp earlier than the latest stamp so far gives a zero interval and a finding, and the next row's interval still starts from that latest stamp, so no time is counted twice.
Rationale: the per-row split of the 2026-10-12 computation; opening at batten's first row adds the time before `Seed-Child` to `setup`, which that computation left out.

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

A batten-row halt (see Stops by settler) is a wait of the same kind its state gives, `escalation` for an awaiting one and `halt` otherwise, from its own batten entry to the next row of either history.

Waits stay inside their phase's time and are also reported by kind.
Sleep is subtracted first; waits are net of sleep, and overlapping waits count once, under the kind listed first above.
Active time is the span minus sleep minus waits, so active plus waits plus sleep equals the span.
An open wait in a run in flight runs to the snapshot time.
Rationale: the board's "waiting on the parent or orch" covers every stop someone else must move on; a halt left overnight otherwise inflates active time.

### Sleep

Decision: sleep intervals come from the systemd journal, read by `reportcli` once per invocation through `journalctl` and handed to the engine as plain intervals in its options; the engine runs no process.
The journal parsing in `tools/tokencount/sleep.go` moves to `internal/sleepjournal`, which `tokencount` and `reportcli` import.
Sleep, clipped to the run's span, is left out of every row interval, and so out of phase, wait and attempt time; it is reported per run and per bucket as its own figure, clipped to the bucket as other intervals are.
A missing or failing journal subtracts nothing and the output's `sleep_note` says why.
The `journalctl` spawn is logged through `internal/logger`.
Rationale: lyx records no sleep durably; the journal holds every suspend and hibernate with both ends.

### Arbiter and alignment time

Decision: every attempt is one spawn, counted per run and per bucket: arbiter attempts, PR-Gate alignment attempts under `internal/alignment`, and parent-review alignment records under `internal/parentreview`.
An attempt's time runs from its `spawned_at` to the first row of either history stamped after it, or to the snapshot time in a run in flight.
In a landed run, an attempt whose `spawned_at` falls outside the span, or has no row after it, has its time absent and adds a finding; its spawn still counts.
Attempt time lies inside the wait it serves and is reported beside it, never added to it.
Rationale: no attempt record carries an end time, and the next row is the applied verdict's or the hand-back's.

### Stops by settler

Decision: a stop is a halt or awaiting stop as `convergence` lists it, a Bouncer `Stuck` entry that `OnStuck` bounced, or a batten-row halt.
A batten-row halt is a batten history entry at one of batten's own rows, never `Run-Shed`, whose outcome left batten blocked or awaiting: a `stuck` entry no `OnStuck` bounced, or an `awaiting` entry.
`convergence` counts neither of the last two as a stop, so the report counts each of the three kinds apart and its halt count matches `convergence`.
Each stop has one settler, the first that holds:

1. `judge`: the stop is a Bouncer bounce;
2. `shed`: a batten automatic step accounts for it, or a Go driver settle record with settler `go-rule` and action `restep` or `verify-retry` names the stop;
3. `tackler`: a settle record with settler `tackler` and action `restep` names the stop;
4. `arbiter`: an applied arbiter attempt accounts for it;
5. `alignment`: an applied alignment verdict accounts for it;
6. the `by` label of the goto or resume that moved it on, read as its own value: `orch`, `driver`, `operator` or another role, except that the label `shed` reads as class `shed`, so a move by the Go driver or batten that no settle record covers reads `shed`;
7. `unknown`: moved on by an entry or resume without a label, as every run before this task;
8. `open`: still open in a run in flight.

A `go-rule` record with action `moved`, `busy-exit`, `running-exit` or `hand-over` settles nothing and leaves the stop to the later items.
A `spawn` record of any settler, a `parent` `hand-back` record and every `arbiter` record settle nothing by themselves; an arbiter settle is read from its applied attempt in item 4.
The settle record type, with its stop key and its settler and action names, and a tolerant decoder over the file's bytes move from `internal/sheddriver` to `internal/shedrun`, which already names the file and is on `reportengine`'s allowlist; a line that does not decode is a finding, and `sheddriver` keeps the file's append and read.
The report reads two settle-record files, the loom run's and batten's, each under its own run directory, at the tag, or at the records branch tip for a run in flight.
A record not yet committed at that tip leaves its stop `open` rather than adding a finding.
The loom file's records name the child's stops and batten's file's records name batten-row halts.
A record's stop key holds the length the driven history had when the driver saw the halt, so its last counted entry is the entry that halted.
A batten-file record names a batten-row halt when its key's producer and state equal the halt's and its history length is the position of the halt's own entry in batten's history.
A loom-file record names a child stop that carries a change when its key's producer and state equal the stop's, and its history length is the greatest among such records whose last counted loom entry is stamped at or before the stop's change.
Batten writes the change when it observes the child, which can lag the halt: a halt the Go driver restepped before batten looked, and that halted again, leaves an earlier record with the same producer and state, which folds into the same stop and adds no finding.
A stop without a change, a note-only stop, is matched on producer and state alone against the loom file's records, taking the earliest one no other stop took; when none is left, its settler is read from the later items.
A change whose producer is empty matches no record and falls to the later items.
Any other record matching no stop is a finding.
Rationale: the board's settlers, plus `tackler`, which settles stops without an LLM parent.

### The caller label

Decision: the goto CLI and the step CLI that resumes a halted run classify the caller from the strand variable through `internal/agentname`, in the CLI layer, and pass the label down explicitly; `shedverbs` and `shedengine` take it as a plain string and read no environment.
Labels: `orch` for the orch's strand, `driver` for the loom LLM run driver's role, the role for any other strand, `operator` for an empty strand.
The Go driver passes `shed` through both its `Deps.Goto` and `Deps.Step` seams, and batten's automatic steps pass `shed`.
A goto writes the label as an optional `by` field on its history entry.
A step on a halted run records its label on the status; the next history entry the run writes carries it as `by` and clears it.
Both fields are tolerant: records without them decode as before.
Bound: the label is for reports only; no Go decision reads it, and an unrecognised strand is recorded as its raw role, never refused.
A later hub-level run driver gets its own label, not `driver`.
Rationale: without the label, the records cannot tell the orch from the operator or the LLM driver; reading the strand in the CLI keeps the shed packages within their import bounds.

### Seconds

Decision: every duration in markdown renders through the formatter the status pane uses for closed spans: with seconds below a day, `45s`, `12m05s`, `1h05m12s`, and from a day on as `2d03h`, without seconds; JSON carries integer seconds.
`FormatSeconds` and `FormatMinutes` move from `internal/statuspane` to `internal/durfmt`, which both import.
Rationale: the operator's 2026-10-15 decision; one formatter prints the same text for the same span.

### Output

Decision: JSON is `{"ok":true}` with `selection`, `runs`, `phases` (medians), `buckets`, `sleep_note` and `findings`.
Markdown is the selection, a per-run table, a phase-median table, a bucket table and a stops-by-settler table per bucket; the run table and the bucket table each carry a sleep column.
Both exit 0 whatever findings hold; a window with no runs prints empty tables.
Rationale: `convergence`'s envelope shape and finding policy.

## Constraints

- `PATTERN-told-geometry`: `reportengine` never imports `lyxcwd`; its allowlist gains `internal/durfmt` only, the settle-record format moves into the already-allowed `internal/shedrun`, and live pairs and sleep intervals are told by `reportcli`.
- `PATTERN-cliwire-sole-wiring`, `PATTERN-cwd-resolution`: `reportcli` resolves the prime as `convergence` does.
- `PATTERN-shed-verb-set` and `PATTERN-shed-producer-seam`: neither `shedverbs` nor `shedengine` gains an import; the label arrives as a string.
- `PATTERN-shed-driver`: the Go driver's label is passed through its existing goto and step seams.
- `PATTERN-tolerant-records`: the `by` field and the status's pending label are optional.
- `PATTERN-agent-name`: strand parsing goes through `internal/agentname`.
- `PATTERN-spawn-observability`: the `journalctl` spawn is logged.
- `PATTERN-leaf-packages`: `internal/durfmt` and `internal/sleepjournal` join its background file as stdlib-only leaves in the same commit.
- `PATTERN-fabric-git`: pair records branches are named through `fabricengine`'s exports and read through `gitrepo`.
- `PATTERN-sandbox-coverage`: the new verb is exercised by the sandbox suite or excluded with a reason.
- Test budget: a package pushed past its `test-budget.yaml` row lists that file.
- Docs land in the same commit: `reportengine` and `reportcli` `doc.go`, the shed docs for the label, `statuspane`'s doc for the moved formatters, `tokencount`'s package doc for the moved parser, `shedrun`'s and `sheddriver`'s docs for the moved settle-record format.

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

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - batten-file-record-matches-no-stop (class decision, severity MEDIUM, open in rounds 3)
  - interval-from-previous-stamp-recovers-time (class consistency, severity MEDIUM, open in rounds 3)
  - settle-record-to-stop-join-unstated (class design, severity MEDIUM, open in rounds 2, 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- For a landed run holding both histories, its phase seconds, `other` included, plus its sleep sum to its span, and active plus waits plus sleep equals the span.
- A row's interval runs from the latest earlier row of either history; a batten row between two loom rows splits the gap.
- A folded entry's `last_at` closes its interval.
- A stamp earlier than the latest stamp so far gives a zero interval and a finding, and the next interval starts from that latest stamp.
- An unknown row name counts under `other` with a finding.
- A run without the batten source shows `land` absent and opens at the seed's `started_at`.
- Overlapping waits count once, under the first kind in the list; an awaiting stop at `PR-Gate` is `pr_gate`, one elsewhere `escalation`, a blocked or failed stop `halt`.
- A sleep inside a wait counts as sleep and not also in the wait; a failed or absent journal subtracts nothing and sets the note.
- An interval spanning UTC midnight splits across both day buckets; `--by week` buckets by ISO week.
- A run in flight appears in the run table and buckets, never in a median; its open wait, open attempt and open stop run to the snapshot time, with no finding.
- A records branch with no tag and no live pair is left out with a finding.
- An in-flight run under a reused slug reads only the batten snapshots after the slug's newest tag.
- Each settler class is reached by a stop, including an entry without `by` reading `unknown`.
- A stop whose only `go-rule` record is `moved`, followed by a goto with `by` `orch`, reads `orch`.
- A loom-file `go-rule` `restep` record whose key matches a batten child change reads `shed`, and a batten-file record names only a batten-row halt, which is counted apart from `convergence`'s halts.
- A halt the Go driver restepped into a second halt before batten observed it reads its settler from the second halt's record, and the first halt's record adds no finding.
- A note-only stop takes its settler from a loom-file record with its producer and state, and a change with an empty producer reads its settler from the later items.
- A goto with `by` `shed` that no settle record covers reads `shed`.
- A landed run's attempt whose `spawned_at` lies outside its span has absent time, a finding and a counted spawn.
- A goto and a resumed step write `by` per caller class, the Go driver's goto and resume write `shed`, and an entry from an older binary decodes and reads `unknown`.
- Each flag refusal names the flag, the value and the accepted form, before any read; a `--since` after today gives an empty report.
- Markdown prints every duration in the status pane's format: with seconds below a day, as `2d03h` from a day on.
- `tokencount`'s sleep subtraction and the status pane's text are unchanged after the moves.

## Notes for the plan writer

- `tools/tokencount/shed.go` splits a run's span by one shed history, with sleep left out of each interval, and is the model for the per-row split; this report merges the loom and batten histories and opens the span at batten's first row, as Run span and intervals states.
- The Go driver's settle records are `shedrun.SettleRecords` (`go-settle.jsonl`), committed with the run directory.
- `internal/hibernateengine` classifies callers by strand the same way the caller label does.
