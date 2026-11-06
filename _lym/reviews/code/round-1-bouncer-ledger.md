---
round: 1
ledger:
  - key: slack-authority-sender-condition-dropped
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: overview-role-file-carries-index
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: role-prose-assertions-in-tests
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings.
F1 (BLOCKING design) maps to `slack-authority-sender-condition-dropped`: the Slack bullets in `contracts/orchrole/role-slack.md` no longer name `<shortname>:SLACK` as the sender.
F2 (LOW consistency) maps to `overview-role-file-carries-index`: `docs/overview.md` says the role file carries the operator index.
F3 (LOW design) maps to `role-prose-assertions-in-tests`: new test assertions pin the role's static prose.
Each fix text names its exact edit, so all three are mechanical.
