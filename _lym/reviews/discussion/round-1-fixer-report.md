---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: "The proposal derives a landing line for any tag whose history holds a Finalize entry, but Stuck outcomes enter the history too (internal/shedengine/reroute.go:49, :70), so a run cancelled after a Stuck Finalize would read as landed. D6 requires a Finalize entry with the outcome Done."
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed-differently
    reason: "The proposal names fix review copies by the attempt record's review round, but ReviewRounds resets on a new row entry, on the record's removal and on a change past an operator's acceptance (internal/landingshed/finalize.go:1124-1126), so copies would collide as F3 describes. They use the same self-allocated sequence as the fixer copies."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed-differently
    reason: "Appending the rule to PATTERN-pair-remote's text alone leaves its trigger at pushing records or tearing down a pair, which a future cleanup verb that prunes tags would not match. The trigger also names deleting an archive tag, and the entry's existing clauses were tightened to keep its rule text within the 280-character cap."
---

## Changes

All in `_lym/discussion/decision-record.md`, with round 1 logged in `_lym/discussion/support-log.md`.

- F1: D5 adds a landing record in the kept directory, written right after the parent-side merge and before the parent push.
  A resumed Finalize whose merge reports already up to date reads the record back instead of the parent's head.
  The record is rewritten when the merge makes a new commit and when an archived parent merge re-reads the commit.
  A new criterion covers another run landing between the landing and the resume.
- F2: D4 copies each fixer turn's report in the gate check that judges the turn, before a re-prompt or halt.
  The rationale now says each new session already renames the earlier report, and only turns within one session overwrite it.
- F3: D4 names each copy by row, row entry, kind and a sequence the copy allocates in the subdirectory, and a copy never replaces a file.
  A criterion covers a copy made after the attempt record was removed or a turn was given back.
- F4: D1 adds the tracked `.gitignore` for `.tmp-*`, with its criterion, and D3 points to that rule instead of its old claim.
- F5: D3 seeds the live ask record from the tracked copy, with its criterion.
- F6 (fixed differently, see frontmatter): D6 lists a tag without an index row and without a Done Finalize entry as not landed, giving its reason.
  D5's rationale no longer calls the tags the list of landed work, and D6's rationale says why.
  New criterion: a cancelled run is listed as not landed.
- F7: D6's Bound gains the four rules: regular files only, a symlink or non-directory `<dir>` refused, a `--tag` outside the slug refused, and the write staged in a sibling directory.
  The refusal criterion and the extract criterion cover them.
- F8: D5 writes the row after the parent push and any archive, before `pushRecords`, and reads the pull request URL before any beam record is dropped.
  "And any later fixer" is gone, and the Scope line and the criteria match.
- F9 (fixed differently, see frontmatter): D4, the Goal, the Scope and the auto-mode line now keep the fix review reports.
  New criterion: a review that rejects once and then approves leaves two reports.
- F10: D2 keys copies by content hash and skips a hash it already holds.
  The rationale is narrowed to findings sets, and the criterion covers a re-run.
- F11: D5 records `_lym/`-relative paths.
- F12: D5's sentence on the landing message now names the credit and flaky sections and a hand-made merge.
- F13: the Constraints name D2's synchronous push through the pull request record.
- F14: D7 drops the exception, and the plain-namespace listing is named as a way forward the operator runs.
- F15 (fixed differently, see frontmatter): `PATTERN-run-records` keeps its records clauses, its `(test)` marker and its background link.
  The retention rule moved to `PATTERN-pair-remote`.
  To keep that entry's rule text under the PATTERN.md cap of 280 characters, which the gate enforced, its existing clauses were tightened without changing their meaning.

## Self-check

The changes below came out of the self-check and are already in the decision record.

- fix-consistency, LOW, support log still calls the tags the list of landed runs: fixed (Rejected alternatives and Q1 reworded).
- fix-consistency, LOW, D1 credits the landing record and index row to D2 to D4: fixed (D2 to D5).
- fix-consistency, LOW, D6's derived criterion does not say the run landed: fixed (criterion says "one landed run without").
- fix-consistency, LOW, the PATTERN-run-records text omits fix review reports and the landing record: fixed.
- fix-consistency, LOW, D4's identical-report skip leaves out the kind; merged with fix-proposal's note that the skip collapses two failing turns: fixed by dropping the skip rule.
- fix-claims, MEDIUM, and fix-proposal, BLOCKING, a Stuck Finalize counts as landed in D6: fixed (Finalize entry with the outcome Done; the criterion covers a Stuck Finalize).
- fix-claims, MEDIUM, Parent-Align runs no fix review: fixed (D4's rationale names Finalize only).
- fix-claims, LOW, wrong causes for review rounds repeating: fixed (the rationale lists each counter's resets).
- fix-way-forward, MEDIUM, and fix-proposal, MEDIUM, the landing record goes stale after the operator resets the parent by hand: fixed (the record is rewritten whenever the merge makes a commit and read back only on an already-up-to-date merge; criterion added).
- fix-way-forward, LOW, and fix-proposal, MEDIUM, the extract's sibling breaks the bound and "absent" is false for an existing empty `<dir>`: fixed (the bound names the sibling, a failure leaves `<dir>` as it was and removes the sibling, and a kill leaves it for the operator).
- fix-way-forward, LOW, the re-run skips a copy whose commit failed: fixed (the verb's commit stages the PR findings subdirectory whole).
- fix-way-forward, note, the `.gitignore`'s own temp file: fixed (it is written with a plain write before any other file).
- fix-proposal, MEDIUM, `landingshed`'s atomic writer uses `.<kind>-*.tmp` names: fixed (every kept file is written through `internal/state`).
- fix-proposal, MEDIUM, only writes seed the ask record: fixed (a read seeds too; criterion extended).
- fix-proposal, LOW, an identical second send-back is not copied, and the send-back path computes no hash: fixed (the hash is computed on that path too, and the skip of a byte-identical send-back is stated).
- fix-proposal, LOW, PATTERN-pair-remote's trigger misses a prune verb: fixed (the trigger names deleting an archive tag).

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
