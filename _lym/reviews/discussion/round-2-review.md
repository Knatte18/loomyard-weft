---
verdict: BLOCKING
findings:
  - id: F16
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D3
    summary: "D3's seed restores the live ask record from its tracked copy but not the report files its done asks name, so after an adopt or takeover every reader of a done ask's report errors where it used to read ask-again"
    fix: "In D3, after the sentence that seeds the live record from the tracked copy, add: the same seed writes each tracked report copy back to this anchor's live report path (ParentReviewFiles.ReportPath of its ask number), and rewrites every seeded ask's Report field to that live path, all under the record's lock before any reader sees the seeded record. Extend the D3 acceptance criterion on deletion: after the live record and the live reports are deleted with the tracked copies present, the PR-Review standing read and the round settle of a done ask read its report and succeed."
  - id: F17
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    carries: F1
    location: _lym/discussion/decision-record.md#D5
    summary: "On the already-landed path with no landing record, and after the archive of a held parent merge, D5 still takes the parent's head as the landing commit, so a hand merge or an adopter-first landing followed by another run's landing gives the row that other run's commit, time and message"
    fix: "In D5, replace the rule that a merge with no record reads the parent's head with: when the parent-side merge makes a commit, the landing commit is the parent's head; when it reports already up to date and no record exists, or after the archive of a held parent merge, the landing commit is the first commit on the parent branch's first-parent history that holds the task's work, found through fabric, and the row's landing time is that commit's committer time. Drop the sentence that the archive reads the landing commit again from the reset branch. Add a D5 acceptance criterion: a parent that received the task's work by a hand merge, with another commit landed on top of it before Finalize runs, gives an index row naming the hand merge."
  - id: F18
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    carries: F7
    location: _lym/discussion/decision-record.md#D6
    summary: "D6's bound rule 4 still lets the extract write outside <dir> and its own new sibling: a sibling left by a killed extract or planted as a symlink is written into or through, and renaming the sibling onto an existing empty <dir> relies on rename(2) replacing an empty directory, an unpinned fact that fails on Windows"
    fix: "In D6's bound rule 4, state that the extract creates the sibling with a create that fails when the path exists, and refuses an existing sibling of that name, naming the way forward to remove it by its path; and that an existing empty <dir> is removed with a remove that deletes only an empty directory right before the rename, so the rename never targets an existing directory. Add to the D6 refusal criterion a leftover sibling, and to the extract criterion an extract into an existing empty directory."
prior:
  - id: F1
    status: open
    answer: "The landing record covers a resumed Finalize, but the already-landed and archive paths still read the parent's head; carried as F17."
  - id: F2
    status: resolved
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: resolved
    answer: "Numbering now continues past the tracked copy; the seed's missing reports are a new defect, raised as F16."
  - id: F6
    status: resolved
  - id: F7
    status: open
    answer: "Rules 1 to 3 hold; rule 4's sibling and rename still break the write bound; carried as F18."
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
  - id: F15
    status: resolved
---

### [BLOCKING] D3's seed restores asks whose reports the new owner lacks

**Location:** `_lym/discussion/decision-record.md#D3`

**Issue:** D3 seeds the live ask record from the tracked copy after an adopt or takeover, but each done ask's `Report` field holds the old anchor's `.lym/parent-review/ask-N.report.yaml` path (`internal/burlerengine/parentreview.go:390-394`, from `ParentReviewFilesOf` at :145-156).
The seed restores no report file there.
`ReadPRReviewStanding` returns the read error (`internal/landingshed/prreviewask.go:118-121`).
The parent seat's coverage gate errors (`internal/burlerengine/parentseat.go:83`), `SettleParentAsk` fails reading the report (`internal/burlerengine/parentreview.go:314-316`), and drift's parent judge errors (`internal/driftengine/judgeparent.go:236`).
Before the seed, a new owner found no record and read ask-again.
With the seed, every PR-Review standing read and every round settle of a done ask errors after the move.
Round 1's F5 fix introduced this regression, on a changed hunk.
Three lenses found it independently, and the handler confirmed the readers.

### [BLOCKING] D5's landing commit is still read from the parent's head on the already-landed and archive paths

**Location:** `_lym/discussion/decision-record.md#D5`

**Issue:** This carries F1.
The landing record fixes a Finalize resumed after its own merge.
But when no record exists and the merge reports already up to date, D5 still reads the parent's head (`internal/landingshed/finalize.go:415-419`, `:742`).
That happens when the operator merged the task on GitHub by hand or an adopter landed first, and `syncParent` fast-forwarded the parent to its upstream (`:519`).
The archive of a held parent merge re-reads the head the same way (`:853-854`).
When another run landed on the parent between the hand merge and this Finalize, the row's landing commit, landing time and the message it points to as the Describe text as merged are that other run's.
D5's own sentence, that a hand merge's message is what the row's landing commit names, is false on this path.
The study this task serves reads exactly those fields, so the defect is silent wrong data, which is why F1 was BLOCKING.

### [MEDIUM] D6's extract sibling and rename still break its write bound

**Location:** `_lym/discussion/decision-record.md#D6`

**Issue:** This carries F7.
Rules 1 to 3 of D6's bound hold.
Rule 4 names a sibling after `<dir>` and leaves it behind after a killed extract, but says nothing about a sibling that already exists.
A leftover sibling mixes stale files into the next extract, and a sibling planted as a symlink sends the writes outside `<dir>` and the sibling.
Rule 4 also renames the sibling onto `<dir>`, which D6 accepts when it exists and is empty.
On Linux, rename(2) replaces an empty directory.
On Windows, Go's `os.Rename` onto an existing directory fails, and the repository handles drive-letter paths (`internal/fsx/fsx.go:32-34`).
The record pins neither behaviour.

## Below the incremental bar

This round is incremental, so a new finding is raised only at BLOCKING.
These lens findings are real but below that bar, and are listed here for the fixer's judgment, not as findings:

- D2 does not order the send-back copy before the record update, and a re-run returns at the stored send-back mark before any copy (`internal/loomcli/prfindings.go:218-220`), so a kill between the record update and the copy leaves no copy (bound-breaker B3).
  D2 also describes the committer as already staging the PR findings subdirectory, but `pullRequestRecordCommitter` stages only the pull request record (`internal/loomcli/landingdeps.go:528-531`); the decision should word it as a change (existing-paths 3).
- D5's pull request URL is lost when the resolver's no drops a non-beam record before the landing, and on a Finalize re-entered after `dropBeamRecord` (walkthrough F-W3, bound-breaker B9).
- D1's claim that `git add -- <dir>` skips the `.tmp-*` names a `.gitignore` lists is not pinned by a test, recorded output or the docs (bound-breaker B6).
- D4's per-turn copy needs to run once per judged turn, after the `decided` short-circuit in `fixerSession.judge`, and D4's bound wrongly says a timed-out turn never reaches the gate check (walkthrough F-W5, bound-breaker B7).
- The Constraints name D2 as the only synchronous push, but Finalize's own records push, which carries D5's row, is synchronous too (interactions 1).
- The declared `PATTERN-pair-remote` text "archive tags are never deleted" sits beside a way forward lym prints that moves a plain-namespace archive tag and deletes the plain ref (`internal/fabricengine/cleanupremotewarp.go:374-376`) (interactions 3, existing-paths 4, bound-breaker B8).

## Rejected

- Bound-breaker B4, that D6 lists a landed motif run as not landed because reportengine sorts non-selected recipes as partial: D6's not-landed test reads the loom history for a Finalize entry with the outcome Done, and uses the partial run's reason only for a tag that test already marks not landed. The record does not route the Done test through `readLoomRecords`' early return.
- Interactions 2, that the row's path list misses an ask answered after the landing: the path list is a convenience. D6's extract writes the kept directory whole from the tag, and teardown's commit carries a later copy, so nothing is lost.

## Focus departures

None. On focus 8: D7's claim that no code deletes an `archive/<slug>/*` tag holds, and the pr-findings push is the synchronous `pullRequestRecordCommitter` push the Constraints name. The plain-namespace way forward is listed above, below the incremental bar.
