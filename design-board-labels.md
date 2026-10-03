# A labels field on board entries, and tier 2 retired

`board-labels` · Notes · feature

Seen 2026-10-03: the board model is now tasks (tier 1) and notes (tier 3, one per entry), with labels as a bracketed prefix in each note's brief (`[ide][orch] …`) because the board has no labels field; tier 2 is unused, and README rendering already says Tasks and Notes (`6a74ee1be`).
Idea: a real `labels` field with a filter in `lyx board list` and labels in the README, and dropping tier 2 from the model and the `promote` verb's vocabulary.
