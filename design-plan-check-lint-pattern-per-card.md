# Plan check does not project the lint-pattern step per card, so a wrong plan-review fix passes review and fails only at self-check

`plan-check-lint-pattern-per-card` · Task · enhancement, loom, burler · running · Code-Bouncer

Imported from [issue #866](https://github.com/Knatte18/loomyard/issues/866).

What happened: a plan-Burler fix moved a shortening card ahead of growth cards on the premise that each intermediate file size stays at or below the final projection.
A card gate does not apply that bound: once a file is above `HeadroomBytes`, a card's `lint-pattern` step passes only when the card does not grow the file.
The move required a `remove-card` plus `insert-card` that renumbered about twenty cards and needed every prose card reference fixed by hand; the self-check then showed the last growth card still failed, and undoing it took another `remove-card` and a second pass over the references.

Why abnormal: the plan check validates only the final size against the base, so neither the original defect nor the wrong fix was visible at the review stage, and card renumbering is costly to repeat.

What to do: make the plan check project `lint-pattern` card by card (the running size against the headroom rule), and consider renumbering helpers that update prose card references automatically.

Provenance: task gate-runs-once; notes burler-plan-r1.md