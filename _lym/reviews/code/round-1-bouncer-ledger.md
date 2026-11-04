---
round: 1
ledger:
  - key: shellguard-quoted-gh-words-bypass-deny
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: syncText-failed-edit-route-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: shellguard-clustered-short-flags-bypass
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: shellguard-multiline-mutation-bypass
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resync-gates-and-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: publish-alternatives-without-rows
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: github-allowlist-beyond-entry
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: stale-comments-refuse-all-gh
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: edit-failure-warns-every-poll
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: wait-comment-names-only-description
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review; no earlier ledger exists.
Two gating design findings are open: quoted gh words slipping past the publish deny, and the untested failed-edit route of the text resync.
Eight lower findings cover further shellguard bypass spellings, missing test rows, an allowlist wider than the entry, and stale or imprecise comments.
All fixes are local corrections whose result the finding text fixes.
