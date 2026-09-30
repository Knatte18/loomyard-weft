# Support log — fabric-readd-weft-push

## Interview

Autonomous session: no operator answered; every decision below is a self-pick recorded in the question ledger.

1. **Is the bug still live?**
   Commit `81872afb5` already deflaked `TestRemove_AddSucceedsAfterWarpBranchDeleted` by removing with `remote=true`, and its comment points at this board task.
   The product gap remains: a plain Remove leaves `<slug>-weft` on origin, and a re-Add's freshly bootstrapped weft branch diverges from it unless both bootstrap commits land in the same second.
2. **What distinguishes an orphan from a live branch elsewhere?**
   Remove pushes `archive/<slug>/<tip>` to the weft origin before any teardown.
   A remote weft tip reachable from such a tag is provably preserved, so replacing the branch loses nothing; an unarchived tip may be a live pair on another hub.
   This makes "Remove records it" unnecessary as new state — the archive tag already is the record.
3. **Adopt, reset, or refuse?**
   Adopting would import the old run's `_lyx` state into a fresh task.
   Chosen: replace when archived, refuse otherwise.
4. **How is the replacement made safe?**
   It is a destructive remote primitive, so it goes through `destroy.go`'s pipeline as a remote-branch deletion leased to the SHA the probe saw, followed by the existing plain push.
5. **Does the warp side have the same gap?**
   Fabric never deletes remote warp branches.
   A re-add after pushed-and-squash-landed warp work fails at the warp push after the whole pair is built, and the rollback cannot delete a bare-slug warp branch under the default empty prefix, leaving a local branch that blocks retries.
   Chosen: a pre-flight fast-forward check that refuses with a remedy; no warp deletion.
6. **Testing.**
   Integration tests force divergence deterministically with an extra pushed weft commit, rather than relying on timestamps; the refusal, lease-race, warp, and `SkipPush` cases each get a test; `destroy.go` unit tests cover the new request shape.

## Rejected alternatives

- **Adopt the remote weft branch** (fetch it, create the local branch from it): carries the removed run's shed status, discussion, plan and origin record (old parent) into the new task, which would mislead the loom run.
- **Unconditional force-push of the new weft branch**: overwrites a live pair's weft on another hub with no proof its content is preserved; violates the chokepoint's ownership/dirtiness checks.
- **Make plain Remove delete the remote weft branch by default**: reverses the deliberate opt-in policy for remote deletion (`--remote`), and does not help hubs whose leftovers already exist.
- **Remove writes a new "removed slugs" record that Add consults**: duplicates what the archive tag already proves, and adds durable state with its own lifecycle.
- **Refuse on every leftover, even archived ones**: correct but leaves the common re-add broken in real use, which is the product gap the task names.
- **Delete the archived leftover at pre-flight**: touches the remote even when Add later fails for an unrelated reason; deleting right before the push keeps a failed Add remote-neutral.
- **A single atomic `--force-with-lease` push instead of delete-then-push**: needs a new mutation Kind (`remote_branch_replaced`) in the closed enum, or would misrecord one primitive as two; delete-with-lease then push reuses existing Kinds and the existing remote-deletion executor.
- **Deleting a diverged remote warp branch**: fabric's warp side has no archive, and remote warp deletion was removed from fabric on purpose.
- **Fixing the webster integration report's missing test name here**: different module and a different defect; belongs on its own board task.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): Adopt, replace, or refuse a leftover remote weft branch? → Replace when its tip is reachable from an `archive/<slug>/*` tag on origin, refuse otherwise. Rationale: the archive proves no data loss; an unarchived tip may be live elsewhere.
- Q2 (auto-pick, resolved): Does Remove need to record anything new? → No; the existing archive tag is the record.
- Q3 (auto-pick, resolved): Replacement mechanism? → Gated remote deletion in `destroy.go` leased to the observed SHA, then the existing weft push. Rationale: chokepoint invariant, closed `Kind` enum unchanged, race-safe.
- Q4 (auto-pick, resolved): When does the deletion run? → Immediately before the weft push; the probe and refusal run at pre-flight.
- Q5 (auto-pick, resolved): Include the warp side? → Yes, as a pre-flight refusal only. Rationale: same verb, same re-add path, and today's failure strands a local warp branch that blocks retries.
- Q6 (auto-pick, resolved): Behaviour under `SkipPush`/`SkipGit`? → No probes, no replacement.
- Q7 (auto-pick, resolved): Rollback after a completed replacement? → Do not restore the old remote branch; the archive tag holds its content.
- Q8 (auto-pick, resolved): Existing test's `remote=true` workaround? → Revert to a plain Remove and add a deterministic-divergence test alongside it.
- Q9 (auto-pick, resolved): Webster integration report lacks the failing test name → out of scope, separate task.
- Q10 (auto-pick, resolved): Adopt path (local weft branch already exists) → unchanged; assumed to already match or fast-forward its remote copy.
