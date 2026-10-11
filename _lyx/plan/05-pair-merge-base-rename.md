# Card 5 — pair-merge-base-rename

**Rename:**
- `internal/loomcli#darnMergeBase` -> `plan:internal/loomcli#pairMergeBase`

**Intent:** Rename the reader of the task branch's merge base with its recorded parent branch from `darnMergeBase` to `pairMergeBase`, because card 6 tells it to every pair run's verify gate, not only darn's.
The rename retargets its one caller in `wireDarn` and its doc comment's opening self-reference, and changes nothing else; the file header of `darnwiring.go` is reworded on card 6, which edits that file.
