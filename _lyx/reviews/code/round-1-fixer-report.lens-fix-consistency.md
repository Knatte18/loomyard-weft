```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: check whether the F1–F3 change set still agrees with its siblings and twins.

Findings:

- **LOW** — `internal/fabricengine/junctionnames.go:171` (`AdoptScratchDir`): no test pins the new accessor.
  - Sibling hub-scratch derivations `HubLogsDir` and `LogArchiveDir` are pinned in `internal/fabricengine/hubscratch_test.go`, which asserts `HubLogsDir(hub) == Join(HubScratchDir(hub), "logs")`.
  - `internal/adoptcli/deps_test.go:61` builds its own `logPath` and never goes through the accessor.
  - Weak finding: the other precedent, `failedPushDir`, is unexported and has no direct test either.
  - Not a stale text or cross-reference.

Checked and consistent:
- `doc.go:61`: "After the resumes" now fits the `# Step 6` section that follows.
- `doc.go:69` and `deps.go:116` both name `fabricengine.AdoptScratchDir`.
- The `lyxdirs.AdoptDirName` doc comment in `internal/lyxdirs/dirs.go:34` still matches the accessor.
- `AdoptDirName` has no remaining non-test caller outside the accessor.
- No other file mentions "Last, after the resumes".
- The new test row's setup mirrors the existing `OwnerHaltedChildResumesWhenThePrimeIsAdopted` row.

Further tool calls:
1. `grep` for `AdoptDirName`/`AdoptScratchDir`/`FailedPushDirName` and for `Last` in `adoptcli`: to find other users and twins of the changed names, which my context lacked.
