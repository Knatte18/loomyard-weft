---
round: 1
ledger:
    - key: reconcile-commits-unloadable-profiles
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: merge-danger-understates-reconcile-reach
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: duplicate-arras-default-assertion
      rounds:
        - 1
      status: open
      class: consistency
      severity: NIT
      fix: mechanical
---
Round 1 raised three findings.

- `reconcile-commits-unloadable-profiles` (F1): the reconcile keeps an operator-repointed `defaults.arras`
  while dropping the arras block it names, and `runprofile.Load` then refuses the whole file.
  The requested fix adds a one-time profiles reinterpretation and an upgrade-path test.
- `merge-danger-understates-reconcile-reach` (F2): the change description's blast radius and reconcile bullet
  leave out the rewrite of existing hubs' files and the kept-default case.
- `duplicate-arras-default-assertion` (F3): an assertion in `TestTemplateLadder` repeats one `TestLoad` already holds.
