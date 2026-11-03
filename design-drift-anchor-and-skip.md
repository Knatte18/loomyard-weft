# drift: capture child spawns at the worktree root, ignoring a subdirectory fabric anchor

`drift-anchor-and-skip` · Note · bug, drift, stencil

Imported from [issue #23](https://github.com/Knatte18/loomyard/issues/23).

Seen in hub NORCE-DrillingAndWells/Models, whose fabric is anchored at src/csharp/NORCE.Models by .lym-anchor. A darn run (testsupport-rule-and-helpers) failed Drift-Baseline three times with 'the capture child ended before it recorded its start'. The capture-child log holds only: cwd is outside the recorded fabric anchor subtree: cwd <worktree root> does not equal the anchored directory <worktree root>/src/csharp/NORCE.Models (recorded by .lym-anchor). The drift producer starts its capture child with cwd at the worktree root whatever directory the run is resumed from, so every re-launch fails identically and drift cannot run in any hub with a subdirectory anchor.

Workaround: the parent runs lym drift defer and lym loom resume in the task worktree, then lym drift waive at Drift-Verify, so the run goes on with no baseline.

Expected: the capture child (and the compare child) start in the anchored directory, the same directory every other lym verb requires.

## From issue #24

Imported from [issue #24](https://github.com/Knatte18/loomyard/issues/24).

Hub NORCE-DrillingAndWells/Models, darn run testsupport-rule-and-helpers, halted at Drift-Baseline by Knatte18/loomyard#23. To skip drift for this one run, the parent ran lym config drift --layer local --set enabled=false in the task worktree; lym config --resolved drift shows enabled: false # local. lym loom resume then ran Drift-Baseline again and spawned the capture child anyway, failing the same way. lym drift defer is refused at a failed or paused Drift-Baseline, and goto cannot move past it, so there is no way to take one run past drift.

Expected: either the local layer's enabled=false makes Drift-Baseline and Drift-Verify pass through on the next resume, or lym drift defer is admitted at a failed Drift-Baseline, as its halt's way forward.