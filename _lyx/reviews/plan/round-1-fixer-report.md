# Plan round 1 — fixer report

## Changes

- F1 (MEDIUM, decision), fixed as proposed: card 2's Intent no longer lets a package argument that resolves outside the worktree root pass.
  Such an argument now falls under the card's existing missing rule, as D3 states.
  `TestValidate_VerifyPackageMissing`'s rows that report gain `go test ../outside` in the overview chain, with an empty card.
- F2 (MEDIUM, design), fixed as proposed: card 2's rows that report nothing gain `true | cd internal && go test ./nope`, `false || cd internal && go test ./nope` and `go -C .. test ./nope`.
  Each reaches one scan-end branch of `missingPackageArgs`' walk, and each would report `./nope` if the scan went on.
- F3 (LOW, consistency), fixed as proposed: card 4's read-error bullet now says that when the re-step is already used, the loop stops at once with `status-unreadable: <error>` and the read-error way forward, with no re-read.
  `TestLoop_InterruptedChildFillsTheErrorForm` gains the subtest "a read error after a used re-step stops without a re-read".

No target and no verify command changed.

## Disputed

None.

## Deferred

None.
