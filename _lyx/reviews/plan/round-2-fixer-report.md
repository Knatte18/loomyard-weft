# Burler plan round 2 — fixer report

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  Premise checked: `Fabric.Pull` builds its record with `NewMutations`, passes it to `ResetHard(rec, upstreamSHA)`, calls `recordWarpAdvance`, and returns it in `PullResult`, which embeds `MutationRecord` (`internal/fabricengine/pull.go`); `PullWeft` takes `SyncOptions` (`internal/fabricengine/recordsgit.go:338`).
  - Card 7: `Fabric.FastForwardPair` is re-signed to `func (f *Fabric) FastForwardPair(opts SyncOptions) (res FastForwardResult, err error)`, and a Create glyph adds `FastForwardResult`, which embeds `MutationRecord` and carries `Refusal FastForwardRefusal` and `NewHead string`.
    The Intent states the record is accumulated with `NewMutations`, passed to `ResetHard(rec, upstreamSHA)`, the advance recorded with `recordWarpAdvance`, `opts` passed to `PullWeft(opts)`, and the record returned even on a refusal, where it is empty.
    The Uses list gains `recordWarpAdvance`, `NewMutations`, `MutationRecord` and `SyncOptions`.
    The integration test asserts a fast-forward's record holds the reset entry and `NewHead` is the upstream SHA, and that each refusal's record is empty.
  - Card 12: `parentPuller` declares `FastForwardPair(opts fabricengine.SyncOptions) (fabricengine.FastForwardResult, error)`; `syncParent` reads `FastForwardResult.Refusal`, and Merge-In passes `fabricengine.SyncOptions{SkipPush: m.deps.PushSkipped}`, the form Finalize's `PushBranch` call uses.
    The Uses list gains `FastForwardResult`.
- **F2 (MEDIUM), fixed as proposed.**
  Premise checked: `contracts/specs/loom-status-spec.md` names four waits, and `reviewWaitingFor` makes `gateWaiting` the outermost wrapper (`internal/loomcli/reviewstatus.go:173`).
  - Card 22 adds `contracts/specs/loom-status-spec.md` to its Edit list.
  - Of the fix key's two options, `parentLockWaiting` goes inside `gateWaiting`, ahead of the verify note, so the spec's gate-wait sentence stays true unchanged.
  - The Intent has the spec's `## The waiting key` count five waits, list the parent lock wait second with its render form, state that a dead writer's lock wait record is ignored, and name the lock wait record among the read sources.

## Disputed

None.

## Deferred

None.
