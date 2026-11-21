```yaml
lens: guard-reach
outcome: reported
```

Task: the guard-reach lens. For each guard, refusal, branch or test row a card adds, I traced whether a real entry point can reach it and whether the guarded site recurs unguarded.

**Per-guard reachability**
- Card 1, `remoteTrackingRef`, a merge ref that no refspec maps answers no upstream: reachable through the new `TestUpstreamSHA` row.
- Card 1, `remoteTrackingRef`, the no-refspec fallback: reachable through the `--unset-all` row. A configured store never reaches it in production; it only keeps today's answer.
- Card 1, `RemoteFetchRefspecs` returning `ErrRemoteNotConfigured`: reachable through a `TestRemoteFetchRefspecs` row.
- Card 2, `originNamespaceAt`, every branch (missing path, no `.git`, no origin cached, a failed read not cached): reachable, each through a `TestOriginNamespaceAt` row.
- Card 3, `UpdateRemoteBranchLeased`:
  - An empty `expectSHA` as a lease on absence: reachable through the new rows.
  - The loosened check does not recur on any other path. `moveRemoteTaskBranch` returns before the push when the tip is empty (destroy.go:2438). `pushOwedCodeTip` moves only when origin holds the branch (handover.go:257-283).
  - A non-empty invalid SHA still answering `ErrInvalidSHA`: reachable through the existing row (push_test.go:407-413).
- Card 3, the `DeleteRemoteBranch` `ls-remote` probe: reachable only through `TestDeleteRemoteBranch`. In production its state cannot occur; see finding 3.
- Card 3, `namesRef` under a namespace: reachable through the `TestRetryablePushRefusal` row.
- Card 5, offline refusal of `SingleRepo` with `WarpURL` set: reachable through card 6's tier0 row. `runCloneWithReset` passes straight to `CloneAndWire` and then `CloneHub`, and `CloneHub` spawns nothing before its first check (fabriccli/clone.go:217-254, clone.go:130-140).
- Card 5, `SingleRepo` with no `--shortname`: reachable through a `TestCloneHub_OneURLRefusals` row.
- Card 5, a failed `readOriginLayout` wrapped by `wrapProbeError`: reachable through `TestCloneHub_UnreachableWeftIsHardError/OneArgument` (warpbinding_clone_integration_test.go:103-127).
- Card 5, refusals 1–4 (`lym-bolt` with and without the flag, the board ref, `.lym-warp`, `.lym-anchor` or `_lym`): each reachable through its row. The ordering of refusals 3 and 4 is reached through the row that holds both `.lym-warp` and `.lym-anchor`.
- Card 5, join binding mismatch and shortname mismatch: reachable through card 7's rows.
- Card 5, join with `--reset` and an explicit `--subpath`: reachable through card 7's row. The pre-reset check's fallback reads a different source from the clone; see finding 4.
- Card 5, seed records lease, board lease, refused board push, restore by update, restore by delete, and failed restore: each reachable through a card 7 hook or `pre-receive` row.
- Card 5, the refused-push classification excluding `(failed)` and `cannot lock ref`: no row reaches it; see finding 6.
- Card 5, the seed lease keyed by the code clone's branch "so an origin whose HEAD names no branch still leases": the state cannot occur; see finding 5.
- Cards 5 and 6, `AbandonSeededClone`: no test reaches it (finding 2), and the site it guards recurs unguarded (finding 1).
- Card 8, `primaryWeftBranch` with the key unset: reachable through its row.
- Card 8, a failed layout or config read: reachable through its row. Depending on the process cache, the unparsable-config row reaches either card 2's read error or the `git config` read error. Both refuse with the error named.
- Card 9, `TestCleanup_NeverPushedOrphanIsIdempotentOnRemote` over a one-repository hub: it reaches the caller's empty-lease skip, never the namespaced read or the probe; see finding 3.

**Findings**

1. MEDIUM · design · `_lym/plan/06-clone-single-repo-flag.md` (Intent, `CloneAndWire` failure routing).
   - `AbandonSeededClone` covers only errors that return. A `--single-repo` clone interrupted between `CloneHub` and `PushCloneBoard` leaves a hub with an unseeded board.
   - The documented recovery is `lym fabric reconcile` (fabriccli/clone.go:43). It reaches `Bolt.PushRecorded`, whose push is unleased; on rejection it replays the local board onto origin's (bolt.go:131-146).
   - So the guarded site, an unleased push of an unseeded board, recurs on the reconcile path. A `main` board that another hub seeded in the meantime gets this hub's commits replayed onto it. D3's bound is "its board push lands only while origin still holds no board branch".
   - The card names no guard on that path, for example the board push refusing, or seeding through `PushCloneBoard`, when the records store reads as one repository and the board has no upstream.
   - Evidence: internal/fabriccli/clone.go:43, internal/fabricengine/bolt.go:131-146, card 6 lines 31-33.

2. MEDIUM · design · `_lym/plan/05-clone-one-repository.md` (`AbandonSeededClone`).
   - The card plans no test for `AbandonSeededClone` and cites refusal-spec's raw I/O exemption.
   - That exemption covers a transient raw I/O failure propagated with its context (refusal-spec.md:47). `AbandonSeededClone` removes the hub and rewrites the cause with a way forward, so it is new behaviour, not propagation.
   - Its causes include `Bolt.Commit`'s secret-path refusal (PATTERN-fabric-boundary, "Weft commits"), which is not raw I/O.
   - An input reaches it: run `CloneHub` with `SingleRepo` over a bare, then call `AbandonSeededClone(res, rec, cause)`. The test asserts that no hub directory remains and that the error names "the same clone again".
   - Evidence: card 5 lines 127-131, contracts/specs/refusal-spec.md:47.

3. MEDIUM · design · `_lym/plan/09-pair-lifecycle-namespace.md` (the `TestCleanup_NeverPushedOrphanIsIdempotentOnRemote` one-repository row) and `_lym/plan/03-namespaced-push-ops.md` (the `DeleteRemoteBranch` probe).
   - Every production `deleteRemoteBranch` caller passes a lease: handover.go:332, cleanupremotewarp.go:195, add.go:735, remove.go:693, taskbranchdiscard.go:151.
   - Cleanup returns before the delete when `leaseSHA` is empty (cleanup.go:427-429). The probe's guarded state therefore cannot occur in production.
   - The one-repository row passes through that skip. An un-namespaced tip read of `refs/heads/&lt;b&gt;-weft` on the one bare is absent too, so the row also passes when the namespace is missing, and it does not pin D4.
   - Fix: seed a code branch `refs/heads/&lt;b&gt;-weft` on the bare in that row. Assert that it survives and that no `remote_branch_deleted` entry is recorded.
   - Evidence: internal/fabricengine/cleanup.go:426-429, internal/fabricengine/destroy.go:1617-1634.

4. LOW · consistency · `_lym/plan/05-clone-one-repository.md` (the join's `--reset` subpath check).
   - In a join, `checkSubpathBeforeReset` stages a weft clone of the one URL, which is the code default branch.
   - When `refs/lym/heads/&lt;branch&gt;-weft` is absent, `recordedAnchorOnWeft` falls back to that HEAD and to its "on weft:main" legacy board marker (resetsubpathcheck.go:85-101). The clone itself adopts from the board on the namespaced `main` (clone.go:337-351).
   - The check before the reset and the clone after it read different sources, so a subpath the check passes can be refused once the old hub is gone. The card names only the namespaced fetch.
   - Evidence: internal/fabricengine/resetsubpathcheck.go:85-101, internal/fabricengine/clone.go:337-351.

5. NIT · consistency · `_lym/plan/05-clone-one-repository.md` (Seed, the observed-tip lookup).
   - The stated reason, "an origin whose HEAD names no branch still leases the ref the seed pushes", describes a state that never reaches the seed push.
   - Such an origin's code clone is unborn, and `refuseUncheckedOutWarpClone` tears the hub down first (clone.go:571; `TestCloneHub_RefusesAWarpRemoteWhoseHeadNamesANonexistentBranch`).
   - Evidence: internal/fabricengine/clone.go:255-257 and 571.

6. LOW · design · `_lym/plan/05-clone-one-repository.md` (the cause classification of `pushCloneSeed`).
   - The branch that sends a `(failed)` or `cannot lock ref` `[remote rejected]` to "run the same clone again", rather than to the refused-host way forward, has no reaching row in card 7.
   - Card 7's rows reach a hook decline, lost leases and a failed restore only.
   - Evidence: card 5 lines 122-125, card 7 lines 58-69.

Tool calls:
- internal/fabriccli/clone.go:217-290 and warpbinding_clone_integration_test.go:96-156: my context lacked `runCloneWithReset`'s body (needed to know whether card 6's tier0 row spawns before the refusal) and the unreachable-weft test rows.
- grep of the `deleteRemoteBranch` callers and the `RecordsBranchName` uses: needed to know whether any production caller passes an empty lease, and where else the prime's records branch is derived.
- internal/fabricengine/cleanup.go:405-445: needed `deleteWeftBranchOnRemote`'s handling of an empty lease, to trace card 9's row.
