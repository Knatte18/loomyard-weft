---
round: 1
ledger:
    - key: beam-with-idle-members-still-refused
      rounds:
        - 1
      status: resolved
      class: scope
      severity: BLOCKING
      fix: design-wrong
    - key: endbeam-fake-stand-in-no-real-board-test
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: endbeam-second-owner-writer
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: stopbeam-skip-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: merge-danger-understated
      rounds:
        - 1
      status: open
      class: consistency
      severity: MEDIUM
      fix: mechanical
    - key: cancel-help-lost-teardown-wait-sentence
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: cancel-help-output-omits-beam-ended
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: beam-probe-branches-lack-rows
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: endbeam-failure-skips-records-push
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
---
Round 1 opened every key above; no earlier ledger exists.

The substantive items are the cancel's refusal of a beam whose members never ran, which the board entry expects to end as a plain project,
and the test side: a fabricated boardengine answer in the cancel table, no real-hub beam cancel, and an untested orch-stop skip.
`Board.EndBeam` also writes an owner outside the path boardengine's doc names as the only owner writer.
The rest concerns the change description, the cancel help text, missing table rows and the order of the records push after a failed flag clear.
