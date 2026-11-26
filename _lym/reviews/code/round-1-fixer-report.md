---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The board carry renames the value on every walk, behind no marker. The beam-pull-request-modes marker is committed in the hub-wide write, before the separate board write, so a failed rewrite behind it would never retry. No layer admits on or merge, so renaming them again is safe."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: "Only the continue row was added. The proposed approval row cannot fail under a reorder: the approval ending needs status.HeadSHA to equal rec.GatedHead (internal/landingshed/prreview.go, judge), and foreignHeadHalt passes that head."
  - id: F7
    disposition: fixed-differently
    reason: "The blast radius is `hub`, and the proposed list went into a new `## Upgrading` section. contracts/specs/loom-final-summary-spec.md allows one word there, and lym loom validate-description refuses more."
  - id: F8
    disposition: fixed-differently
    reason: "The proposed sentence reads backwards: after 'halts unless X', 'otherwise it is reopened' points at the halting case. The comment now says which closed pull requests are reopened and merged, and that any other halts."
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed-differently
    reason: "The test runs prContinueCmd through clihelp.Execute on a loomCLI over a temp location, instead of clihelp.ExecuteIn over a pair. It reads the same environment variable and needs no git fixture."
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
  - id: F19
    disposition: fixed
  - id: F20
    disposition: fixed
---

# Round 1 fixer report

All findings are fixed. Nothing is pushed.

## Changes

- F1: a re-entry whose record holds another gated head no longer pushes at the row's entry.
  The wait pushes that head after the gate passes. Both halt texts and `docs/overview.md` match.
  Existing rows cover the refused and the passing resume, with an exact entry-push count and a push-after-gate check.
- F2: `wake` judges a hand commit before a due gate.
- F3: the reconcile walk renames `on` and `merge` in board `run_settings`.
- F4: a new `agents-pr-reviewer-role-retired` reinterpretation drops the role from the hub `agents.yaml`.
- F5: the `on` notice and flag say what `review` no longer does.
- F6, F10, F11, F12: added the missing test cases.
- F7, F13: edited `_lym/landing/summary.md`. It is not committed by me, since `_lym` belongs to the loop.
  F13 took the finding's second option, an operator step in the upgrade notes.
- F8, F9, F14, F15, F16, F20: comment, doc and fixture corrections.
  F14 also drops the same row's trailing clause about the PR-Review row, and F15 a "like PR-Review's" session clause in `docs/overview.md`.
- F17: deleted the redundant test, with a `Loosened-Check:` trailer.
- F18, F19: added the keep markers.

## Gate repair

The gate refused an over-long comment line in `internal/shellguard/rows.go`, in the publish row that F14 changed.
It is now two shorter sentences, one per line.

## Reach note

The gate's reach note asked for any missing tests. I added none and changed no code.
In `.lym/gate/reach.json`, the one function listed with no reaching test is `loomCLI.wireLanding` (`internal/loomcli/sharedbootstrap.go`).
This round did not touch it: the branch removed the PR-Review session's wiring there before the review.
The file's own caveat says a test of `internal/loomcli` may run it unseen.

## Left open by F1

The row's own push now waits for the gate, but other pushes of the task branch do not.
`pullRequestRecordCommitter` (`internal/loomcli/landingdeps.go`) pushes both sides of the pair after a record commit,
so `lym loom pr-continue` on a halted run can still put a refused head on the pull request branch.
`PATTERN-pair-remote` has the code branch go along with run-record pushes, so the background pusher likely does the same; I did not trace that path.
Closing it is a design call outside this round. The texts claim only the row's push.

## Verification

`lym gate test --tier 0 --tier 1` over the branch diff, after the last edit: tier0 and the guards passed.
Tier1 reported one failure, the memory guard killing `cmd/lym`'s `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`.
That test was killed again on one rerun and passed on the next, run alone.
Other `cmd/lym` tier1 tests that the killed binary had not finished may be unreported in that run.
The run before the self-check edits passed tier0 and tier1 whole.

## Self-check

- fix-code-correctness, LOW, `landingForDecision` wording: fixed (with fix-proposal's second finding).
- fix-code-correctness, LOW, comments lost by `DropPRReviewerRole`: fixed; an emptied section now stays as an empty mapping.
- fix-consistency, LOW, the failed carry's warning: fixed.
- fix-consistency, LOW, `docs/overview.md` push sentence: fixed.
- fix-consistency, LOW, `PATTERN-config-layers` lacks the value rename: fixed.
- fix-affected-set, MEDIUM, run profiles still hold `on` or `merge`: fixed with the `profiles-pull-request-modes` reinterpretation.
- fix-affected-set, LOW, local `agents.yaml` holding the retired role: fixed; it is flagged, not rewritten.
- fix-affected-set, LOW, flat `pr_review_*` name untested in local and `run_settings`: fixed.
- fix-proposal, MEDIUM, other pushes still publish a refused head: fixed differently; the texts are narrowed to the row's push, and the other push paths are left as described under "Left open by F1".
- fix-proposal, LOW, F8 sentence reads backwards: fixed.
- fix-proposal, LOW, header comment lost by the F4 drop: fixed.

## Disputed

None.

## Deferred

None of the review's findings. See "Left open by F1".

## Rulings

None.
