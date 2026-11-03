```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: 1 finding.

1. **LOW**, header of `internal/ideengine/spawn_hub_integration_test.go`, lines 1-2.
   - **Summary:** The file header lists what the checks cover and omits the new degrade check. It says "the prime opens the lym-generated hub workspace, a beam its own workspace, a task slug still opens its bare folder, and the window command follows a launch".
   - **Evidence:** `checkBeamWithoutReadableOriginOpensBareFolder` is a new top-level check in the file. It asserts that a beam with an unreadable or absent origin record opens its bare folder. The header was not touched by commit `0d030e79`. A reader choosing this file for that behavior gets no hint.

The rest of the change set agrees with its surroundings:
- The `TestSpawnScenario` doc comment says the prime-touching steps follow the task-pair steps. The new step sits after the beam step and before the prime steps, so that holds.
- The new check restores the lock directory. The marked pair left behind has no origin record, so the later hub-workspace regeneration lists it as a task, as `checkPrimeWritesHubWorkspace` expects.
- The `.weft` string literal matches existing test precedent in `fabricengine/add_hubreader_integration_test.go:45,124`. `weftLockDirName` is unexported in `fabricengine`, so the literal is the only option from `ideengine`.
- `TestSpawnAnchoredScenario` carries no beam steps, so the new step's absence there is not a twin left behind.
- The `Spawn` and `isBeam` doc comments describe the degrade as the new rows assert it.

Further tool calls:
1. `git show HEAD --stat`, the scenario header and a grep for `.weft` literals, in one Bash call: my context lacked the scenario file's header and any other uses of the lock-directory name.
