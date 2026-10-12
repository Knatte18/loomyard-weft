---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/12-landing-merge-in-producer.md
    summary: syncParent returns the upstream tip ReadParentTips read, not FastForwardResult.NewHead, so an upstream that advances between the two fetches records a merged_tip older than what was merged and leaves the gap unreviewed
    fix: In card 12 step 3, replace "After a fast-forward it returns the upstream tip." with "After a fast-forward it returns `FastForwardResult.NewHead`, the tip the pair was moved to, which can be newer than the `Upstream` that `ReadParentTips(true)` read." In the `mergein_test.go` bullet list, extend the behind row so the fake puller returns a `NewHead` that differs from the fake `ReadParentTips` upstream, and assert the merge record's `merged_tip` equals that `NewHead`.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-fabric-parent-reads.md
    summary: FastForwardPair's weft step says only that it calls PullWeft as Pull does, but Pull probes the weft upstream first, treats a weft pull failure as non-fatal and records the weft advance, and the card states none of this, so a parent weft with no upstream fails Merge-In after the warp has already moved
    fix: In card 7's FastForwardPair bullet, replace "It pulls the weft with `PullWeft(opts)` as `Pull` does, so `opts.SkipGit` gates the weft side as it does in `Pull`." with "It pulls the weft first, as `Pull`'s weft arm does. A weft with no upstream (`weftHasUpstream` false) is skipped. A `PullWeft(opts)` failure is logged as a Warn and does not stop the warp side. A weft HEAD that moved, sampled before and after with `weftSHAOrEmpty`, appends `KindRepoAdvanced` at the weft path to the record. `opts.SkipGit` skips the weft side." Add `internal/fabricengine#Fabric.weftHasUpstream` and `internal/fabricengine#weftSHAOrEmpty` to the card's Uses. In the test paragraph, add a case asserting that a fast-forward over a parent pair whose weft has no upstream moves the warp with a nil error.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/22-status-lock-wait.md
    summary: The status-spec text card 22 prescribes says the lock wait record is written by the step waiting at Merge-In, but cards 11 and 20 have Publish, PR-Gate and Finalize write it through the shared guard too
    fix: In card 22's spec paragraph, replace "which the step waiting at Merge-In writes" with "which a landing row writes while it waits for the parent lock".
---

### [MEDIUM] Merge-In records the pre-fetch upstream tip as merged_tip after a fast-forward

**Location:** `_lyx/plan/12-landing-merge-in-producer.md`, step 3.

**Issue:**
Card 7 has `FastForwardPair` fetch the code side again and reset to the upstream that its own fetch returned, and it reports that SHA in `NewHead`.
Card 12's `syncParent` reads only `FastForwardResult.Refusal` and then "returns the upstream tip", which is the `Upstream` that `ReadParentTips(true)` read before the second fetch.
Suppose the upstream advances, still as a fast-forward, between the two fetches.
The pair then moves to the newer tip, and step 5 merges the local parent branch at that newer tip.
But step 6 writes the older tip as `merged_tip`, and the overlap is computed over `reviewed_tip..<older tip>`, so the Merge-Review never sees the newer commits.
At Publish, `moved` sees a local tip that differs from `merged_tip` and reroutes.
The next Merge-In finds the parent equal, so its merge is up to date.
`SkipsReview` is true for an up-to-date merge, so Merge-Bouncer skips, and its approve seam records the newer tip as reviewed.
The commits between the two fetches are merged and landed but never reviewed, which breaks D1's "merges the local parent branch … and records that tip" and the goal that the merge review covers what the parent gained.
This is the second focus question: card 12 reads card 7's `Refusal` but not its `NewHead`.

### [MEDIUM] FastForwardPair's weft step leaves upstream, failure and recording semantics unstated

**Location:** `_lyx/plan/07-fabric-parent-reads.md`, the `FastForwardPair` bullet.

**Issue:**
The card says `FastForwardPair` "pulls the weft with `PullWeft(opts)` as `Pull` does".
In `internal/fabricengine/pull.go`, `Pull` wraps that call in three guards the card does not state:
- it probes `weftHasUpstream` and skips the weft when there is no upstream, because "the freshly bootstrapped hub's suffixed primary exists only locally until the first push lands";
- it warns past a `PullWeft` failure and still runs the warp side;
- it samples the weft SHA before and after, and appends `KindRepoAdvanced` at the weft path, which its comment calls load-bearing for the record.

`PullWeft` itself (`internal/fabricengine/recordsgit.go:338`) only calls `gitrepo.Repo.Pull`, which runs `git pull --ff-only` and fails when the branch has no tracking ref.
The card's own sentence "Any other failure is returned as the error" therefore turns a weft with no upstream into a Merge-In error.
Because the card orders the weft pull after `ResetHard`, the warp has already moved by the time that error returns.
The card's test asserts only "the reset entry" in the record, so an unrecorded weft advance would pass.
`PATTERN-mutation-record` requires the result's record to reflect what the verb changed.
The first focus question asks whether the signature supplies `SyncOptions`.
It does, but the weft half of the verb is under-specified.

### [LOW] Status-spec text names Merge-In as the only lock-wait writer

**Location:** `_lyx/plan/22-status-lock-wait.md`, the `loom-status-spec.md` paragraph.

**Issue:**
Card 11's `parentGuard.ensure` writes the wait record through `Waiter.OnWait`.
Card 20 calls `ensure` from Publish, PR-Gate and Finalize as well as Merge-In.
The spec sentence card 22 prescribes ("which the step waiting at Merge-In writes") would therefore be wrong for three of the four lock-taking rows.

## Focus verdicts

- Card 7, `Fabric.FastForwardPair`: not a defect on the signature or result.
  `FastForwardPair(opts SyncOptions) (res FastForwardResult, err error)` takes the `SyncOptions` that `PullWeft` needs.
  `FastForwardResult` embeds `MutationRecord`, filled from `NewMutations` and returned even on a refusal, which satisfies `PATTERN-mutation-record`'s embed rule.
  The refusal path is reached by named tests: dirty, merge in progress and a force-pushed upstream in `parenttips_integration_test.go`, and each Merge-In Stuck in `mergein_test.go`.
  The weft half is F2.
- Card 12, `parentPuller` and `syncParent`: `parentPuller` declares the card 7 signature.
  Merge-In passes `fabricengine.SyncOptions{SkipPush: m.deps.PushSkipped}`, matching Finalize's `PushBranch` call at `internal/landingshed/finalize.go:280`, and reads `Refusal`.
  Not reading `NewHead` is F1.
- Card 22: `contracts/specs/loom-status-spec.md` is on card 22's Edit list.
  The card places the lock wait inside `gateWaiting`, ahead of `verifyWaiting`, which matches `reviewWaitingFor` at `internal/loomcli/reviewstatus.go:173-174`, where `gateWaiting` wraps `verify`.
  It keeps `gateWaiting` outermost and leaves the spec's gate-wait sentence unchanged.
  The writer wording is F3.
