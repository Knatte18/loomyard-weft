```yaml
lens: fix-affected-set
outcome: reported
```

**Lens fix-affected-set.** Task: check that the F2 and F3 fixes cover every member of their sets. F1, F4 and F5 have no set premise, so I left them unchecked.

**F2 (pair-push callers without `Here` or `RunSlug`):** covered. The non-test `PairPush{` / `PushPairAnchored(` sites are:
- `internal/fabriccli/weft_verbs.go:297` passes `Here`.
- `internal/pairteardown/handover.go:50` passes `RunSlug` and `Here`.
- `internal/beamcli/deps.go` `PushBeam` now goes through `pairteardown.PushRun`, which sets both.
- `internal/fabricengine/teardownlock.go:142` calls `CoalescePushRequest` with a `here` argument.

No caller is left with a zero `MachineNames` or an empty slug.

**F3 (landing-record sites):** covered. `recordLanding` is called from the two merge-success sites in `landUnderLock`. The only parent-reset path in `finalize.go` is `archiveHeldParentMerge`, reached from `pushParent`. That path is now followed by `recordLanding`. The merge-in-required retry also lands through `landed()` → `pushParent`, so it gets the re-record too.

**Findings:**

- **LOW** — `internal/beamcli/deps.go`, `PushBeam` → `pairteardown.PushRun(prime, beam, slug, pairteardown.RunPush{LockWait: …})`.
  - The new call leaves `InsidePair` at false.
  - `runRecordsPusher` in `internal/loomcli/landingdeps.go:~950`, the sibling caller of `PushRun`, sets `InsidePair: true` and `Parent`. With false, `HandOver` removes the local pair "unless push.InsidePair" (`internal/pairteardown/handover.go`, `HandOver` doc).
  - `internal/beamcli/land_integration_test.go:24` is named `TestLand_RunsFromATaskPair`, so a beam verb may run inside the pair it pushes. I did not confirm that `PushBeam` can be invoked from within `beam`. If it can, a takeover would remove the pair under the caller.
  - The change is correct only if the beam verbs always run from the prime or another pair.

I used 3 tool calls:
1. Grep for non-test `PairPush{` / `PushPairAnchored(` sites, to enumerate F2's caller set. The context held only the files the fixer read.
2. Grep for `ArchiveParentMerge`, `archiveHeldParentMerge` and `RecordLanding(` callers, to enumerate F3's set. The context held only `finalize.go` and `finalize_test.go`.
3. Grep for the other push entry points (`CoalescePushRequest(`, `PushPair(`, `.PushBeam(`), plus a read of `internal/loomcli/landingdeps.go:935-990`, to compare the sibling `PushRun` caller's `RunPush` fields. The context lacked that file.
