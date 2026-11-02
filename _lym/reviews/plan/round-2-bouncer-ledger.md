---
round: 2
ledger:
  - key: card9-ends-fixers-on-unmoved-run
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: runproc-doc-lacks-fixer-roles
    rounds: [1]
    status: resolved
  - key: card3-stale-headers-naming-deleted-members
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: recipe-spec-lacks-fixer-element-key
    rounds: [1]
    status: resolved
  - key: card9-slug-read-error-marked-transient
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: card9-loomcli-doc-listed-under-uses
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four MEDIUM findings; round 2 carries one BLOCKING and three LOW.
The BLOCKING finding is new: card 9's `endGateFixers` returns a pair-slug read error marked transient, which `PATTERN-transient-stop` reserves to `internal/shedengine`.
Its fix removes the mark from the card's Intent and its test row.

Card 9's round 1 concern, ending fixers on a refused or no-op move, is now a LOW wording and doc-contract gap: the opening sentence says "once shed has admitted", but `Relaunch` runs the guard before `Admit`.
Card 3's stale comments remain as a LOW target gap in `render.go`, `strand.go`, `recoverbatch.go` and `testReplyPath`.
The runproc doc and recipe-spec findings are not raised again and are resolved.
Card 9 also lists `internal/loomcli/doc.go` under Uses where Extends fits.
