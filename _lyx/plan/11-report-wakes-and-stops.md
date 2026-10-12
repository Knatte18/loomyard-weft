# Card 11 — report-wakes-and-stops

**Create:**
- `plan:internal/reportengine#WakeSource` -> `type WakeSource struct`
- `plan:internal/reportengine#Stop` -> `type Stop struct`
- `plan:internal/reportengine#AlignmentOutcome` -> `type AlignmentOutcome struct`
- `plan:internal/reportengine#failureClass` -> `func failureClass(producer, reason string) string`
- `//internal/reportengine/wakes_test.go`

**Edit:**
- `//internal/reportengine/doc.go`
- `//internal/reportengine/fake_test.go`
- `//internal/reportengine/seam_enforcement_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/battenshed#ClassifyRecords`
- `plan:internal/battenshed#NoticeRules`
- `plan:internal/haltnote#Parse`
- `plan:internal/haltnote#IsNoteName`
- `plan:internal/loomengine#LoomFrictionDirRel`
- `plan:internal/loomengine#IsLoomFrictionArchive`
- `plan:internal/alignment#ParseAttemptDir`
- `plan:internal/parentreview#ParseRoundDir`
- `plan:internal/parentreview#RoundNumber`
- `plan:internal/shedadapters#ReviewRounds`
- `internal/loomrecipe#BouncerRunSubdir`
- `internal/loomengine#LoomParentReviewDirRel`
- `internal/loomengine#LoomReviewsDirRel`

**Intent:**
Each full run gains its orch wakes by source and kind, batten's automatic steps, its paused changes and alignment outcomes (D6), and its stop and failure classes (D7), in new files `wakes.go` and `stops.go`.
Every record is read at the run's tag commit, or at its batten snapshots, through the owning package's tolerant parse; a record that does not decode is a finding naming the tag and the file, and the run's other sources still count (D5).

Wakes, per source, each `nil` on the `RunRow` when the run lacks the source:
- `batten`: `battenshed.ClassifyRecords` over the run's batten snapshots and its loom history at the tag, told `NoticeRules{VerifyRows: [loomshed.NameMergeIn, loomshed.NamePublish], GateRow: loomshed.NamePRGate, EpisodeRows: [battenrecipe.NameLand, battenrecipe.NameWorktreeTeardown]}` and an `Escalations` map built per changed producer: when `loomrecipe.BouncerRunSubdir` names the producer a Bouncer, the count of `shedadapters.ReviewRounds(...).EscalatedRounds` over the file names in that subdirectory of `LoomReviewsDirRel()` at the tag.
  Each `Wake` counts under its kind (`review-escalation`, `pr-gate`, or `<state> at <producer>`), each `AutomaticStep` under `RunRow.Automatic` by kind, and each paused change under `RunRow.Paused`, none of them a wake.
  The run keeps the classification's `Changes` and `AccountedChild` in unexported `RunRow` fields, which encode to no JSON, for the stop matching below and card 12's LLM steps.
- `parent-review`: each round directory under `LoomParentReviewDirRel()` that `parentreview.RoundNumber` accepts is read through `ParseRoundDir`; a round whose `Alignment` is nil or not in state `applied` and whose `Delivery.DeliveredAt` is set is one wake, fallbacks included, and every round's `Delivery.Pings` sum into `RunRow.ParentReviewPings`, beside the wakes and never as one.
  The source is absent when the tag's tree has no parent-review directory.
- `alignment`: each attempt directory under each point directory of `TreeRunRels(slug).Alignment` is read through `ParseAttemptDir`; an attempt whose `Applied` is `approve` and whose verdict lists superseded entries is one wake.
  Every attempt also counts in `RunRow.Alignment`, an `AlignmentOutcome` per point and verdict word (`none` when no verdict file was written) with its count and how many of those were applied, so wakes the agent absorbed show.
  The source is absent when the tag's tree has no alignment directory.
- `land-ready-tasks`: always absent, since its records are ephemeral and no git object holds them (D6).
A `WakeSource` holds a source's per-kind counts and its total; `RunRow.Wakes` maps source name to `*WakeSource`, so a run lacking one source still reports its total per source (D5).

Stops (D7), with the halt-note source present when the tag's tree holds `LoomFrictionDirRel()` or a directory under `LoomDurableDirRel()` that `IsLoomFrictionArchive` accepts:
- Notes are the files `haltnote.IsNoteName` accepts, in archive directories by name and then the live friction directory, each directory's notes in suffix order, parsed by `haltnote.Parse`.
- Each batten change into `blocked` or `failed`, in order, is matched to the first unmatched note with the same producer and state; a matched pair is one `Stop` classed by the note's anomaly, with its state and producer, so a stop and its note count once.
- With the halt-note source present, such a change with no matching note is a `Stop` of class `no-note`; a note left unmatched is a `Stop` of its own class with no batten change, which covers a halt resumed between two batten reads.
- A change into `awaiting` is a `Stop` of class `awaiting` at its producer, never a failure.
- Each stop carrying a note has a failure class `failureClass(producer, reason)`: the producer, `: `, and the reason's leading clause up to the first `:`, `;` or `(`, with each quoted string replaced by `<str>`, each path (a token holding `/`) by `<path>` and each digit run by `<n>`, spaces collapsed and trimmed; so `status-missing`, `watchdog` and `publish_verify failed after merging parent branch` each form one class.
- A run whose tree holds neither the friction directory nor an archive sibling reports `Stops` and `Failures` as absent (`nil`), whatever its batten changes.

`seam_enforcement_test.go`'s allowlist gains `battenshed`, `haltnote`, `loomengine`, `loomrecipe`, `alignment`, `parentreview` and `shedadapters`.
`fake_test.go` gains builders for a review segment's round files, a parent-review round directory, an alignment attempt directory and a halt note.
`doc.go` describes the wake sources, the automatic and paused steps, and the stop matching.

`TestCompute_WakesAndStops` (new, in `wakes_test.go`) is table-driven over fake sources, each row one run and its exact per-source wakes, automatic steps, paused count, alignment outcomes, stops, failure classes and findings:
- the engine's own part of the batten source, the classification rules themselves being card 7's rows: a Bouncer escalation that halts the child (an `awaiting` at `Plan-Bouncer` whose `plan` segment holds a round with an escalation file) is one `review-escalation` wake, which pins the `Escalations` map built from the review files; the same `awaiting` with no escalation file in the segment is a wake of kind `awaiting at Plan-Bouncer`;
- one wiring row whose batten record holds an `awaiting` at `PR-Gate` left to the orch, a `Publish` verify retry and a `Land` failure episode shows one `pr-gate` wake, one `failed at Land` wake and one automatic step under the `batten` source, which pins the row names the engine tells `NoticeRules`; every row's `land-ready-tasks` source is absent;
- a parent-review round settled by an applied alignment verdict is no wake, a fallback round that delivered is one wake, its two pings show in `ParentReviewPings` and add none, and a round whose `delivery.json` records no delivery is no wake;
- an applied alignment approve naming superseded entries is one alignment wake, and an `approve` at `parent-review`, a `reject` at `pr-gate` and an `escalate` at `pr-gate` each show under their own point and verdict;
- a run with no parent-review directory reports that source absent and its batten and alignment totals per source;
- an `awaiting` stop at `PR-Gate` is classed `awaiting` and has no failure class, and a `failed` change with its halt note is one stop;
- a `blocked` stop's class is its note's anomaly with the note's state and producer;
- two halts whose reasons differ only in a path, a number or a quoted string share a failure class, and two that share the text before the first `:`, `;` or `(` share one too;
- a batten `failed` change with no note in a run holding notes is class `no-note`, and a note left after matching is its own stop;
- a run whose only notes sit in an archive sibling reports them, and a run with neither a friction directory nor an archive sibling reports stops absent;
- a halt note that `haltnote.Parse` refuses is a finding naming the tag and the note's path, and the run's other stops stand.

`cmd/lyx/testdata/test-budget.yaml` raises `internal/reportengine` to 3.

**ImpactSummary:** Extends the unreleased report engine and its test support; nothing outside `internal/reportengine` and the budget file changes.

**Commit:** `11: report engine: wakes, automatic steps and stop classes`
