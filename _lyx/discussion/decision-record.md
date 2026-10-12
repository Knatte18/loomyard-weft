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
- Token and cost counting, which `tools/tokencount` owns.
- Parsing LLM-authored prose, such as drive-report bodies or review texts.

## Decisions

### D1 — A `lyx` verb in a new `report` module

Decision: `lyx report convergence` in new packages `internal/reportcli` and `internal/reportengine`, mounted under the root in `cmd/lyx/main.go`, audience operator.
The `report` group is a bare group that prints help and carries no audience.
Rationale: the board entry asks for a `lyx` verb, and the orch and the operator can run it on any hub without a source checkout or a Go toolchain, which a `tools/` program needs.

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
Its batten record is `_lyx/shed/<slug>/status.json` on the prime's records branch, at the newest commit before the slug's next fresh seed: the first later commit whose history restarts at Worktree-Create.
Flags: `--since <YYYY-MM-DD>` keeps runs and landings on or after that date, default twelve weeks before the invocation; `--fresh-days <N>` sets the fresh-code window, default 3, at least 1; `--format json|markdown`, default json.
A malformed date, a window below 1 or an unknown format is refused before any read.
Rationale: a tag is the one record every finished run leaves, and the next run's batten commits precede its own tag, so the next tag is the wrong bound; the default range bounds D11's blame cost.

### D5 — An absent record is unknown, never zero

Decision: each count is computed per source, and a run whose tree lacks a source (an archive older than the record, such as batten history before batten landed runs, or alignment attempts before `alignment-agent`) reports that source as absent, not 0.
Weekly aggregates sum over runs that have the source and state, per source, how many runs in the week lack it; a week with no run or no fix carrying a source reports that figure as no data, never 0.
A record that exists but does not parse is a finding naming the run and the file; the run's other sources still count, and the verb exits 0.
Every reader this task adds or calls accepts fields it does not know.
Rationale: older archives predate most of these records, and counting them as zero would fake a downward trend.

### D6 — Orch wakes, by kind

Decision: an orch wake is a notice or prompt the records show reached the orch.
Batten's state-change entries ("child X → Y at P") record every observed child state change whether or not batten noticed it, so they are classified by a reader in `internal/battenshed`, beside the writer of that text, which applies batten's notice rules to recorded state:

- A change into `awaiting`, `blocked` or `failed` is a wake, of kind `review-escalation` at a Bouncer whose round holds an escalation file, `pr-gate` at PR-Gate, else the state and producer.
- A change batten handles itself is not a wake and is reported as an automatic step: the first verify failure at Publish or Merge-In followed by batten's own goto and resume, and an awaiting at PR-Gate resumed because the parent moved.
- A change into `paused` is not a wake: a pause is ordered (`lyx-halt`) and is reported as kind `paused`.
- A failure episode of Land or Worktree-Teardown is a wake.

Other wakes, each from its own record:

- Parent review: a round whose `alignment.json` is not in state `applied` and whose `delivery.json` records a delivery is one wake, fallbacks included; keep-alive pings are counted beside it, not as wakes.
- The alignment approve that names superseded entries, from its applied verdict.
- The Land row's ready-tasks notice, when a durable record of the tasks it named exists; otherwise that kind is absent per D5.

Alignment outcomes are reported per point and verdict beside the wakes, so wakes the agent absorbed show.
Rationale: the notice queue is removed on delivery and trace logs are machine-local, so the durable records behind each notice are the only lasting source, and classifying in `battenshed` keeps one owner of both the text and the notice rules.

### D7 — Stop classes, failure classes and the halt-note reader

Decision: a `blocked` or `failed` stop's class is the `anomaly` of its Go-authored halt note, as `loomengine.ClassifyHalt` assigned it, with the note's state and producer.
Its failure class is the producer plus the halt reason's leading clause, up to the first `:`, `;` or `(`, with quoted strings, paths and numbers replaced by placeholders, so `status-missing`, `watchdog` and `publish_verify failed after merging parent branch` each form one class.
An `awaiting` stop is a planned hand-off, classed `awaiting` with its producer.
Halt notes are matched to the run's batten state changes in order by producer and state, so one stop counts once; a batten `blocked` or `failed` change with no matching note is class `no-note`.
The halt-note format moves into a new leaf package that `internal/loomcli` writes through and `reportengine` reads through, and `pattern/PATTERN-sole-parsers.md` gains a halt-note section naming it as the sole parser.
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
For each segment with a `_lyx/reviews/<segment>/` directory (discussion, plan, webster, merge, darn and any other present), rounds are the numbered rounds with a review file, and the outcome is the last round's: `accepted`, `escalated` (an escalation file) or `circling-decided` (a circling-decision file).
Parent-review rounds count the same way under `parent-review`, with the outcome from the round's verdict record.
A file is read through its format's existing reader where one exists, else through a reader added to the package that writes it.
Rationale: these are the review loop's own records, so a change meant to cut rounds (twill for Plan-Write, `webster-pack-read-set`) shows as a per-segment mean per week.

### D10 — Landings by kind

Decision: landings are the first-parent commits on the prime's parent branch in range.
A landing maps to its run through a record written at landing time, the prime records branch's `loom: run records checkpoint for landing <slug>` commit, so a landing whose pair is not yet torn down still maps.
A mapped run whose batten history has no Land `done` was landed by hand, kind `hand`; a commit that maps to no run is kind `unmapped`.
Otherwise the kind comes from the run's board entry labels as last recorded in board history before the landing: `fix` when they include `bug`, else `feature` for `feature` or `enhancement`, else `other`.
Rationale: board labels are the one kind a person assigned, board history keeps an entry after it is pruned, and the landing checkpoint exists before the archive tag does.

### D11 — The fresh-code fix share

Decision: for each `fix` landing, the pre-image lines it modifies or deletes are blamed at the landing's first parent, excluding Go test files, `docs/` and markdown.
The fix is `fresh` when more than half of those lines come from first-parent commits landed within `--fresh-days` before it, `aged` otherwise, and `no-pre-image` when it modifies or deletes no counted line.
The weekly share is fresh over fresh plus aged.
Blame runs only for fix landings in range.
Rationale: a majority threshold separates a fix repairing new code from one that touches a hot file in passing.

### D12 — Output

Decision: the JSON envelope carries the selection (refs, `--since`, `--fresh-days`), per-run rows, per-week aggregates, per-source absent counts and findings.
`--format markdown` renders the same data as one section per board question: wakes per run by week and kind; stop and failure classes by week with the number of runs each recurred in; landings by kind and the fresh-code share by week; review rounds per segment by week.
Rationale: JSON is the surface an agent consumes; the operator reads trends, which tables show.

## Constraints

- `pattern/PATTERN-told-geometry.md` and `pattern/PATTERN-sole-parsers.md` change in the same commit as the code (D2, D7); a new leaf package is listed in `pattern/PATTERN-leaf-packages.md`.
- No production file in `report*` spells `_lyx` or `.lyx`; record paths come from `shedrun`, `lyxdirs` and the record packages.
- The CLI test runs over a `hubforge` hub; new packages get rows in `cmd/lyx/testdata/test-budget.yaml`.
- `run-robustness` makes run-record readers tolerate unknown fields; any reader this task calls before that lands is made tolerant here (D5).

## Auto-mode assumptions

- A `lyx` verb, not a `tools/` program (D1), named `lyx report convergence`.
- ISO weeks in UTC, and a default range of twelve weeks (D4).
- `bug` among the labels means fix (D10).
- A three-day fresh window with a majority threshold (D11).

## Open risks

- Batten reads the child status on a change of mtime or a probe, so a halt resumed between two reads leaves no state-change entry and is not counted; repeated identical entries folded into `last_at` may undercount too.
- Board labels are assigned unevenly, so a bug fix labeled `enhancement` counts as a feature, and a squash landing bundling fix and feature is classed by labels and blamed whole.
- Archive tag commit dates can skew against other timestamps (`tokencount-cutoff-skew`), which can move a run across a week boundary.
- Archives older than a record report that source as absent, so early weeks carry few comparable runs.
- `orch-split`, `go-driven-shed` and `orch-any-worktree` move where notices start and where the orch runs; the prime-only rule and the readers follow today's records and may need changing when those land.

## Acceptance criteria

- A full loom run fixture holding every source (batten state changes, halt notes, arbiter attempts, drive reports, alignment attempts, parent-review rounds, review rounds per segment, parent decisions) yields the expected count for each.
- An archive lacking a source reports it absent, the week states how many runs lacked it, and a week with no data for a figure reports no data rather than 0.
- A malformed record yields a finding naming run and file, the run's other counts stand, and the exit code is 0; a record with an unknown field reads.
- A slug with two archive tags yields two runs, each with its own batten history, when the second run's batten commits precede its tag.
- A tag of a run parked after Discussion-Write counts as `partial` and enters no per-run figure.
- One Bouncer escalation that halts the child counts as one `review-escalation` wake.
- A first Publish verify failure that batten retried, an awaiting resumed because the parent moved and a `paused` change are not wakes.
- A parent-review round settled by an applied alignment verdict is not a wake; a fallback that delivered to the orch is one wake.
- An `awaiting` stop at PR-Gate is classed `awaiting`, not counted as a failure, and a `failed` stop and its halt note count once.
- Two halts whose reasons differ only in a path or a number share a failure class.
- A resume backed by an applied arbiter verdict is an arbiter repair, not an LLM step.
- A fix whose modified lines mostly came from a landing two days earlier is `fresh`, ten days earlier `aged`, and a pure-addition fix `no-pre-image`; test and doc lines do not count.
- A mapped run with no Land `done` is a `hand` landing, a landing whose pair is not yet torn down maps through its checkpoint, and a commit mapping to no run is `unmapped`.
- Runs and landings fall into ISO weeks in UTC, including a run at a Sunday-to-Monday boundary.
- A malformed `--since`, `--fresh-days 0` and an unknown `--format` are refused; no archive tags yields an empty report and exit 0.
- Invoked outside the prime, the verb refuses and names the prime.
- `--format markdown` renders one section per board question from the same data as the JSON.
- One test runs `lyx report convergence` through the real CLI wiring on a `hubforge` hub with an archive tag, a batten record on the prime's records branch and a landing on the parent, so the wiring is pinned without fake dependencies.

## Notes for the plan writer

- `tools/tokencount/shed.go` reads archive tags and decides full loom runs; it is prior art, not a dependency.
- The halt-note writer is `internal/loomcli/halt.go`; the state-change text is `stateChangeText` in `internal/battenshed/innerrun.go`.
- Alignment attempts are under `shedrun.AlignmentDir`, verdicts through `alignment.ReadVerdict`; parent-review round state is in `alignment.json` and `delivery.json`.
- `websterengine/gitwrap.go` runs a first-parent walk through `gitexec` because `gitrepo` has none; it is prior art for D3's new read and is not changed by this task.
