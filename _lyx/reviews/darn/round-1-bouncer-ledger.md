---
round: 1
ledger:
  - key: setting-value-html-escaping
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: setting-value-backtick-code-span
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW design findings on `internal/boardengine/render.go`.
F1 (`setting-value-html-escaping`): `settingValue` HTML-escapes non-string values via `json.Marshal`.
F2 (`setting-value-backtick-code-span`): a backtick in a setting value ends the code span early.
Neither gates.
