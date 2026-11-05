# fabric clone --reset removes the hub before refusing a mismatched --shortname

`clone-reset-deletes-first` · Task · bug, fabric · running · Finalize

Imported from [issue #67](https://github.com/Knatte18/loomyard/issues/67).

Seen: `lym fabric clone --reset --into ~/Code/Knatte18 --shortname scribe https://github.com/Knatte18/scribe-weft https://github.com/Knatte18/scribe` on an existing hub (old layout with _board) failed with 'this repo's recorded shortname is "scr", not "scribe"; refusing to change it', but its envelope reported mutation path_removed '.' recursive and partial: true: the whole existing hub directory was already deleted when the refusal came.

Workaround: rerun `lym fabric clone` without --reset and without --shortname; the hub came back from its remotes. Nothing was lost only because every branch was pushed.

Expected: --reset validates every argument against the remotes (shortname, binding) before removing anything, so a refusal leaves the existing hub untouched.