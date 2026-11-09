```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1–F6, F8, F9) and build the case where each is wrong. F7 was disputed, so it is out of scope.

Findings:

1. **LOW** — `internal/fabricengine/replay.go:250` and `internal/fabricengine/doc.go:1761` (F8, F9 proposals)
   - **Summary:** the F8/F9 fixes cover only `docs/overview.md:1302` and `raddle-recipe.yaml:4`. Two siblings still say Raddle-File ends the run's own entry done.
   - **Evidence:**
     - `replay.go:250` reads "that row ends the run's own entry, so an entry origin shows done is the run's own finish".
     - `doc.go:1761` reads "That row ends the run's own entry, so a filed run goes on through Land and Worktree-Teardown".
     - After an arras family filing the entry is a project with no status, never done. The `finalizeEntered` reasoning about origin showing it done therefore does not hold for that run.
   - **Caveat:** whether the reconcile code itself mishandles a family run is outside this change set. I did not check it. These two comments are stale by the same wording the review flagged.

2. No finding on the other proposals. I built a case for each and none held:
   - **F1:** `repointDependents` skips a done dependent and re-points an open one onto the sink part (`plan-b`). The new tasks-shape row asserts both.
   - **F2:** `repaired` is true only for a family filing found as its own, so clearing `entry.StatusProducer` affects only the store-only filing. A fresh filing keeps its claim. Clearing it does not break a later use of `entry` in `Call`.
   - **F3:** `arrasProjectOf` clears profile, run settings and notify-on-land, so the real `FileRaddle` conversion still passes the stricter lock. The empty-but-non-nil `RunSettings` case is covered by `len() > 0`.
   - **F4:** the integration test would fail if the wrapper were applied to every recipe (tapestry-marked row), or if the mark were read under the wrong run id (arras-marked row).
   - **F5:** the three null-status writes each follow a real run-status write, so each is a real change.
   - **F6:** the "no added part" row can only be refused by the `len(parts) > 0` check, because the derived project otherwise equals `now`. The other new lock rows hit their own branches.

Calls made: none. My context already held every changed file.
