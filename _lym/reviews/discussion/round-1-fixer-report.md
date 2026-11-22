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
    disposition: fixed
  - id: F7
    disposition: fixed
---

# Round 1 fixer report — discussion

## Changes

- F1: D2 now asks the live writer by creel while no pause covers the slug, then holds the run with `lym tender pause <slug>`, and the orch adds the tip SHA and merge base read after the hold.
  D2's Rationale pins `lym batten pause` to batten's own row boundary, the inner run to the tender pause record, and the pane hold of a covering pause, with the cited evidence.
  It also pins that a slug-scope pause does not block the cancel, which lifts it on completion (`internal/battencli/cancel.go:299-304`, `:572-576`).
  D1's verb list, D3 (holds, then writes), D4 step 1, D7 (`lym tender resume <slug>`), the Goal and the D2, D3 and D7 acceptance bullets follow.
- F2: the Constraints bullet on `--remove` now says the tags are created locally and a detached teardown child pushes them after the command returns, with the failure notice, pinned at the cited fabricengine lines.
  D4 step 3 names the code tag from the recorded tip, never from `archive_tags`, and confirms it on origin with `ls-remote`, repeating until it shows or a failure notice arrives.
  The owner Constraint depends on that check.
- F3: new D4 step 2 checks before the cancel that the new parent can take a member, and stops the move with `lym tender resume <slug>` and a word to the operator when it cannot; the D4 acceptance bullet carries it.
- F4: new D4 step 5 runs `git -C <records repo> fetch --prune origin` before the release while `stale-records-ref-blocks-rerun` has not landed, citing that entry's body; Open risks points at the step; the acceptance bullet names it.
- F5: D4 step 3 defines a partial finish and a refusal by the cancel's output; D7 points at that definition.
- F6: the stored-record branch of D4 step 4 adds an H3 `### Move note` under `## Notes for the plan writer`, adding that section last when missing, with the parts as bullets or H4 headings.
- F7: D8's glossary line reworded as proposed.
- Support log: a Round 1 section, three rejected alternatives, and Q6 and Q8 resolved.

## Self-check

- fix-consistency, LOW, D1's verb list misses `lym board get` and the `git -C` steps: fixed, D1 lists them.
- fix-consistency, LOW, support log's "asking the writer without holding the run" reads against D2: fixed, reworded to reading the tip without a hold.
- fix-consistency, LOW, Turn 1 and Turn 2 lines on `archive_tags` left uncorrected: fixed, the Round 1 F2 line names the correction.
- fix-claims, MEDIUM, "status is empty" is narrower than `RunHolds`: fixed, "no run status (`<state> · <producer>`)".
- fix-claims LOW, fix-proposal LOW and fix-way-forward MEDIUM (merged), origin's tip strictly containing the local tip leaves the recorded tip's tag off origin: fixed, D4 step 1 records origin's tip in that case.
- fix-way-forward, MEDIUM, no rule when the tag is not on origin yet: fixed, D4 step 3 repeats the check until the tag shows or a failure notice arrives.
- fix-proposal, MEDIUM, the prune can run before origin deletes the records branch: fixed, D4 step 5 prunes once `ls-remote` lists no records branch of the slug.
- fix-proposal, LOW, a persistent partial-finish error loops: fixed, a repeated same error stops the move and goes to the operator with the pause in place.

## Disputed

None.

## Deferred

None.

## Rulings

None.
