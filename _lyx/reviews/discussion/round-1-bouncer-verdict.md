---
verdict: APPROVED
rationale: "The reviewer approved, and the fixer's edits for F1-F7 are present in decision-record.md: the marker is written after Spawn succeeds, with a consumed-approval reason; the archive tag is pushed before Remove's first mutation, force is not a bypass, and Add's rollback is excluded; the done-wait is exempt; the dirty-weft recovery is lyx loom commit-records; the archive siblings are staged; and the in-flight loss is accepted. The one wording ambiguity left is pinned by an acceptance criterion and does not block."
---

## Round 1 judgment

The round-1 report returned `APPROVED` with four MEDIUM and three LOW findings.
The fixer report says all seven were fixed with nothing deferred.
Each fix is present in the current `decision-record.md`:

- **F1:** the resume Decision writes the marker only after `Spawn` succeeds, and a `Spawn` error stays the hard error.
  A consumed approval produces its own Stuck reason, which names re-running `lyx loom approve`.
  Acceptance criteria cover the spawn-once, spawn-error and new-approval cases.
- **F2:** the archive Decision places `Remove`'s tag push before its first mutation.
  It states that the push runs whatever `remote` says, that `force` does not bypass it, and that an unreachable origin aborts.
  The Open risk "Offline pair removal" records the cost.
- **F3:** the done-wait Stuck is budget-exempt.
  An acceptance criterion pins that it neither consumes the running budget nor is blocked by it.
- **F4:** the done-wait Decision states the resulting state: batten is blocked at `Worktree-Teardown`, and the pair is intact apart from the portal and launchers.
  It names `lyx loom commit-records` followed by a batten resume as the recovery, never `--force`, and an acceptance criterion requires that reason.
- **F5:** the status-commit Decision and its acceptance criterion cover the friction archive siblings.
- **F6:** the archive Decision is scoped to an existing pair and excludes `Add`'s rollback by name.
  An acceptance criterion pins the exclusion.
- **F7:** the in-flight Open risk states the silent loss and why it is accepted.
  The duplicate auto-mode assumption is gone.

The report has no `## Focus departures` section, so there is nothing to ratify or reject.
Its focus-4 relocation pass applied the completeness-before-leanness test correctly.
The `start.go` and liveness-key assumptions carry plan requirements and are correctly framed as items to verify, so they stay where they are.
No rejected alternative has leaked into `decision-record.md`.

## Non-blocking observation

The archive Decision says `Remove` pushes the tag "after its refusals and before its first mutation".
In `Remove` today, the dirty-warp and dirty-weft refusals run after `removePortal` and `removeLaunchers`, which are its first mutations.
Read literally, the phrase cannot place the tag.
The acceptance criterion settles the order: a failed push must leave the portal, launchers, worktrees and both branches in place.
That puts the tag after the merge refusals and before `removePortal`.
On the done-wait's dirty-weft path, that order also means a tag gets pushed for the pre-commit tip before `Remove` refuses.
The retry after `lyx loom commit-records` then pushes a second tag for the new tip.
This is harmless, since tag names are derived from the tip SHA, and it falls under the existing "Archive tags accumulate" risk.
The plan writer can resolve this from the acceptance criterion, so it does not justify another round.
