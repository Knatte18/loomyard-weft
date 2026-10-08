# Plan review round 1 — a Rename-vs-Edit finding the plan gate cannot accept

Review finding F2 asked for symbol renames in Cards 4 and 5 to be typed `Rename` over member glyphs, on the reading of the spec's type table alone.
The fix attempt failed `lyx loom validate-plan` with blocking `containment-file-overlap` findings, because other cards target the files holding those symbols whole (`destroy.go`, `fabric.go`, `weftwiring.go`), so the finding's remedy cannot exist in this plan's file-level layout.
The rubric's `Do not flag` list covers checks the gate already runs, but not this interaction between a card type and the cross-card containment check; running the gate before recommending a retyping would have caught it in the review phase.
