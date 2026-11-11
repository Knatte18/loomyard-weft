## Interview

Autonomous run: no operator answered.
Every pick below is an auto-pick, recorded in the question ledger.

### Turn 1 — reading

Read the board entry, the project `test-suite-trim`, the touching entries `testmap-cross-package`, `coverage-test-select`,
`gate-scope-coverage`, `testmap-backlog-drain` and `redundancy-sweep-existing`,
the stored Sketch records of the three build tasks, the research report and its three proposals.
Three subagents read testmap, the dobby trigger, gateslot's pool, the redundancy step and the config-retirement mechanism.
`lym loom peers` shows `testmap-cross-package` running at Darn-Write with no changed path.

Today's code, as read: a map test gets an entry only on a pass that covers a block;
a failed build replaces the map with a failure-only map; the exit record's `Unbuilt` has no production reader;
the slot is held from the first build to the end of the call; an anchor move waits for a running refresh;
the redundancy step counts a spawn-marked map test as a cover and refuses a map with `BuildFailure` set.

### Turn 2 — the staleness model

Chose one `Commit` per map moved by the selector's verdict at HEAD (proposal O's three outcomes),
with B's gaps for affected tests so the map reaches HEAD at once.
A's per-entry commit was rejected: it is the largest change to the record and to the selector, for a gain only lagging worktrees see.

### Turn 3 — the entry's five questions

- Linked-only change: current exactly when the selector finds no affected test and no miss (D1).
- Failed build: keep the old map (D5).
- Spawn-marked tests: never measured (D4).
- Another package's coverage: by function (D3).
- Other entries: D13.

### Turn 4 — slot use, trigger, order

Non-blocking take only while another slot stays free, yield by killing in-flight runs (B), no quiet-pool wait (O's starvation argument),
work trigger replacing the landing count, order by run-record seconds with no package left out.
Found that an anchor move waits for a running refresh; with no batch cap that wait is unbounded, so D11 ends upkeep instead.

### Turn 5 — lens review

Six forks returned reports; each is saved beside this log.

- bound-breaker, interactions, walkthrough (BLOCKING): the audit never runs, since a pass ends on an empty queue. Fixed: every pass ends with one audit round (D7), with its bound.
- existing-paths, walkthrough, bound-breaker: a re-stamp leaves the marker's `Commit` behind, so every re-stamped map misses. Fixed: every `Commit`-moving write sets the marker (D1).
- existing-paths: D5 reusing `BuildFailure` makes the redundancy step refuse a usable map. Fixed: the failure goes in a field of its own; `BuildFailure` keeps today's meaning (D5).
- interactions, walkthrough: a failed-at-HEAD package and spawn-marked tests count as queued work and restart passes in a loop. Fixed (D4, D5, D8).
- interactions: a Current verdict and the outcome-entry drop rule disagreed. Fixed: an outcome entry drops only on a miss or an own-file change (D4).
- interactions, existing-paths, bound-breaker: a hand call overwrites the exit record and stalls the backlog. Fixed: only a dobby-started pass writes it (D8).
- interactions, existing-paths, bound-breaker: a pass waiting for a slot ignores landings and never ends on a one-slot hub. Fixed: HEAD read on each wake; a one-slot hub's pass plans and ends (D8, D9).
- bound-breaker (BLOCKING): retiring `test_map_every` drops its 0 off switch. Fixed: `test_map_upkeep` and a reinterpretation of 0 (D8).
- bound-breaker, walkthrough (BLOCKING): D13's carrier said "before Plan", but the build tasks are darn runs with no Plan. Fixed: before Darn-Write's writer starts (D13); T4's running status stays a parent question under Open risks.
- bound-breaker: a run or build killed by a yield or signal read as a failure. Fixed (D4).
- bound-breaker: own-package non-Go files had no rule once the age limit goes. Fixed: recorded and compared as a linked package's (D2, D3).
- existing-paths, walkthrough: a hand call skips a package a pass holds locked, so the sweep could judge a stale map. Fixed: a hand call waits for the lock (D9).
- existing-paths: a call under an inherited slot. Fixed: neither takes nor yields (D9).
- existing-paths: an unregistered pass lets a second start. Fixed: bound stated (D8).
- interactions: a tuned `parallel` changes every stamp. Fixed: digest before substitution (D6).
- bound-breaker: D10's discard missed added files. Fixed: discard on a HEAD move or a dirty linked file (D10).
- interactions: the plan's handling of a held lock was unstated. Fixed: it skips the map (D1).
- walkthrough: the first run with no exit record. Fixed (D8).
- walkthrough: the D9 criterion wording. Fixed.
- interactions, walkthrough, leanness: D13's build split was wrong and duplicated in Notes. Fixed: one sentence on what orders after the selector; Notes cut.
- board-scope: `coverage-test-select`'s D5 Bound names the age limit. Fixed (D13).
- board-scope (decision): whether `testmap-cross-package`'s run is held. Kept under Open risks for the parent.
- board-scope: no run exists yet for two of the entries. Fixed with the darn carrier wording (D13).
- leanness: Goal's question map, the today's-code Constraint, the `slots: 3` Constraint, four restating auto-mode assumptions, D1's re-derivation and the Notes bullet. Fixed: cut or moved here and into D9.
- leanness: the Review criterion checked parent actions. Fixed: it now checks D13's text.
- leanness: Turn 3 repeats the ledger. Rejected: the support log is not read by Plan, and the turn records the order of the interview.

## Rejected alternatives

- Per-entry commits with up to two entries per test (proposal A): largest change to the map and the selector; one `Commit` with gaps serves the gate.
- A request-file yield protocol in `gateslot` (A): new pool state; checking `Holders` and `Draining` per poll bounds a gate's wait the same.
- A pinned checkout for measuring (O): a new fabric verb and a second checkout; per-round writes shrink the prime's discard cost instead.
- Waiting for a quiet pool before starting (A, B): starves the queue at this hub's landing rate.
- Skipping upkeep for cheap packages (O): the redundancy step reads every package's map.
- A 30-day age backstop (B, O): the audit measures what a backstop would hide, and the landing gate bounds any wrong entry.
- An audit started by the dobby on idle ticks: a new trigger and record field; one round per pass audits as often as maps change.
- Running every test side by side with a retry alone (A, B): PATTERN-test-speed lets a test without `t.Parallel()` touch shared fixtures.
- Measuring spawn-marked tests on a hand call: no consumer reads their blocks but the advisory redundancy step.
- Dropping outcome entries on any recorded-file change (B): it re-measures failing tests after nearly every landing; the gate runs them anyway.
- Reusing `BuildFailure` for a failure beside kept entries: the redundancy step and the selector read it as an unusable map.
- Retiring `test_map_every` with no successor: it removes the only off switch.
- Sharing maps across machines: all three proposals rank it last, and nothing yet needs it.
- Narrowing the non-body miss in this design: a selector change needing its own soundness review; filed as a note (D13).
- A `--remeasure` flag (B): a hand call already rebuilds the named packages whole.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): one `Commit` per map or per entry? One, moved by the plan (D1).
- Q2 (auto-pick, resolved): may a map hold gaps? Yes; D3 of `gate-scope-coverage` runs unrecorded tests (D1).
- Q3 (auto-pick, resolved): does a linked-only change leave a map fresh? Only when the selector finds nothing (D1).
- Q4 (auto-pick, resolved): own-package files by function, test files by `Test` function, own non-Go files as linked? Yes, in the shared selector (D2).
- Q5 (auto-pick, resolved): linked coverage by function or block? Function (D3).
- Q6 (auto-pick, resolved): skip spawn-marked tests? Yes, always (D4).
- Q7 (auto-pick, resolved): outcome entries for failing tests? Yes, dropped on a miss or an own-file change (D4).
- Q8 (auto-pick, resolved): failed build keeps the old map? Yes, in a failure field of its own (D5).
- Q9 (auto-pick, resolved): age backstop? None; stamp, self-check and an audit round per pass (D6, D7).
- Q10 (auto-pick, resolved): trigger? Remaining work, with `test_map_upkeep` as the off switch (D8).
- Q11 (auto-pick, resolved): slot rule? Non-blocking, another slot free, yield per poll (D9).
- Q12 (auto-pick, resolved): which tests side by side? Those opening with `t.Parallel()` (D10).
- Q13 (auto-pick, resolved): prime or pinned checkout? Prime (D12).
- Q14 (auto-pick, resolved): measurements before the design? None; D12 reports them live.
- Q15 (open, for the parent): `testmap-cross-package` runs at Darn-Write although held; see Open risks.
