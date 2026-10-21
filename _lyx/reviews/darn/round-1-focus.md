---
round: 1
focus:
  - "Intake isolation: does any path in `internal/hubissues` or `internal/boardcli` intake handlers still import or call `internal/selfreportengine`, or a hard-coded `Knatte18/loomyard`? The board entry says intake never goes through selfreport."
  - "`fabricengine.HubRepoURL`: is the binding-then-origin-then-error order implemented as the summary claims, and does the error name both sources? Check the `.lyx-warp` binding read against how the binding is written elsewhere in `fabricengine`."
  - "Offline board verbs: does `internal/boardcli` open the hub repository only when an intake verb runs, so every other verb stays offline and token-free?"
  - "Stale references: do any docs, `--help` texts, `doc.go` files or skills in `internal/boardengine`, `internal/boardcli`, `internal/selfreportengine`, the `ly:board` skill or the orch role stencil still say intake reads the selfreport inbox?"
  - "Orch role stencil: against the board entry, does the diff drop, weaken or contradict a rule the entry did not ask to change, and does it make every change the entry asks for (no intake check at wakes or `landed`, intake only on the operator's word with the fork procedure kept, no review or review fork in parent-review, PR-Gate and the hand-on bullets)?"
  - "Summary claims against the diff: the unrecorded parent correction about the hub-wide orch rules sections is filed as a MEDIUM `decision` finding deferred to the parent, not fixed by revert; check the other summary claims, such as the parallel tests and the failing-transport token test, against the committed tests."
  - "Test economy: the intake tests carried over to `hubissues` and `boardcli`; does any new test duplicate a covering one, and does any added behavior, such as the unresolvable-repository error envelope, lack coverage?"
---

Seeded from the board entry and the change description; the concrete instances above are checked against the rubric's `Do not flag` list.
