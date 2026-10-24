# Plan-Burler fix round 1 — gate-runs-once

F1's proposed fix was to move the shortening card ahead of the growth cards. That rested on the bound "each intermediate size stays at or below the final projection", which a card gate does not apply: once the file is above `HeadroomBytes`, a card's `lint-pattern` step passes only when it does not grow the file.
I carried out the move, including a `remove-card` plus `insert-card` that renumbered twenty cards and needed every prose card reference fixed by hand, before the self-check showed the last growth card still failed.
Undoing it took a second `remove-card` and a second pass over the same references.

A plan check that projects `lint-pattern` per card, rather than only the final size against the base, would have caught both the original defect and the wrong fix at the review stage.
