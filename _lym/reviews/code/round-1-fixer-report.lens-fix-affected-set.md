```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the sets the fix had to cover, namely the seam-filled check, the tag callers and the wire path.

No findings.

- **F5, seam-filled check:** the `RunNoticeDeps` seam fields are `ReadChild`, `DriverAlive`, `Agents`, `NewestAgentEnd`, `Pause`, `ReviveAPIError`, `MarkerPath` and `Post`. The check at `dobbydeps_integration_test.go:115` now tests all eight. `Quiet` is a value, and the next line pins it.
- **F3, tag callers:** the non-test `PostTags{` sites are:
  - Tagged: `beamcli` `runNoticeTags`, `NotifyParent` (`pair.WorktreeName`), `battencli` `runNotifyTags`, and `pushnotifier.go:43`.
  - Untagged by design: `NotifyPrime`, `NotifyFrom`, `NotifyFromScoped`, `NotifyAsker` and `NotifyOrchAt`.
  - `pushnotifier.go:43` is tested through the daemon in `TestCreelNoticeTags`. `NotifyParent` is tested end to end by the `NotifyParentMailIsDroppedWithThePairsSlug` subtest.
  - The `battencli` and `beamcli` closures are tested only through their extracted helpers (`runNotifyTags`, `runNoticeTags`). A wrong tag at the call sites in `notifyRunParent` or the dobby closure would go unnoticed, but that is the extraction route the review allowed.
- **F2, wire path:** `server.go:306` is the only place a post's Topic and Run are mapped, and `TestCreelNoticeTags` reaches it through `GoClient.PostTagged`. The `DropRun` path through `server.go:410` and `GoClient.DropRun` is reached by the `dropRunMail` subtests.
- **Not checked:** F1, F6, F7 and F8 are not set-premise findings.

Calls made:
1. Searched `RunNoticeDeps` fields in `internal/dobbyengine/runnotice.go`, because my context lacked the full seam list.
2. Searched `PostTags{` across non-test code, to confirm every tag site is covered.
