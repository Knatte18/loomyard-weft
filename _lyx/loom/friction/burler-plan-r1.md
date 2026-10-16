# Burler fix, plan round 1: the installed lyx disagrees with HEAD's plan checks

After the fixes, `lyx loom validate-plan` blocked on three `edit-test-file` findings against Edit entries this round never touched.
The check exists only in the installed binary, not in this worktree's HEAD.
What it asks for contradicts the plan's Shared Decision on naming external-test members, and the review had called the plan clean.
Working out that the findings came from a newer binary, and were not a regression from the fix, took a detour.
