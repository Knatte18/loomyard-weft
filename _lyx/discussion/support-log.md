# Support log: convergence-plan

## Interview

Autonomous session: no operator answered; every pick below is the writer's own and is logged in the question ledger.

### Turn 1 — sources

Read the board entry and the entries it names (`orch-split`, `go-driven-shed`, `twill-plan-followups`, `kickstart-pack-measure`, `run-robustness`, `watchdog-bare-reason`, `spawn-audit-log`, `tokencount-cutoff-skew`).
Read an archive tag's tree in the hub's records store: decision record, plan, review rounds per segment, parent-review rounds, arbiter attempts, drive reports, friction and halt notes, webster records.
Found that orch notices are files removed on delivery, and the only lasting log of them, the hub's notice log, is untracked, so no git object records a notice; batten's history on the prime's records branch records each child state change; halt notes are Go-authored with `anomaly`, `state` and `producer`; drive reports are written by the LLM driver.

### Turn 2 — design

Settled D1 to D12 as auto-picks (ledger Q1 to Q9).

### Turn 3 — lens review

Six lens forks spawned before the files were written; all six returned.

- premises, BLOCKING: suppressed batten notices still leave state-change entries. Fixed: D6 classifies them as automatic steps through a `battenshed` reader.
- premises, MEDIUM: batten misses halts resumed between reads. Fixed: Open risks.
- premises, MEDIUM: halt notes exist only for blocked and failed, so awaitings flood `unclassified`. Fixed: D7 classes awaitings as `awaiting`, the class comes from `ClassifyHalt`'s anomaly.
- premises and pattern-fit, MEDIUM: `discussionparser` has no added-decision reader. Fixed: D9 adds it.
- premises and pattern-fit, MEDIUM/LOW: `gitrepo` has no first-parent walk or blame. Fixed: D3 adds both.
- premises, LOW: parent-review delivery rule vague. Fixed: D6 names `alignment.json` and `delivery.json`.
- premises, LOW: verdict-file readers unverified. Fixed: D9 adds a reader in the writing package where none exists.
- scenarios, BLOCKING: the batten-record bound picks the next run's history. Fixed: D4 bounds at the next fresh seed.
- scenarios, MEDIUM: partial and bench runs counted. Fixed: D4 counts them as `partial`.
- scenarios, MEDIUM: no halt-note-to-stop match rule. Fixed: D7 matches in order by producer and state.
- scenarios, MEDIUM: landing before archive tag maps as hand. Fixed: D10 joins exactly through the archived task commit's `Warp-SHA` trailer, else by time between the landing checkpoint commit and the board done commit (review rounds 1 and 2).
- scenarios, MEDIUM: empty denominators. Fixed: D5 reports no data.
- scenarios and board-fit, MEDIUM: ordered pauses counted as wakes. Fixed: D6 reports `paused` outside wakes.
- scenarios, LOW: blame cost unbounded by default. Fixed: D4 defaults `--since` to twelve weeks.
- scenarios, LOW: invalid `--since` and `--format` uncovered. Fixed: D4 and acceptance criteria.
- scenarios, LOW: fallback and keep-alive placement. Fixed: D6.
- goal-scope, MEDIUM: Land's ready-tasks notice and the superseded notice missed. Fixed: D6, with the Land notice always absent, since no git object records it (review round 1).
- goal-scope, MEDIUM: hand landings redefined. Fixed: D10 `hand` for a mapped run with no Finalize done, `unmapped` for the rest (review round 2).
- goal-scope, MEDIUM: repairs and hand steps double-count with no actor. Fixed: D8 splits automatic, arbiter and LLM steps and says driver and orch are not separable.
- goal-scope, MEDIUM: failure classes undefined. Fixed: D7 defines them.
- goal-scope, LOW: departure from "from the drive reports" not stated. Fixed: D8 says so.
- goal-scope, LOW: mark the weeks named changes landed. Rejected: the landings table already dates them, YAGNI.
- pattern-fit, MEDIUM: halt-note reader in `loomcli` cannot be imported by an engine. Fixed: D7 moves the format to a leaf package.
- pattern-fit, MEDIUM: PATTERN-sole-parsers extension not named as a PATTERN edit; batten text parsed in `reportengine`. Fixed: Constraints and D6, D7.
- pattern-fit, LOW: fabric-git scope claim. Fixed: D3 cites the read-only exemption.
- pattern-fit, LOW: `_lyx/` exclusion spells the literal. Fixed: dropped from D11.
- board-fit, MEDIUM: Publish retry counted as wake and hand step. Fixed: D6 and D8.
- board-fit, MEDIUM: `run-robustness` dependency unnamed. Fixed: Constraints and D5 confine tolerance to the report's own parse functions and leave live readers to `run-robustness` (review round 1).
- board-fit, LOW: `orch-any-worktree` and `tokencount-cutoff-skew` not in risks. Fixed: Open risks.
- board-fit, LOW: `spawn-audit-log`'s summary verb could join `report`. Rejected: that entry is unbuilt, YAGNI.
- board-fit, LOW: `watchdog-bare-reason` fits D7. No change.
- leanness, MEDIUM: Constraints restated PATTERN. Fixed: only task-specific bullets kept.
- leanness, MEDIUM and LOW: duplicates (bound-packages, suppressed notices, scope vs constraints). Fixed.
- leanness, LOW: Auto-mode assumptions copy the decisions. Fixed: only picks with no decision of their own, as pointers.
- leanness, LOW: idle Scope Out bullet. Fixed: cut.
- leanness, LOW: PATTERN edit buried in D7. Fixed: named in Constraints.
- leanness, LOW: Notes repeating D7 and D6 evidence. Fixed: Notes trimmed to pointers.

## Rejected alternatives

- A `tools/` program beside `tools/tokencount`: no CLI tree, sandbox or test-budget cost, but it needs the source checkout and a Go toolchain and only runs in this repository's hub, and the board entry asks for a `lyx` verb.
- A durable notice log written by `orchengine.QueueNotice`: the most exact wake count, but it is new bookkeeping, which the board entry rules out, and it has no history for past runs.
- Counting wakes from the hub's notice log (`shedrun.NoticeLog`): untracked and machine-local, and it logs held-back and dropped notices too.
- Counting wakes from the orch's Claude transcripts: provider-specific (`PATTERN-shuttle-provider-seam`), machine-local, and mixes operator turns with notices.
- Parsing drive-report bodies for stop class and repairs: LLM prose with no format contract; a parser would pin a format no one maintains.
- Including in-flight runs: their counts change as they run, which makes week totals move between invocations.
- A fix is fresh when any modified line is recent: one touched line in a hot file would mark nearly every fix fresh.
- Landing kind from the commit subject: no convention exists; labels are the one assigned kind.
- A hub-wide `report.yaml` for the window and the range: flags suffice for an on-demand report.
- Bounding a run's batten record at the slug's next archive tag: the next run's batten commits precede its tag (lens review).
- Counting every archive tag as a run: bench pairs and other recipes would distort per-run figures.
- Classing stops with no halt note as `unclassified`: planned awaitings would flood the table.
- Splitting LLM steps between the loom driver and the orch: the records carry no actor.
- A halt-note reader in `loomcli`: an engine may not import a cli package.
- Joining a landing to batten's Land `done` commit: Land only pulls, deploys and prunes after Finalize's own merge, so another landing can fall between them (review round 2).
- Making `shedrun.ParseSeed` tolerant for the report: a live reader calls it, so every live seed read would loosen (review round 2).
- Excluding direct pushes from a run's time-join window: no record tells a direct push from a landing, so the residual case is an open risk instead (review round 3).

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): `lyx` verb or `tools/` program? `lyx` verb, per the board entry and so it runs on any hub (D1).
- Q2 (resolved, auto-pick): module and verb names? `lyx report convergence`, packages `reportcli` and `reportengine` (D1).
- Q3 (resolved, auto-pick): which runs count? Finished runs, one per archive tag (D4).
- Q4 (resolved, auto-pick): how to count orch wakes with no notice record? From batten halt entries and parent-review deliveries, split so one escalation counts once (D6).
- Q5 (resolved, auto-pick): stop class source? Go-authored halt notes; drive reports only counted (D7).
- Q6 (resolved, auto-pick): what about records missing from older archives? Absent, never zero (D5).
- Q7 (resolved, auto-pick): landing kind? Board labels from board history, `bug` meaning fix; `hand` is a run mapped by the exact join with no Finalize `done`, a time-joined run is never `hand`, a run whose batten Land never reached `done` is flagged `land-incomplete`, and `unmapped` is a commit that maps to no run (D10).
- Q8 (resolved, auto-pick): fresh-code definition? Majority of the fix's modified pre-image lines from landings within 3 days (D11).
- Q9 (resolved, auto-pick): output? JSON envelope plus `--format markdown` (D12).
- Q10 (resolved, auto-pick): default range? Twelve weeks, to bound blame (D4).
- Q11 (resolved, auto-pick): failure class? Producer plus the normalised leading clause of the halt reason (D7).
- Q12 (resolved, from the code): is there a durable record of the Land row's ready-tasks notice? No: only scratch markers cleared at a fresh seed and the untracked notice log, so that kind is always absent (D6).
