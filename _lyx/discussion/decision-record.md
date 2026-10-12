# Decision record: convergence-plan

## Goal

A read-only `lyx` verb reports, per finished run and per ISO week, what woke the orch, what stopped and repaired runs, which steps an LLM took by hand, how many review rounds each segment took, and, from the parent branch's git history, landings by kind and the share of fixes that repair freshly landed code.
Every count is derived from records lyx already writes; no orch, driver or agent keeps new bookkeeping.
The report answers the board entry's four questions: do orch wakes per run fall, do the same stop and failure classes recur, does the fresh-code fix share fall, and do review rounds per segment fall after a change meant to cut them.

## Scope

In:

- A new CLI module, `lyx report`, with one verb, `lyx report convergence`, and its engine package.
- Readers for the records each count needs, each in the package that owns the format (D7, D8, D9), and the `gitrepo` reads D10 and D11 need.
- Per-run rows and per-week aggregates, as the JSON envelope and as a markdown rendering.
- The landing-kind and fresh-code-fix computation over the parent branch's first-parent history.
- The module's `doc.go`, the module table in `docs/overview.md`, the CLI tree, sandbox coverage and the test-budget rows the new packages need.

Out:

- Any new record, log line or field written by the orch, a driver, batten, shed or any agent.
- Runs still in flight: a run counts once its pair is torn down and its `archive/<slug>/<sha>` tag exists.
  A landed pair not yet torn down still maps its landing under D10, but enters no per-run figure.
- Token and cost counting, which `tools/tokencount` owns.
- Parsing LLM-authored prose, such as drive-report bodies or review texts.

## Decisions

### D1 — A `lyx` verb in a new `report` module

Decision: `lyx report convergence` in new packages `internal/reportcli` and `internal/reportengine`, mounted under the root in `cmd/lyx/main.go`, audience operator.
The `report` group is a bare group that prints help and carries no audience.
Rationale: the board entry asks for a `lyx` verb, and the orch and the operator can run it on any hub without a source checkout or a Go toolchain.

### D2 — Where it runs and what it is told

Decision: the verb runs from the hub's prime worktree only, as `lyx orch` and `lyx batten` do, and refuses elsewhere naming the prime as the way forward.
`reportcli` resolves the geometry and tells `reportengine` every absolute path it reads: the prime's code repository, the hub's records store holding the archive tags and the prime's records branch, and the board worktree for board history.
`reportengine` never imports `internal/lyxcwd`, and `pattern/PATTERN-told-geometry.md`'s bound-packages list gains it.
Rationale: every source is reachable from the prime, and one fixed home keeps the numbers comparable across invocations.

### D3 — Read-only, from git objects

Decision: every record is read from git objects at refs resolved once at the start: archive tags, the prime's records branch, the board's branch and the parent branch.
The verb writes nothing, takes no lock and makes no commit; its git use is read-only, which `PATTERN-fabric-git`'s read-only exemption covers.
Git reads go through `internal/gitrepo` (`PATTERN-gitrepo-client-boundary`); `gitrepo` gains a first-parent range walk and a blame read, which it lacks today.
Rationale: archived runs exist only as git objects, and one ref snapshot keeps the report consistent while runs land concurrently.

### D4 — The unit: one full loom run per archive tag

Decision: each `archive/<slug>/<sha>` tag is one run, so a reused slug yields one run per tag.
A run counts when its archived seed and status name the loom recipe and its history holds a Finalize entry, the rule `tools/tokencount` applies; other tags (bench pairs parked after Discussion-Write, other recipes) are counted per week as `partial` and enter no per-run figure.
A run's time is its tag's commit date, and its week is the ISO week (Monday start) of that date in UTC.
The run's loom records are the tree at its tag.
Its batten record is `_lyx/shed/<slug>/status.json` on the prime's records branch, at the newest commit that still holds that file, within the span from the run's own fresh seed up to the slug's next fresh seed (the first later commit whose history restarts at Worktree-Create), or up to the resolved branch tip when no later fresh seed exists.
The run's own fresh seed is the newest fresh seed for the slug that is after the slug's previous archive tag's commit and at or before this tag's commit.
When no fresh seed falls in that interval, or no commit in the span holds the file, the batten source is absent per D5.
Flags: `--since <YYYY-MM-DD>` keeps runs and landings on or after that date, default twelve weeks before the invocation; `--fresh-days <N>` sets the fresh-code window, default 3, and any value of 1 or more is accepted; `--format json|markdown`, default json.
A malformed date, a window below 1 or an unknown format is refused before any read.
Blame runs once per blamed landing in the selected range (D11), so an explicit early `--since` widens the cost linearly, which the operator accepts by passing it.
Rationale: a tag is the one record every finished run leaves; the default range bounds D11's blame cost.

### D5 — An absent record is unknown, never zero

Decision: each count is computed per source, and a run whose tree lacks a source (an archive older than the record, such as batten history before batten landed runs, or alignment attempts before `alignment-agent`) reports that source as absent, not 0.
Weekly aggregates sum over runs that have the source and state, per source, how many runs in the week lack it; a week with no run or no fix carrying a source reports that figure as no data, never 0.
A composite figure follows the same rule: a run's wake total is reported per source, and the total enters a week's mean only for runs holding every wake source, with the remainder counted as lacking.
A record that exists but does not parse is a finding naming the run and the file; the run's other sources still count, and the verb exits 0.
Tolerance of unknown fields is confined to the report's own read path: each owning package gains a tolerant parse-from-bytes function for the report, separate from any existing strict one.
Where a strict parse-from-bytes function already exists and a live reader calls it, as `shedrun.ParseSeed` does for `shedrun.ReadSeed`, that function and its live callers stay unchanged and the report never calls it.
The bound: no live reader's acceptance changes, and loosening an existing shared parser never meets it.
Rationale: older archives predate most of these records, and counting them as zero would fake a downward trend.

### D6 — Orch wakes, by kind

Decision: an orch wake is a notice or prompt the records show reached the orch.
Batten's state-change entries ("child X → Y at P") record every observed child state change whether or not batten noticed it, so they are classified by a reader in `internal/battenshed`, beside the writer of that text, which parses bytes under D9's rule and applies batten's notice rules to recorded state:

- A change into `awaiting`, `blocked` or `failed` is a wake, of kind `review-escalation` at a Bouncer whose round holds an escalation file, `pr-gate` at PR-Gate, else the state and producer.
- A change batten handles itself is not a wake and is reported as an automatic step: the first verify failure at Publish or Merge-In followed by batten's own goto and resume, and an awaiting at PR-Gate resumed because the parent moved.
- A change into `paused` is not a wake: a pause is ordered (`lyx-halt`) and is reported as kind `paused`.
- A failure episode of Land or Worktree-Teardown is a wake.
  A failure episode is the row's batten history entries from its first `failed` state through its next `done`, or through the end of the history when none follows, and counts once however many `failed` entries it holds.

Other wakes, each from its own record:

- Parent review: a round whose `alignment.json` is not in state `applied` and whose `delivery.json` records a delivery is one wake, fallbacks included; keep-alive pings are counted beside it, not as wakes.
- The alignment approve that names superseded entries, from its applied verdict.
- The Land row's ready-tasks notice leaves only ephemeral records: the row's ready-snapshot and row-notice markers in the `.lyx` scratch dir, cleared at a fresh seed, and a line in the hub's untracked notice log.
  No git object holds it, so under D3 that kind is always absent.

Alignment outcomes are reported per point and verdict beside the wakes, so wakes the agent absorbed show.
Rationale: the notice queue is removed on delivery and trace logs are machine-local, so the durable records behind each notice are the only lasting source, and classifying in `battenshed` keeps one owner of both the text and the notice rules.

### D7 — Stop classes, failure classes and the halt-note reader

Decision: a `blocked` or `failed` stop's class is the `anomaly` of its Go-authored halt note, as `loomengine.ClassifyHalt` assigned it, with the note's state and producer.
Its failure class is the producer plus the halt reason's leading clause, up to the first `:`, `;` or `(`, with quoted strings, paths and numbers replaced by placeholders, so `status-missing`, `watchdog` and `publish_verify failed after merging parent branch` each form one class.
An `awaiting` stop is a planned hand-off, classed `awaiting` with its producer.
Halt notes are read from the run's loom friction dir and its timestamped reflection-archive siblings in the tag's tree.
The halt-note source is present when the tag's tree holds the friction dir or any of its timestamped archive siblings; a run whose tree holds neither reports stop and failure classes as absent per D5.
No committed record shows Tier 2 was on, so a Tier 2 run with no note also reads as absent, not as zero stops (Open risks).
Halt notes are matched to the run's batten state changes in order by producer and state, so one stop counts once; where the halt-note source is present, a batten `blocked` or `failed` change with no matching note is class `no-note`.
A halt note left over after the in-order match counts as a stop of its own class with no batten state change, which covers a halt resumed between two batten reads (Open risks).
The halt-note format moves into a new leaf package that `internal/loomcli` writes through and `reportengine` parses bytes through under D9's rule, and `pattern/PATTERN-sole-parsers.md` gains a halt-note section naming it as the sole parser.
Rationale: halt notes carry the class in Go-written fields, `reportengine` may not import a cli package, and a stop-class table flooded by planned awaitings would hide recurrence.

### D8 — Repairs and LLM steps

Decision: repairs and hand steps are split by actor as far as the records allow:

- Automatic steps: batten's own retries and resumes (D6).
- Arbiter repairs: arbiter attempts by verdict, from `attempt.json` and `verdict.yaml`, with whether the verdict was applied.
- LLM steps: goto and reroute entries in the loom and batten histories and resumes after a halt that no automatic step or applied arbiter verdict accounts for, plus hand landings (D10).
  The records do not tell the loom driver from the orch, so this is one count.

Drive reports are counted per run, one per driver attempt, and their bodies are not parsed; this departs from the board entry's "from the drive reports", since those bodies are LLM prose.
Rationale: every count comes from Go-written records, and an actor split the records cannot support would be a guess.

### D9 — Parent decisions and review rounds

Decision: parent decisions are the run's `### Added after Discussion (<by>, <date>)` entries, counted by `by`; `internal/discussionparser`, which writes them, gains their reader.
For each segment with a `_lyx/reviews/<segment>/` directory (discussion, plan, webster, merge, darn and any other present), rounds are the numbered rounds with a review file; a segment with no directory is absent for that run, not zero rounds.
The outcome is the last round's: `circling-decided` when it holds a circling-decision file, whether or not an escalation file is also present, `escalated` when it holds only an escalation file, and `accepted` otherwise.
Parent-review rounds count the same way under `parent-review`, with the outcome from the round's verdict record.
Every record is parsed from bytes read out of a git object, through the tolerant parse-from-bytes function D5 places in the package that owns the format; no path-based, locking or existing strict reader is called.
Rationale: these are the review loop's own records, so a change meant to cut rounds (twill for Plan-Write, `webster-pack-read-set`) shows as a per-segment mean per week.

### D10 — Landings by kind

Decision: landings are the first-parent commits on the prime's parent branch in range.
The child's Finalize performs the parent-side merge; batten's Land row only pulls the prime, deploys and prunes the board, so a batten Land `done` commit is not the landing and no join uses it.
A landing maps to its run through two sources, the first applied to every run before the second.
First, an exact join for a run with an archive tag: its landed task commit is the `Warp-SHA` trailer of the newest commit in the tag's history that carries one, since every fabric weft commit, the pending-records commit before the archive tag included, names the task worktree's HEAD there.
The landing is the first-parent commit that is that commit (a fast-forward), has it as a parent (a merge), or has its tree (a squash).
When the landing is a fast-forward, the first-parent commits after the previous landing and before the run's task commit are the task branch's own commits: they belong to that landing and are neither counted as landings nor classed `unmapped`.
Second, a time join bounded by two records, for a run the first source left unmapped or whose pair is not yet torn down.
For a tagged run, the window opens at the newest `loom: run records checkpoint for landing <slug>` commit on the prime's records branch dated after the slug's previous archive tag's commit and at or before this tag's commit, the same interval D4 applies to fresh seeds; for an untagged run, it opens at the newest such checkpoint for the slug dated after the slug's newest archive tag's commit.
Finalize writes the checkpoint before its merge, and its retry path can write a second one, so the newest checkpoint in the interval is the one just before the merge.
The window closes at the first board commit after that checkpoint that marks the slug's entry done, which Finalize writes right after the merge.
The landing is the first first-parent commit dated in that window; every window comparison reads committer dates.
A run with no checkpoint in its interval, or whose board entry was not marked done after it, gets no time join.
A first-parent commit already claimed by another run or source is never claimed again, and two runs claiming one commit in the same source both become unmapped and are reported as a finding.
A landing that matches neither source maps to no run.
A run mapped by the exact join whose loom history at its tag holds no Finalize `done` entry was landed by hand, kind `hand`.
A run mapped by the time join is never `hand`, since the checkpoint it joined through is written by Finalize itself.
An untagged run mapped by the time join is identified by its slug and contributes to the landing table only, never to per-run rows, and its `land-incomplete` flag is absent, since D4 defines no batten span without a tag.
A commit that maps to no run, such as a direct push to the parent branch, is kind `unmapped`.
Otherwise the kind comes from the run's board entry labels as last recorded in board history before the landing: `fix` when they include `bug`, else `feature` for `feature` or `enhancement`, else `other`.
A tagged run whose Finalize is done but whose batten history (D4) never reaches Land `done` keeps its label kind and carries a `land-incomplete` flag; with the batten source absent, the flag is absent.
Rationale: board labels are the one kind a person assigned, board history keeps an entry after it is pruned, every archived run's weft history names its task commit, and the time join's bounds are both written by Finalize around its own merge.

### D11 — The fresh-code fix share

Decision: for each `fix` landing, the pre-image lines it modifies or deletes are blamed at the landing's first parent, excluding Go test files, `docs/` and markdown.
The fix is `fresh` when more than half of those lines come from first-parent commits landed within `--fresh-days` before it, `aged` otherwise, and `no-pre-image` when it modifies or deletes no counted line.
The weekly share is fresh over fresh plus aged; a week with no fresh or aged fix reports the share as no data.
`unmapped` commits, which include the orch's direct-push repairs, are blamed the same way and reported as their own `unmapped` share beside the fix share, never mixed into it, since the records carry no label telling a direct-push repair from other direct pushes.
A fast-forward landing's own task-branch commits (D10) are neither landings nor `unmapped`, so they are never blamed.
Blame runs only for fix and `unmapped` landings in range.
Rationale: a majority threshold separates a fix repairing new code from one that touches a hot file in passing.

### D12 — Output

Decision: the JSON envelope carries the selection (refs, `--since`, `--fresh-days`), per-run rows, per-week aggregates, per-source absent counts and findings.
`--format markdown` renders the same data as one section per board question: wakes per run by week and kind; stop and failure classes by week with the number of runs each recurred in; landings by kind and the fix and `unmapped` fresh-code shares by week; review rounds per segment by week.
Rationale: JSON is the surface an agent consumes; the operator reads trends, which tables show.

### Added after Discussion (parent, 2026-10-12): Seed parsing follows run-robustness D4

Decision: run-robustness lands first. Its D4 makes shedrun.ParseSeed and ReadSeed tolerant of unknown keys. After Merge-In, convergence-plan drops ParseSeedTolerant and its 'ReadSeed still rejects it' criterion, uses the tolerant ParseSeed, and keeps internal/alignment/records.go compiling where run-robustness removed the encoding/json import.

Rationale: Two parsers for one record would contradict PATTERN-tolerant-records, which run-robustness adds; the alignment agent at run-robustness's PR-Gate found the conflict.

## Constraints

- `pattern/PATTERN-told-geometry.md` and `pattern/PATTERN-sole-parsers.md` change in the same commit as the code (D2, D7); a new leaf package is listed in `pattern/PATTERN-leaf-packages.md`.
- No production file in `report*` spells `_lyx` or `.lyx`; record paths come from `shedrun`, `lyxdirs` and the record packages.
- The CLI test runs over a `hubforge` hub; new packages get rows in `cmd/lyx/testdata/test-budget.yaml`.
- The report's parse functions tolerate unknown fields, and no reader a live run or gate calls changes (D5); reader tolerance for live run records stays in `run-robustness`'s own scope.

## Auto-mode assumptions

- A `lyx` verb, not a `tools/` program (D1), named `lyx report convergence`.
- ISO weeks in UTC, and a default range of twelve weeks (D4).
- `bug` among the labels means fix (D10).
- A three-day fresh window with a majority threshold (D11).

## Open risks

- Batten reads the child status on a change of mtime or a probe, so a halt resumed between two reads leaves no state-change entry and counts only through its leftover halt note (D7), or not at all when the run has none; repeated identical entries folded into `last_at` may undercount too.
- Board labels are assigned unevenly, so a bug fix labeled `enhancement` counts as a feature, and a squash landing bundling fix and feature is classed by labels and blamed whole.
- Archive tag commit dates can skew against other timestamps (`tokencount-cutoff-skew`), which can move a run across a week boundary.
- Archives older than a record report that source as absent, so early weeks carry few comparable runs.
- A hand landing or direct push dated inside a run's checkpoint-to-board-done window that no exact join claimed is taken by the time join, and the run's real landing then reads `unmapped` (D10); the parent lock keeps out only other Finalize landings, and no record tells a direct push from a landing.
- Git never tracks an empty friction dir and no committed record shows Tier 2 was on, so a clean Tier 2 run reports stop classes absent (D7); weekly stop-class figures cover only runs that left a note, which makes recurrence look denser than it is.
- `orch-split`, `go-driven-shed` and `orch-any-worktree` move where notices start and where the orch runs; the prime-only rule and the readers follow today's records and may need changing when those land.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - fast-forward-landings-flood-unmapped (class design, severity MEDIUM, open in rounds 3)
  - landing-run-mapping-conditional-checkpoint (class design, severity BLOCKING, open in rounds 1, 2, 3)
  - missing-acceptance-criteria (class design, severity MEDIUM, open in rounds 1, 2, 3)
  - time-join-reused-slug-window (class decision, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A full loom run fixture holding every source (batten state changes, halt notes, arbiter attempts, drive reports, alignment attempts, parent-review rounds, review rounds per segment, parent decisions) yields the expected count for each; it checks counts only, and each classification rule has its own criterion below.
- An archive lacking a source reports it absent, the week states how many runs lacked it, and a week with no data for a figure reports no data rather than 0.
- A malformed record yields a finding naming run and file, the run's other counts stand, and the exit code is 0; a record with an unknown field reads.
- A seed with an unknown field reads through the report's tolerant parse, and `shedrun.ReadSeed` still rejects it.
- A slug with two archive tags yields two runs, each with its own batten history, when the second run's batten commits precede its tag.
- A slug whose first tag came from a batten run and whose second tag came from a run batten never seeded reports the batten source absent for the second tag, and the first tag's batten history counts once.
- A slug whose run directory was removed by `batten cleanup` still reads its batten record from the last commit holding `status.json`.
- A tag of a run parked after Discussion-Write and a tag whose seed names another recipe each count as `partial` and enter no per-run figure.
- One Bouncer escalation that halts the child counts as one `review-escalation` wake.
- An awaiting at PR-Gate that batten does not resume itself is one `pr-gate` wake, and the Land ready-tasks kind is reported absent for every run.
- A first Publish verify failure that batten retried, an awaiting resumed because the parent moved and a `paused` change are not wakes.
- A parent-review round settled by an applied alignment verdict is not a wake; a fallback that delivered to the orch is one wake.
- An `awaiting` stop at PR-Gate is classed `awaiting`, not counted as a failure, and a `failed` stop and its halt note count once.
- A `blocked` stop's class is its halt note's `anomaly` with the note's state and producer.
- Two halts whose reasons differ only in a path, a number or a quoted string share a failure class, and two whose reasons share the text before the first `:`, `;` or `(` but differ after it share one too.
- A resume backed by an applied arbiter verdict is an arbiter repair, not an LLM step, and an arbiter attempt whose verdict was not applied is reported under its verdict as not applied.
- A fix whose modified lines mostly came from a landing two days earlier is `fresh`, ten days earlier `aged`, exactly half from a recent landing `aged`, and a pure-addition fix `no-pre-image`; test and doc lines do not count.
- A run mapped by the exact join whose loom history holds no Finalize `done` is a `hand` landing, a landing whose pair is not yet torn down maps through its checkpoint and board done commit, keeps its label kind, is never `hand`, has no `land-incomplete` flag and enters no per-run row, and a commit mapping to no run is `unmapped`.
- A slug with two runs, each with its own checkpoint and board done commit, maps each landing to its own run, and a run with two checkpoints in its interval opens its window at the newer one.
- A run with no checkpoint in its interval, or whose board entry was never marked done after its checkpoint, gets no time join.
- A non-squash fast-forward landing of a task branch with several commits yields one landing and no `unmapped` commit, and none of its task-branch commits is blamed.
- A run whose Finalize landed but whose batten Land never reached `done` keeps its label kind and is flagged `land-incomplete`, not `hand`.
- A loom landing whose checkpoint commit was a no-op still maps to its run through its task commit's `Warp-SHA` trailer, by commit, by parent and by tree for a squash.
- A first-parent commit inside a run's checkpoint-to-board-done window that another tagged child's exact join already claimed is not claimed by the time-joined run, and two runs claiming one commit both become unmapped with a finding.
- A run's labels map `bug` to `fix`, `feature` or `enhancement` to `feature` and anything else to `other`, read as last recorded before the landing, including for an entry pruned since.
- An `unmapped` commit is blamed into the `unmapped` share and never into the fix share, and a `feature` or `other` landing is never blamed.
- An applied alignment approve naming superseded entries is one wake, and a Land failure episode is one wake.
- A Worktree-Teardown failure episode with two `failed` entries before its next `done` is one wake, and a Land `failed` entry after that `done` opens a second episode and a second wake.
- A change into `blocked` at a producer other than a Bouncer or PR-Gate is one wake whose kind is that state plus that producer.
- A parent-review round's keep-alive pings are counted beside its wake and add no wake, and a round whose `delivery.json` records no delivery is not a wake.
- Alignment outcomes are reported per point and verdict, so an `approve`, a `reject` and an `escalate` verdict at different points each show under their own point and verdict.
- A first Merge-In verify failure that batten retried is counted as an automatic step, a second verify failure at the same row is a wake, and a `paused` change is reported as kind `paused`.
- A goto entry and a reroute entry each count as an LLM step, a resume after a halt with no automatic step or applied arbiter verdict is an LLM step, and a `hand` landing is an LLM step.
- Parent decisions are counted per `by`, so two `Added after Discussion` entries by one author and one by another yield two and one.
- A segment whose last round holds an escalation file is `escalated`, one holding a circling-decision file is `circling-decided` with or without an escalation file, one holding neither is `accepted`, and a run with no directory for a segment reports it absent; a numbered round with no review file is not a round.
- A parent-review round takes its outcome from its verdict record.
- A batten `failed` change with no halt note in a run holding halt notes is class `no-note`, a run with neither a friction dir nor an archive sibling reports stop classes absent, and a halt note left after matching counts as its own stop.
- A run whose only friction notes sit in an archive sibling reports its halt notes.
- A run lacking one wake source reports its wake total per source and is left out of the week's wake mean.
- Runs and landings dated before `--since` are left out.
- Runs and landings fall into ISO weeks in UTC, including a run at a Sunday-to-Monday boundary.
- A malformed `--since`, `--fresh-days 0` and an unknown `--format` are refused; no archive tags yields an empty report and exit 0.
- Invoked outside the prime, the verb refuses and names the prime.
- `--format markdown` renders one section per board question from the same data as the JSON.
- One test runs `lyx report convergence` through the real CLI wiring on a `hubforge` hub with an archive tag, a batten record on the prime's records branch and a landing on the parent, so the wiring is pinned without fake dependencies.

## Notes for the plan writer

- `tools/tokencount/shed.go` reads archive tags and decides full loom runs; it is prior art, not a dependency.
- The halt-note writer is `internal/loomcli/halt.go`; the state-change text is `stateChangeText` in `internal/battenshed/innerrun.go`.
- Alignment attempts sit at `shedrun.AlignmentDir`'s place in the tag tree; `alignment` owns the verdict format, and the report reads verdicts through the tolerant parse-from-bytes function D5 adds there; parent-review round state is in `alignment.json` and `delivery.json`.
- `websterengine/gitwrap.go` runs a first-parent walk through `gitexec` because `gitrepo` has none; it is prior art for D3's new read and is not changed by this task.
