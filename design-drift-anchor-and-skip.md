# Drift runs in an anchored hub, can be skipped for one run, and its baseline agent picks scope off when the suite cannot reach the change

`drift-anchor-and-skip` · Task · bug, drift, stencil · running · Code-Burler-Fix

Three drift faults from one darn run (`testsupport-rule-and-helpers`) in hub NORCE-DrillingAndWells/Models, whose fabric is anchored at `src/csharp/NORCE.Models` by `.lym-anchor`; issues #23, #24 and #25.

## 1. The capture child starts at the worktree root, ignoring the anchor (#23)

Drift-Baseline failed three times with "the capture child ended before it recorded its start".
The capture-child log holds only: "cwd is outside the recorded fabric anchor subtree: cwd <worktree root> does not equal the anchored directory <worktree root>/src/csharp/NORCE.Models (recorded by .lym-anchor)".
The drift producer starts its capture child at the worktree root whatever directory the run is resumed from, so every re-launch fails the same way and drift cannot run in any hub with a subdirectory anchor.

Expected: the capture child, and the compare child, start in the anchored directory, as every other lym verb requires.

Workaround today: `lym drift defer` and `lym loom resume` in the task worktree, then `lym drift waive` at Drift-Verify.

## 2. No way to take one run past drift once Drift-Baseline failed (#24)

To skip drift for the one run, the parent ran `lym config drift --layer local --set enabled=false` in the task worktree; `lym config --resolved drift` showed `enabled: false # local`.
`lym loom resume` ran Drift-Baseline again and spawned the capture child anyway, failing the same way.
`lym drift defer` is refused at a failed or paused Drift-Baseline, and goto cannot move past it.

Expected: the local layer's `enabled=false` makes Drift-Baseline and Drift-Verify pass through on the next resume, or `lym drift defer` is admitted at a failed Drift-Baseline and named in its halt's way forward.

## 3. Drift-Baseline picks scope full for a change the suite cannot reach (#25)

The task added a PATTERN.md entry and moved three test helpers between test projects.
The agent's own `baseline.md` says it touches no production code and expects no drift, yet it recorded scope full.
The capture suite, `NORCE.Models.NumericalBaseline`, references only `NORCE.Models.WellboreSimulation`, so nothing the task changes reaches it; scope off was right and would have spared the run the capture.

Expected:
- The Drift-Baseline stencil tells the agent to check whether the capture command's projects reach the changed files through their project references, and to record scope off when they cannot or when the change touches no code the suite runs.
- The parent can reset a wrongly recorded scope: today a second `lym drift scope` is refused.
