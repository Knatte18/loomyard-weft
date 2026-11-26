# Fixing code review round 1: audit-flags-master-friction

Two proposed fixes could not be applied as written.
F1 asked for a rev-parse through the existing Git seam, but `HeadSHA` reads through go-git, which cannot open the records subdirectory.
F12 asked for wording that `TestRenderPrompts_FrictionDirective` forbids with friction off.

F2's proposal also created a terminal dead end after a hand merge between runs, which only the self-check forks caught.

The verify gate also failed once on `TestGateTestMap/NamedPackageIsRebuiltWhenFresh` (`internal/gatecli`),
a test this branch does not reach.
Its fixture test `alpha.TestAlphaTwo` passed only when run by itself in the full run, and all 38 targets passed in a run of that test alone.
It looks load-dependent and is worth a fix of its own.
