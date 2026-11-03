# drift: capture child spawns at the worktree root, ignoring a subdirectory fabric anchor

`drift-anchor-and-skip` · Note · bug, drift, stencil

Imported from [issue #23](https://github.com/Knatte18/loomyard/issues/23).

Seen in hub NORCE-DrillingAndWells/Models, whose fabric is anchored at src/csharp/NORCE.Models by .lym-anchor. A darn run (testsupport-rule-and-helpers) failed Drift-Baseline three times with 'the capture child ended before it recorded its start'. The capture-child log holds only: cwd is outside the recorded fabric anchor subtree: cwd <worktree root> does not equal the anchored directory <worktree root>/src/csharp/NORCE.Models (recorded by .lym-anchor). The drift producer starts its capture child with cwd at the worktree root whatever directory the run is resumed from, so every re-launch fails identically and drift cannot run in any hub with a subdirectory anchor.

Workaround: the parent runs lym drift defer and lym loom resume in the task worktree, then lym drift waive at Drift-Verify, so the run goes on with no baseline.

Expected: the capture child (and the compare child) start in the anchored directory, the same directory every other lym verb requires.