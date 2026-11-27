---
round: 2
ledger:
    - key: closed-span-ways-forward-name-refused-verb
      rounds:
        - 1
      status: resolved
    - key: archived-review-dir-reopens-scope
      rounds:
        - 1
      status: resolved
    - key: pr-agent-operator-exemption-not-built
      rounds:
        - 1
        - 2
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: role-files-stencils-still-name-decision-add
      rounds:
        - 1
      status: resolved
    - key: scope-closed-at-untested-on-real-dir
      rounds:
        - 1
      status: resolved
    - key: scope-closed-recipe-key-untested
      rounds:
        - 1
      status: resolved
    - key: hard-cap-answer-route-names-in-scope
      rounds:
        - 1
      status: resolved
    - key: added-branches-without-test-rows
      rounds:
        - 1
      status: resolved
    - key: recipe-spec-omits-scope-closed
      rounds:
        - 1
      status: resolved
    - key: decision-add-help-drops-facts
      rounds:
        - 1
      status: resolved
    - key: plan-spec-tier0-seconds-unconditional
      rounds:
        - 1
      status: resolved
    - key: tier0-budget-key-not-guard-category
      rounds:
        - 1
      status: resolved
    - key: merge-danger-blast-radius-narrow
      rounds:
        - 1
      status: resolved
---
Round 1 raised the scope line that closes at the first Code-Review round:
ways forward naming the refused verb, a run-directory read that could reopen the line, missing tests and stale docs.
Round 2 reports those resolved.

One thread remains.
The operator exemption at PR-Review was settled by the parent's ruling: there is none.
The code already behaves that way,
but the role file `contracts/orchrole/orch-role.md` and the change description still claim an exemption for operator feedback on the pull request.
