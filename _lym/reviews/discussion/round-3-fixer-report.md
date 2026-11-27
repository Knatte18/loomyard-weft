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
---

# Round 3 fixer-report

Every finding's premise held against the code: `internal/fabricengine/pushanchored.go:3` (prime push is records-only), `internal/fabricengine/failedpush.go:250` (`git push` for the prime's code branch), `internal/landingshed/finalize.go:957`, `:987`, `:993` (`handedOverStop` around the parent push), `internal/runfsm/act.go:229` (`lym fabric remove` named as a step), and `internal/shedrun/paths.go:201` (`ScratchDir`).

## Changes

All in `_lym/discussion/decision-record.md`, with a Round 3 entry in `_lym/discussion/support-log.md`.

- F1:
  - D5's refusal now names the parent target's push: `git push` in the prime's worktree for the prime's code branch, and `lym fabric push` for a beam parent.
  - A landing whose work origin's parent already holds under another commit is dropped by the pusher. The drop makes no `done` write, clears the refused-push record, and its pair hold outlives the entry so the held records commit reaches no branch.
  - A run owned elsewhere is then handed over through `pairteardown.HandOverOwnedElsewhere`, which resets the parent when the landing commit alone puts it ahead.
  - When other commits stand on the landing commit, the refusal names a rebase, and the next pass after it reads the remaining tips again.
  - D3's way-forward sentence and its criterion name the verb per target. New D5 criteria cover the prime's way forward clearing the record, and the drop.
- F2:
  - D5 and the PATTERN-owner-machine constraint read the owner of every run the parent push carries, and that run's owner again before its `done` write.
  - A run owned elsewhere is a refusal naming `lym adopt` of that run, or leaving the run to its owner.
  - It holds back only the tips that carry its merge.
  - New D5 criterion: a beam run adopted elsewhere gets no push and no `done` write.
- F3:
  - D13: while a landing entry stands, every other teardown refuses before any mutation, `lym fabric remove` with `--force` or `--remote` included. The check sits in the `internal/pairteardown` teardown composite, and its way forward names the owed parent push.
  - A removal after a dropped landing pushes the held commit to no branch.
  - New D13 criterion added.
  - D5 points to the bound at the records-hold sentence.
- F4:
  - D9 writes the pre-merge task head to a base file under `shedrun.ScratchDir` before merging, and keeps it until the removal has run. A resume reads the base from that file alone.
  - The sentence about writing the merge record before the clean check, which only served the old base source, is gone.
  - The D9 criterion adds a kill between the merge-in and the record write, on a first merge-in and on a later one.

## Self-check

All four forks returned a report. Duplicates are merged and listed once.

- fix-consistency, fix-proposal and fix-way-forward, MEDIUM: the drop case left the pair hold, the held commit and the actor that removes the entry and clears the record unstated. Fixed: the pusher makes the drop, clears the record, the hold outlives the entry, and the pair goes by hand-over or by `lym fabric remove`.
- fix-proposal and fix-way-forward, MEDIUM: the run-owned-elsewhere refusal pointed at adoption when the owner had already landed the run, and its hand-over option could not be run. Fixed: the pusher checks before each push whether origin holds the run's work and hands the run over itself, and the option now reads "leave the run to its owner".
- fix-claims, MEDIUM: the hand-over already resets a parent that its landing commit alone puts ahead. Fixed: the text states that reset, and keeps the refusal for the case where other commits stand on top.
- fix-claims, MEDIUM: the hand-over pushes the local records tip under an owner archive tag. Fixed: the text says the held commit reaches origin only under that tag, never on a branch, and that the hand-over's pre-push of the records stops short of it.
- fix-consistency, LOW: D3's way-forward list left out the prime's records branch. Fixed: reworded back to examples of "the way forward it names".
- fix-consistency, LOW: D3's all-clear reason said "every `done` write". Fixed: it now says "settled or dropped".
- fix-consistency, LOW: D7 did not list the new refusals. Fixed: D7 lists them.
- fix-proposal, LOW: the hold-back reason overreached. Fixed: only tips that carry the run's merge are held back.
- fix-way-forward, LOW: the re-read after a rebase had no actor and no test. Fixed: the next pusher pass makes it, and the D5 drop criterion covers it.

## Disputed

None.

## Deferred

None.

## Rulings

None.
