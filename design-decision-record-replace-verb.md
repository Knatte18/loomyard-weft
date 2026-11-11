# lym loom decision replace folds a run's record under a lock, cuts the history check at it, and refreshes every stored record

`decision-record-replace-verb` · Task · feature, shed, loom, board · running · Code-Burler-Fix

Decision record: [Sketch](decision-decision-record-replace-verb.md)

Filed from the Sketch `decision-record-replace`.

Takes: lym board upsert --record-file from record-file-upsert, named by D3's filed refusal and by D6's parent directive; add's guards, refusals and notice in decision.go; CommitAnchoredPathsWithTrailers, FindTrailerCommit and commitEmptySnapshot in fabricengine; Board.HoldsRaddleFiling and raddleRunFiling
Delivers: lym loom decision replace with its guards, base-hash check, carry-over and added-entry checks, trailer commit and notice; the per-run decision lock shared by add, replace, store and writeCarryOver; recordHistory cut at the newest Lym-Record-Replaced trailer; D3's board copy and filed refusal; the D6 spec, parent directive and overview text
Exports: fabricengine.CommitEmptyWithTrailers, a first-parent trailer-key commit list and an anchored file's first-parent history with commit SHAs and positions; a gitrepo.Repo method that commits one path allowing an empty commit; discussionparser checks for carry-over block equality and the added-entry subset

The part changes these packages:

- internal/loomcli (about 900 changed lines)
- internal/fabricengine (about 300 changed lines)
- internal/gitrepo (about 80 changed lines)
- internal/discussionparser (about 150 changed lines)
- contracts/specs (about 15 changed lines)
- contracts/stencils/parent (about 15 changed lines)
- docs (about 10 changed lines)
