---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:126
    summary: D10 maps landings by time to a batten Land done commit, but batten's Land row does not land; the child's own Finalize merge does, so the join picks whatever landed last, and the checkpoint source has no join rule at all
    fix: In D10, replace the second source's rule. State that the child's Finalize performs the parent-side merge and batten's Land row only pulls the prime, deploys and prunes the board, so a batten Land done commit is not the landing. Choose a join that names the landing commit itself, not the newest first-parent commit at or before a records-branch commit; for example, match the landing's merged-in task-branch commit against the Warp-SHA trailer that the archive tag's pending-records commit records (fabricengine commitPendingRecords). If no exact join exists for some landings, keep a time join only with a stated bound. A first-parent commit already claimed by another run or by the checkpoint source is never re-claimed. Two runs claiming one commit both become unmapped and are reported as a finding. Also name the window the join may span. For the first source, state how the checkpoint commit joins to a parent-branch commit. The checkpoint is written before the merge, so the join is the first first-parent commit after it, under the same no-reclaim bound. Correct the Rationale's claim that every loom landing leaves a batten Land done commit to say what the chosen source guarantees.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:131
    summary: D10 classes a mapped run with no batten Land done as a hand landing, but Land done is missing whenever Land's deploy, pull or board prune failed after the run's own Finalize had already landed it
    fix: In D10, redefine `hand` from a record that shows who performed the parent-side merge, not from batten's Land row state. Under F1's join, for example, `hand` is a mapped run whose loom history has no Finalize done before the landing commit. State separately how a run whose Finalize landed but whose batten Land never reached done is reported, for example as kind by labels plus a `land-incomplete` flag. Update the matching acceptance criterion at line 187 to say so.
  - id: F3
    severity: BLOCKING
    class: consistency
    location: _lyx/discussion/decision-record.md:67
    summary: D5 bounds tolerance to new report-only parse functions, but D9 adds a parse-from-bytes function only where none exists, and shedrun.ParseSeed already exists, is strict, and is the function the live ReadSeed calls
    fix: In D9, replace "added where none exists" with this rule. Each owning package gains a tolerant parse-from-bytes function for the report, separate from any existing strict one. Where a strict parse-from-bytes already exists and a live reader calls it, name `shedrun.ParseSeed` as the instance. That function and its live callers stay unchanged, and the report never calls it. In D5, state the same rule once and point D9 at it, so the bound "no live reader's acceptance changes" cannot be met by loosening an existing shared parser.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:206
    summary: Notes for the plan writer direct verdict reads through alignment.ReadVerdict, a path-based reader that decodes with KnownFields(true), contradicting D9's no-path-based-reader rule and D5's tolerance rule for archive-tag blobs
    fix: In Notes for the plan writer, replace "verdicts through `alignment.ReadVerdict`" with a pointer saying that `alignment` owns the verdict format and the report reads it through the tolerant parse-from-bytes function D9 adds there. Drop the `shedrun.AlignmentDir` path pointer, or reword it as the record's location inside the tag tree rather than a path to read through.
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:54
    summary: D4 never defines which fresh seed is a tag's own, so a slug's later tag with no batten run of its own (a hand-run or bench run) inherits the previous batten run's span and the same batten history counts twice
    fix: "In D4, define the run's own fresh seed as the newest fresh seed for the slug that is after the slug's previous archive tag's commit and at or before this tag's commit. When no fresh seed falls in that interval, the batten source is absent for that tag per D5. Add an acceptance criterion: a slug whose first tag came from a batten run and whose second tag came from a run batten never seeded reports the batten source absent for the second tag, and the first tag's batten history counts once."
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:97
    summary: D7 treats a missing friction dir as no halt-note source, but git never tracks an empty dir, so a Tier 2 run with no halt and no agent note, or one whose notes were all archived, also lacks it and reports stop classes absent instead of zero stops
    fix: "In D7, fix the presence test. The halt-note source is present when the tag's tree holds the friction dir or any of its timestamped archive siblings, or when another committed record shows Tier 2 was on for the run. Pick and name that record, for example a committed Tier 2 setting in the run's records. Otherwise state the bias in Open risks. Then a run with Tier 2 on and no notes reports zero stops, not absent. Add an acceptance criterion: a run whose only friction notes sit in an archive sibling reports its halt notes, and the stated Tier 2 record with no notes yields zero stops."
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:172
    summary: Several classification rules carrying requirements have no acceptance criterion a test can fail on its own, only coverage through the all-sources fixture or none
    fix: "In Acceptance criteria, add one criterion per classification rule that currently has none of its own. Examples of the method gap: D10's label mapping (bug to fix, feature or enhancement to feature, else other, labels as last recorded before the landing, including a pruned entry); D9's `accepted` outcome and parent-review outcome from the verdict record; D6's `pr-gate` wake kind and the Land ready-tasks kind always reported absent; D7's truncation of the reason at the first colon, semicolon or parenthesis and quoted-string placeholders; D11's exactly-half pre-image share being aged. Then state in the all-sources criterion that it checks counts only, not classification."
---

### [BLOCKING] D10's landing join attaches runs to the wrong commit

**Location:** `_lyx/discussion/decision-record.md:126`

**Issue:** D10's second source joins a run to "the newest first-parent parent-branch commit at or before" the batten status commit whose history first shows Land `done`.
Batten's Land row does not perform the landing.
`internal/battenshed/doc.go:127` describes Land as taking the land lock, pulling the prime, running `deploy` and pruning the board for a child whose Run-Shed already returned Done.
`internal/loomcli/landingdeps.go:306` shows that the child's Finalize performs the parent-side merge.
So the landing precedes Land `done`, sometimes by a long gap.
Land waits up to ten minutes on a dirty prime and up to thirty minutes on `deploy`.
Batten's status commit is also skipped while the prime is mid-merge (`internal/battencli/commitstatus.go`), which pushes the Land `done` commit later still.
Finalize merges serialize under the parent lock, not batten's land lock, so another child's Finalize merge or a direct push can land in that gap.
"Newest at or before" then picks that other commit.
The run is attributed to the wrong landing, and its real landing becomes `unmapped`, or two runs claim one commit.
The Decision states what a landing matching no source becomes, but it puts no bound on the fallback.
Nothing prevents a commit from being claimed twice or claimed against another source's claim.
The first source has no join rule at all.
It names the checkpoint commit but never says which parent-branch commit it maps to.
That commit is written before the merge (`landingdeps.go:305`), so the second source's "at or before" rule, applied by analogy, would select the previous landing.
The record-side premise behind the checkpoint holds.
`parentRecordsCommitter` commits only when the parent's runs root holds a file and the commit is non-empty, so the fallback is needed.
The fallback as written is the defect.

### [MEDIUM] `hand` is inferred from batten's Land row, which does not land

**Location:** `_lyx/discussion/decision-record.md:131`

**Issue:** "A mapped run whose batten history has no Land `done` was landed by hand" rests on the same false premise as F1.
Land `done` is missing whenever Land fails after Finalize has landed the run: a stuck `deploy`, a failed prime pull or a failed board prune.
Each case would count an automated landing as `hand`, which inflates the LLM-step count D8 draws from hand landings.

### [BLOCKING] D9's "added where none exists" reaches a live strict parser

**Location:** `_lyx/discussion/decision-record.md:67`

**Issue:** D5 confines tolerance to "D9's parse-from-bytes functions, which tolerate unknown fields and sit in each owning package beside the existing reader", and bounds it with "no live reader's acceptance changes".
D9 (line 121) adds a parse-from-bytes function only "where none exists".
`shedrun.ParseSeed` (`internal/shedrun/seed.go:131`) is an existing parse-from-bytes function for the archived seed that D4 reads.
It decodes with `DisallowUnknownFields`, and the live `shedrun.ReadSeed` calls it (`seed.go:121`).
Read literally, D9 sends the report through `ParseSeed`.
That breaks D5's acceptance criterion "a record with an unknown field reads".
The other option, making `ParseSeed` tolerant, loosens every live seed read and breaks D5's bound.
The two Decisions cannot both hold for this record.
The plan writer is left to pick which rule to violate.

### [MEDIUM] Notes point the plan writer at a path-based, strict verdict reader

**Location:** `_lyx/discussion/decision-record.md:206`

**Issue:** The note says "verdicts through `alignment.ReadVerdict`".
`ReadVerdict` takes an `Attempt` and reads its `VerdictPath()` from disk (`internal/alignment/records.go:173`), and it decodes with `KnownFields(true)` (`records.go:184`).
D9 says "no path-based or locking reader is called", and D5 says the report's parse functions tolerate unknown fields.
The Plan producer reads only `decision-record.md`, so this pointer is the most concrete instruction it gets for verdicts, and it contradicts both Decisions.

### [MEDIUM] D4's batten span has no rule for which fresh seed is a tag's own

**Location:** `_lyx/discussion/decision-record.md:54`

**Issue:** The span runs "from the run's own fresh seed up to the slug's next fresh seed", but the Decision never says how a tag finds its own fresh seed.
The natural reading is the newest fresh seed at or before the tag's commit.
Under that reading, a slug whose first run went through batten (tag T1) and whose second run did not (a hand-run or bench pair, tag T2) gives T2 the same fresh seed as T1.
With no later fresh seed, T2's span runs to the branch tip.
Both tags then read the first run's batten history, so its wakes and automatic steps count twice.
The span needs a lower bound at the slug's previous tag, and an absent result when no fresh seed falls between the two tags.
The cleanup case and the no-later-seed case are otherwise covered by the newest-commit-holding-the-file rule.

### [MEDIUM] D7's absent test confuses a clean Tier 2 run with no source

**Location:** `_lyx/discussion/decision-record.md:97`

**Issue:** D7 says that a run whose tree holds no friction dir, "because Tier 2 was off or the run predates halt notes", reports stop and failure classes absent.
The friction dir is durable (`loomengine.LoomFrictionDir`, `internal/loomengine/config.go:325`).
`ensureFrictionDirAfterSeed` creates it with a bare `os.MkdirAll` (`internal/friction/friction.go:221`), and git does not track an empty directory.
A Tier 2 run with no halt and no agent friction note therefore has no friction dir in its tag tree, and neither does a run whose reflection moved every note into a timestamped sibling.
Both report absent instead of zero stops.
The stop-class weekly figures then cover only runs that had notes, so clean runs drop out and recurrence looks denser than it is.
The other two points of this round's focus hold.
D7 states where halt notes are read from and what a leftover note counts as.

### [MEDIUM] Classification rules lean on the all-sources fixture for their only test

**Location:** `_lyx/discussion/decision-record.md:172`

**Issue:** The first criterion's all-sources fixture checks "the expected count for each" source.
A wrong classification that keeps the count right passes that fixture.
Several rules that carry a requirement have no criterion of their own:
- D10's label-to-kind mapping, its "as last recorded before the landing" timing and its pruned-entry case.
- D9's `accepted` outcome and the parent-review outcome from the verdict record.
- D6's `pr-gate` wake kind and the Land ready-tasks kind, which is always absent.
- D7's leading-clause truncation and its quoted-string placeholder.
- D11's threshold at exactly half.
This is one method gap, filed once.
