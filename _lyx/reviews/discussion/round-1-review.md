---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:118
    summary: D10 maps landings through the landing checkpoint commit, but that commit is written only when the parent's run records hold an uncommitted change, so real loom landings such as plan-gate-fixes map to no run and become unmapped
    fix: In D10, replace the claim that the checkpoint is a record written at every landing with a mapping every loom landing leaves, and state its failure behavior. Keep the checkpoint commit as the first source, and add a fallback the records hold for every landing, for example the slug's batten status commit on the prime's records branch whose history first shows Land, joined to the newest first-parent parent-branch commit at or before it, bounded by that run's archive tag existing for the slug. State what a landing that matches neither is (unmapped) and that the checkpoint is conditional. Add an acceptance criterion that a loom landing whose checkpoint commit was a no-op still maps to its run. Correct the D10 Rationale and the support log's scenarios entry accordingly.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:64
    summary: D5 and the run-robustness Constraint make every reader the report calls accept unknown fields, which downgrades the strict decodes guarding live gates (alignment.ReadVerdict and loomengine.ReadArbiterVerdict use KnownFields, state reads use DisallowUnknownFields) with no stated bound
    fix: In D5, replace the sentence on tolerant readers with one that confines tolerance to the report's own read path. The report parses through tolerant parse functions added in each owning package beside the existing reader, and the existing readers that live runs and gates call keep their strictness unchanged. Name the bound in D5, that no live reader's acceptance changes. Rewrite the fourth Constraints bullet the same way, leaving reader tolerance for live run records to run-robustness's own scope.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:112
    summary: D9's rule to read each file through its format's existing reader conflicts with D3, since the existing readers take filesystem paths and some take a lock (state reads take the status lock, ReadVerdict reads an Attempt directory), while D3 reads only git objects and takes no lock
    fix: In D9, replace the existing-reader sentence with this rule. Every record is parsed from bytes read out of a git object, through a parse-from-bytes function in the package that owns the format, added where none exists. No path-based or locking reader is called. Point D7's halt-note reader and D6's battenshed classifier at the same rule.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:54
    summary: D4's batten-record bound picks the newest commit before the next fresh seed, which for a cleaned slug is lyx batten cleanup's removal commit with no status.json, and the case with no later seed is left implicit
    fix: "In D4, restate the bound as: the newest commit on the prime's records branch after the run's tag-time history began and before the slug's next fresh seed that still holds `status.json` for the slug, or before the resolved branch tip when no later fresh seed exists. When no commit in that span holds the file, the batten source is absent per D5. Add an acceptance criterion for a slug whose run directory was removed by batten cleanup."
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:81
    summary: D6 leaves the Land ready-tasks notice conditional on Q12 being answered, though the code answers it, and the hub notice log (shedrun.NoticeLog, every batten notice composed) is weighed nowhere
    fix: "In D6, replace the ready-tasks bullet with a decided rule. The Land row's ready notice leaves only ephemeral records: the row's ready-snapshot and row-notice markers in the `.lyx` scratch dir, which are cleared at a fresh seed, and a line in the hub's untracked notice log. No git object holds it, so under D3 that kind is always absent. Close Q12 in the support log, and add the hub notice log to Rejected alternatives with its reason: untracked, machine-local, and it logs held-back and dropped notices too. Correct Turn 1's no-durable-notice-record sentence to name it."
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:91
    summary: D7 does not say where halt notes are read, treats a run with no halt-note source as no-note stops instead of absent, and gives no rule for a halt note batten recorded no change for
    fix: In D7, add three sentences. Halt notes are read from the run's loom friction dir and its timestamped reflection-archive siblings in the tag's tree. A run whose tree holds no friction dir, because Tier 2 was off or the run predates halt notes, reports stop and failure classes as absent per D5, and `no-note` applies only when the source is present. A halt note left over after the in-order match counts as a stop of its own class with no batten state change, which covers the resumed-between-reads case from Open risks.
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:62
    summary: D5's absent-versus-zero rule is not carried through the composite figures. The wakes-per-run total spans sources with different presence, a segment with no reviews directory is neither absent nor zero, and a week whose fixes are all no-pre-image has no stated share
    fix: In D5, add the rule for composite figures. A run's wake total is reported per source, and the total enters a week's mean only for runs holding every wake source, with the remainder counted as lacking. In D9, a segment with no `_lyx/reviews/<segment>/` directory is absent for that run, not zero rounds. In D11, a week with no fresh or aged fix reports the share as no data.
  - id: F8
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:161
    summary: Several rules that carry a requirement have no checkable acceptance criterion and are covered only by the all-sources fixture, among them the superseding approve wake, goto and reroute as LLM steps, an unaccounted resume as an LLM step, D9's escalated and circling-decided outcomes, the Land and Worktree-Teardown failure wake, the no-note class and the --since filter
    fix: "Under Acceptance criteria, add one criterion per rule: an applied alignment approve naming superseded entries is one wake; a goto entry and a reroute entry each count as an LLM step; a resume after a halt with no automatic step or applied arbiter verdict is an LLM step; a segment whose last round holds an escalation file is `escalated` and one holding a circling-decision file is `circling-decided`; a Land failure episode is one wake; a batten `failed` change with no halt note in a run holding halt notes is class `no-note`; runs and landings dated before `--since` are left out."
  - id: F9
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:119
    summary: Commits pushed straight to the parent branch, the orch's own fixes among them, are unmapped and so enter neither the landing kinds nor the fresh-code share, which the board's third question and its audit count, and no Decision says that they are excluded or why
    fix: "In D10 or D11, decide what happens to unmapped commits and state it: either they are counted in the fresh-code computation under their own kind `unmapped`, blamed like fixes, or they are excluded from the share. If they are excluded, add an Open risks bullet saying the share leaves out direct-push repairs."
  - id: F10
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:110
    summary: D9 gives no precedence when the last round holds both an escalation file and a circling-decision file, which a circling escalation settled by a decision leaves
    fix: "In D9, state that when the last round holds a circling-decision file the outcome is `circling-decided` whether or not an escalation file is also present, `escalated` when it holds only an escalation file, and `accepted` otherwise."
  - id: F11
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:55
    summary: D4 bounds blame cost only by the default --since. An explicit early --since makes blame cost unbounded, and --fresh-days has no upper bound, but the Decision states neither what the flags let through nor that this is accepted
    fix: In D4, add one sentence stating that blame runs once per fix landing in the selected range, so an explicit --since widens the cost linearly and the operator accepts that. Either cap --fresh-days at a stated maximum or state that any value of 1 or more is accepted.
  - id: F12
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:87
    summary: The ledger's Q7 says unmapped commits are hand landings, while D10 splits them into `unmapped` and keeps `hand` for a mapped run with no Land done. The plan writer reads only the decision record, so the stale ledger misleads only the review gate
    fix: In support-log.md Q7, replace the unmapped-commits clause with D10's split, so `hand` is a mapped run with no Land `done` and `unmapped` is a commit that maps to no run.
  - id: F13
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:57
    summary: D4's Rationale argues against a rejected alternative, the next-tag bound, which support-log.md's Rejected alternatives already records, and D1's Rationale likewise argues against the tools/ program
    fix: In D4's Rationale, delete the clause about the next run's batten commits preceding its tag and the next tag being the wrong bound, and keep the sentence on tags and the default range. In D1's Rationale, delete the trailing clause about a tools/ program needing a source checkout and a Go toolchain.
---

### [BLOCKING] D10's landing-to-run mapping rests on a commit that is not written for every landing

**Location:** `_lyx/discussion/decision-record.md:118`

**Issue:** `parentRecordsCommitter` in `internal/loomcli/landingdeps.go` commits nothing when the parent's run-records directory holds no file.
`CommitRecordsPaths` in `internal/fabricengine/commitweftpaths.go` also reports a no-op as not committed when nothing changed.
Batten commits its own `status.json` on every transition through `internal/battencli/commitstatus.go`, so the prime's run records are often already clean when Finalize runs.
I checked the hub's records store: `main-weft` holds no `loom: run records checkpoint for landing plan-gate-fixes` commit.
That run's archive tag `archive/plan-gate-fixes/7495849cda6e` exists and its landing on `main` is `7b17ddfac` (2026-10-12 12:25).
The landing `314816cde` at 12:07 has no checkpoint either, while the landings at 13:27 and after do.
Under D10 these full loom landings are `unmapped`: they lose their board-label kind, D11 never blames them, and D8 cannot count them.
The Rationale's premise that "the landing checkpoint exists before the archive tag does" is false for exactly these cases.

### [BLOCKING] D5 removes strict decoding from live gate readers with no bound

**Location:** `_lyx/discussion/decision-record.md:64`

**Issue:** D5 says "Every reader this task adds or calls accepts fields it does not know", and the fourth Constraint makes "any reader this task calls" tolerant.
The readers D6, D8 and D9 call include `alignment.ReadVerdict`, which uses `KnownFields(true)` and validates the agent's approve that lands a task with no orch turn.
They also include `loomengine.ReadArbiterVerdict`, which uses `KnownFields(true)`, and the `state` JSON readers, which use `DisallowUnknownFields`.
Loosening these changes what a live run accepts from an agent, and that is a downgraded guard the rubric's attack-surface rule requires a bound for.
`run-robustness`'s board entry covers "the whole-file records batten, shed and loom read across processes", not agent verdict files, so D5 reaches past the dependency it cites.
A read-only report needs tolerance only on its own path.

### [MEDIUM] D9's existing-reader rule conflicts with D3's git-object, lock-free reads

**Location:** `_lyx/discussion/decision-record.md:112`

**Issue:** D3 reads every record from git objects and takes no lock.
The existing readers D9 points at read from the filesystem: `alignment.ReadVerdict` takes an `Attempt` and calls `os.ReadFile`, `readCirclingFile` reads a run-dir path, and the `state` readers acquire the status lock first.
None of them can read a blob at an archive tag.
The plan writer has to decide on byte-level parse entry points, and nothing in the record tells them to.

### [MEDIUM] D4's batten-record bound selects the cleanup commit

**Location:** `_lyx/discussion/decision-record.md:54`

**Issue:** `lyx batten cleanup` (and `lyx cleanup`) removes a done slug's records run directory and commits `batten cleanup: remove the run directory of <slug>` (`internal/battencli/cleanup.go:218`).
For any cleaned slug, the newest commit before the next fresh seed is that removal, whose tree holds no `status.json`, so the batten source reads as absent for runs that have it.
The rule also does not say what the bound is when the slug has no later fresh seed.
The intended answer is the branch tip, but the Decision only implies it.

### [MEDIUM] The Land ready-tasks notice is left open though the code settles it

**Location:** `_lyx/discussion/decision-record.md:81`

**Issue:** D6 says the kind counts "when a durable record of the tasks it named exists", and Q12 is open "for Plan".
Plan never reads the support log, so the plan writer cannot tell the question was ever asked.
`internal/battenshed/land.go` and `rownotice.go` record the notice only in `<row>-ready-snapshot` and `<row>-row-notice` markers under the `.lyx` scratch dir, which `ClearRunMarkers` removes at a fresh seed.
It also appears in the hub's untracked notice log, `shedrun.NoticeLog`, which `battenshed/doc.go` says receives every notice a batten process composes.
No git object holds the notice, so under D3 the kind is always absent, and the conditional describes no real case.
The support log's Turn 1 says "no durable notice record exists" without mentioning the notice log, and Rejected alternatives does not weigh it.

### [MEDIUM] D7's halt-note source location, absence and unmatched notes are undefined

**Location:** `_lyx/discussion/decision-record.md:91`

**Issue:** `writeHaltNote` (`internal/loomcli/halt.go:62`) is a no-op when the friction dir is empty, which `wiring.go` sets only when `loom.yaml`'s friction key is set.
`internal/frictionengine` moves covered notes into a timestamped archive sibling after reflection.
D7 says neither where the notes are read from nor what a run without them is.
As written, every `blocked` or `failed` change in such a run is `no-note`, which reads a missing source as a class and contradicts D5.
The in-order match also does not say what a halt note with no matching batten change counts as.
Open risks names the case that produces one: a halt resumed between two batten reads.

### [MEDIUM] Absent-versus-zero is not carried into composite figures

**Location:** `_lyx/discussion/decision-record.md:62`

**Issue:** D5 states the rule per source, but wakes per run, the board's first question, sum D6's batten, parent-review and alignment sources, and older archives lack some of them.
The Decision does not say whether such a run's total is partial, absent, or excluded from the week's mean.
D9 counts rounds "for each segment with a directory", so a run with no `discussion` directory is neither absent nor zero for that segment.
D11 gives no figure for a week whose fixes are all `no-pre-image`, where fresh plus aged is zero.

### [MEDIUM] Requirement-carrying rules lack acceptance criteria

**Location:** `_lyx/discussion/decision-record.md:161`

**Issue:** The first criterion's all-sources fixture "yields the expected count for each", and that is the only coverage for the superseding-approve wake (D6), goto and reroute entries and unaccounted resumes as LLM steps (D8), the `escalated` and `circling-decided` outcomes (D9), the Land and Worktree-Teardown failure wake (D6), the `no-note` class (D7) and the `--since` filter (D4).
Each of these is a distinct classification rule a test can get wrong while the fixture's totals still match.
The negative cases have their own criteria, such as an arbiter-backed resume not being an LLM step, but the positive rules do not.
This is filed once, as a method gap.

### [MEDIUM] Direct pushes to the parent branch are silently outside the fix share

**Location:** `_lyx/discussion/decision-record.md:119`

**Issue:** `main`'s first-parent history is a line of squash commits.
Loom landings and direct pushes look alike, for example `7452b52cb` and `c05fb1389` next to loom landings.
D10 leaves every direct push `unmapped`, and D11 blames only `fix` landings, so no orch repair pushed straight to `main` ever enters the fresh-code share.
The board entry's third question and its 2026-10-12 audit are about fixes landed on the parent branch.
The record does not say whether that exclusion is intended.

### [LOW] Outcome precedence between escalation and circling-decision files

**Location:** `_lyx/discussion/decision-record.md:110`

**Issue:** `internal/shedadapters/round.go` places `round-<N>-escalation.md` and `round-<N>-circling-decision.md` side by side in the run dir.
A circling escalation that a decision later settles leaves both.
D9 lists the outcomes as alternatives with no precedence.

### [LOW] Flag bounds stated only for the default range

**Location:** `_lyx/discussion/decision-record.md:55`

**Issue:** D4's Rationale says "the default range bounds D11's blame cost".
An explicit `--since` far in the past blames every fix landing since then, and `--fresh-days` has a floor but no ceiling.
The Decision does not state what an explicit value lets through.
The read-only guarantee against concurrent landings is stated: D3 resolves every ref once at the start, and the verb takes no lock and writes nothing.
No finding is raised there.

### [LOW] Q7 contradicts D10

**Location:** `_lyx/discussion/support-log.md:87`

**Issue:** Q7 says "unmapped commits are hand landings", while D10 says "a commit that maps to no run is kind `unmapped`".
The decision record governs because `Plan-Write` reads only it, so the plan writer is not misled.
The ledger still records a resolution that was later reversed by the goal-scope lens fix, and that misleads the review gate.

### [NIT] Rationales argue against rejected alternatives

**Location:** `_lyx/discussion/decision-record.md:57`

**Issue:** D4's "the next run's batten commits precede its own tag, so the next tag is the wrong bound" argues against an alternative that the support log's Rejected alternatives already records.
D1's "which a `tools/` program needs" does the same for the `tools/` program.
The writer stencil keeps rejected alternatives out of `## Decisions`.
Neither clause carries a constraint the plan writer needs, so moving them loses nothing.

The relocation pass found nothing else to move.
Auto-mode assumptions are pointers to their Decisions, Open risks are genuine risks, and the Notes subsection is optional pointers.
