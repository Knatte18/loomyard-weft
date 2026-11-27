---
round: 1
ledger:
  - key: docgo-claims-title-display-on-every-launcher-form
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: content-doc-comment-blurs-title-lines
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised two wording findings on `internal/desktoplauncher`.
The first (F1) is that `doc.go` and the change description claim the `lym: <shortname>` title is displayed on every launcher form,
while a `.cmd` launcher shows its file name.
The second (F2) is that the `content` doc comment does not say which line of each form carries the title.
Neither is gating.
