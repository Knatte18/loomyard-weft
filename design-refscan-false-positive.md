`fabricengine.RefScanner` (`internal/fabricengine/refscanner.go`) flags any Bash command whose text matches `lyx(\.exe)?\s+(fabric|weft|warp)\b`, including a quoted search pattern.
Run `operator-surface` blocked at Webster batch 28 (docs card, commit `d369f8915`, report OK) because the fork ran `grep -n "fabric remove\|lyx fabric.*remove" docs/overview.md` — a content search in the warp, not a fabric operation.
The flagged transcript never enters `seenForkTranscripts`, so every retry of `record-batch 28` re-audits it and fails again: the run cannot resume without a code change.
Fix direction: match `lyx fabric|weft|warp` only in command position (start of the command or after `;`, `&&`, `||`, `|`, `$(`), not inside quoted arguments; keep the weft-path match.
Docs cards will keep hitting this, since docs describe fabric verbs.

Fixed for the spelling in `777d93af7` (hub prime, deployed): command position only, quoted spans blanked.
Still open: the path half (`\S*-weft\b`) matches any word ending in `-weft`, quoted or not, so a docs grep for `main-weft` would trip the same dead end.