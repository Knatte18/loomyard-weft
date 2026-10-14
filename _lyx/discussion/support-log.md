# Support log — run-window-early

## Interview

A darn run holds no interview.
The parent, `ly:orch`, sent a scope correction before any code was written, on the operator's word: grow the task to three changes (early window, detached seed push, detached pair-creation pushes).
The writer asked the parent to confirm it, since it first arrived as pasted text; the parent confirmed and asked for it to be recorded as a parent decision.

## Rejected alternatives

- Spawning the detached push inside `Topology.Add`: every in-process caller, test fixtures included, would fork the test binary; the push moves to the CLI layers instead.
- Moving the terminal open wholly from Run-Shed to Worktree-Create: a run resumed past Worktree-Create with `--window` would never open its window.
- Opening the window before `Add` returns, mid-transaction: `Add` is one transaction with rollback, and the pair's config is reconciled only after it, which the reed session boot reads.
- Keeping `AddResult.Pushed`: `Add` no longer pushes, so the field would always be false.

## Review rounds

_No rounds yet._

## Question ledger

- Is the pasted scope correction the parent's? Resolved: confirmed by `ly:orch`.
- Does the detached push create the upstream of a never-pushed branch? Resolved (self-pick from the code): `gitrepo.PushRebaseFree` runs with `push.autoSetupRemote=true`.
- Does the detached push retry later when it fails? Resolved (self-pick from the code): every fabric commit and sync spawns a push of both sides, and a branch with no upstream counts as unpushed.
- Keep the archived-leftover records branch deletion synchronous? Resolved (self-pick): yes, a detached push onto the leftover would be rejected.
