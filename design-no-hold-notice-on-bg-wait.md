# Shuttle hold notices fire while Merriam waits on its own background recovery

`no-hold-notice-on-bg-wait` · Note · bug, shuttle, webster

Imported from [issue #876](https://github.com/Knatte18/loomyard/issues/876).

Seen (umbrella-anchor, 2026-10-25): Merriam started `lyx webster recover-batch 06` in the background and ended its turn to wait for it, as its stencil allows. Each such turn produced a shuttle notice to the parent, 'ended a turn without its output files and is held' (#310, #311, #315, #317), although Merriam was waiting by design and needed nothing from the parent.

Expected: a turn that ends while the session's own background job runs (the RequirementBackgroundWake case) is not reported to the parent as held; only a turn that ends with no background job and no output files is.