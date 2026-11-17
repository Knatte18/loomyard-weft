Lens fix-affected-set.

No findings.

Checked: F1 (no other `.Dir = ...WorktreePath()` site in non-test Go), F2 (no copy of the false claim left), F5 (the non-test callers of `orchcli.BeamDown` do not assume an empty report on error), F9 (the only tree line naming `internal/hibernatecli`).
Unchecked: F3, F4, F6, F7 and F8, whose premises are not sets; F9's other overview mentions, and whether the `hibernatecli` package doc needs a `PauseHub` mention.

Tool calls: four greps, one per finding checked: the `.Dir` assignments, the claim wording, the `BeamDown(` callers, the `hibernatecli` mentions in markdown.
