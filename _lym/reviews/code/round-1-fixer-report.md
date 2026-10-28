# Fixer-report — code round 1

## Changes

- F1: `prepare_test.go` lines 293 and 295 and the `settings.go` `noticeTextForbiddenChars` comment now name the batching sentence and say the notice shares its line with the sentences that follow it.
- F2: the reading sentence in `burler-step-1-explore.md` now sits before the `{{.tool_use_rules}}{{.fork_reading}}` line, which stays last and unchanged.
  Fixed differently: the proposed wording "Read the files of your focus ..." was changed to "Make the reads your focus needs, or the whole target when no focus is set, in one message."
  Reason: the self-check fork showed "the files" clashes with `burler-fork-reading.md`'s "Read definitions and call sites, not whole files".
  The `template_test.go` comment the fix named is left as it was: with the marker line last again, "at the end of the stencil" is accurate.
- F3: the creel entry in `docs/overview.md` names the creel sentence and the further sentence asking for batched calls.
- F4: step 2 of `webster-body-implementer.md` runs the card's gate "as step 3 lists it, after the card's last edit rather than after each edit".
- F5: `darn-template-write.md` Step 4 item 1 reads "before the first edit of each commit"; item 2 has one subject.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal: LOW, F2 wording clashes with the fork reading's whole-file rule — fixed with my own change (see F2).
- fix-proposal: LOW, F5 item 2 has a second semicolon, stylistic only — no change, the reviewer's proposed wording stands.
