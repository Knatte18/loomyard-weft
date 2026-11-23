---
dispositions:
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed-differently
    reason: "Carried out as proposed except that internal/gitrepo/doc.go goes under Edit, not Uses: doc.go's remote-refs lock line (internal/gitrepo/doc.go:44) enumerates the methods that run under the lock, so adding DeleteRemoteTrackingRef changes what it says. The self-check also showed the proposal alone leaves the next push of the pruned branch failing in checkOrigin, so card 1 additionally changes UpstreamSHA and checkOrigin (see Self-check)."
---

## Changes

- **F16 (fixed):** card 10 drops `internal/loomcli#intentionallyNil` from **Edit:**.
  The TeardownDeps bullet now keeps only the `nilLegal` addition and says why `intentionallyNil` stays as it is:
  `NilSeams` skips every `nilLegal` path (`internal/testkit/envkit/envkit.go`, `walkNil`),
  and `TestWire_EverySeamFilled`'s second loop fails for an `intentionallyNil` entry `NilSeams` never reports.
- **F17 (fixed differently):** card 1 gains the **Create:** target `plan:internal/gitrepo#Repo.DeleteRemoteTrackingRef`
  with the proposed signature, and `internal/gitrepo/doc.go` under **Edit:**, not **Uses:**.
  `internal/gitrepo/doc.go:44` lists the methods that run under the remote-refs lock, so the card has the line name the new method.
  The Intent now says `checkOrigin` deletes the ref through `DeleteRemoteTrackingRef`.
  That method removes only the refspec-mapped tracking ref under `ns`, in one hold of the remote-refs lock.
  It touches no config, and an absent ref or an unmapped remote is nil.
  The Intent gives why `DropRemoteTracking` is not used (`internal/gitrepo/push.go:215-221` unsets `.remote` and `.merge`).
  The two ended-branch rows of `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` now assert that `branch.<branch>.merge` stays set,
  and the card states that these rows reach the new method through the pair push.
  The ImpactSummary names the gitrepo change.

`lym loom validate-plan` reports `ok: true` with informational findings only, before and after the self-check changes.

## Self-check

- fix-consistency: no findings.
- fix-claims, MEDIUM, and fix-proposal, BLOCKING (merged as one finding): after the delete, the next push of that branch fails in `checkOrigin`,
  because `UpstreamSHA` returns a plain resolve error, not `ErrNoUpstream`, for a set upstream with an absent tracking ref
  (`internal/gitrepo/blobread.go:331-333`, `internal/fabricengine/pushclassified.go:626-630`).
  Card 10's "pushes with no error" would not hold on a second push.
  Fixed: card 1 edits `internal/gitrepo#Repo.UpstreamSHA` so that error also wraps `ErrRefNotFound`, with its text unchanged.
  `checkOrigin` reads it as a pruned ref and goes on to `awaitOrigin`, which skips a gone branch whose entry ended with nil.
  The code-branch row gains a second push after a new records commit that returns nil, with origin still lacking the code branch.
  The other `UpstreamSHA` callers are unchanged: `refuseUnseededBoard` tests only for `ErrNoUpstream`, and the later push steps run only once `checkOrigin` proceeds.
- fix-affected-set, LOW: with both seams in `nilLegal`, no test checks that `battenCLI.wire` fills `Teardown.Push` and `Teardown.Level`.
  Fixed: card 10 lists `internal/battencli#TestWire_BuildsLazilyForANonexistentTaskWorktree` under **Edit:**
  and gives it `TeardownPush` and `TeardownLevel` rows beside its existing `TeardownShutdown` and `TeardownRemove` rows (`internal/battencli/wire_test.go:64-65`).

## Disputed

None.

## Deferred

None.

## Rulings

None.
