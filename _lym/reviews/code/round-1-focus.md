---
round: 1
focus:
  - "Draft filing in internal/boardengine/raddlefile.go and internal/loomshed/raddlefile.go: does every filed task, project members of the beam shape included, land as a draft, and does any path still release or read the removed Release flag?"
  - "Removal of Board.ReleaseTasks and the Release seam (board.go, loomcli/raddlewiring.go, shedrecipe, envkit, battencli wire_seams_test): is any caller, doc comment or test double left referring to them, and is any behavior beyond the answer key's draft filing removed?"
  - "internal/raddleparser/proposal.go order_departures key: is it optional, validated and parsed correctly, and does the check that refuses two unordered parts on one package still hold?"
  - "internal/loomengine/raddletable.go: does the approval table carry a column of each part's dependencies, by part, so the order reads from the table alone, and are empty and multi-dependency cases rendered correctly?"
  - "Stencil and spec wording (raddle-template-agent.md, raddle-proposal-spec.md): does the text order a shared-package chain by the Sketch's order first, put a part the record says waits for nothing first, fall back to largest stated saving or most gained from, list every departure with its cost to the operator before approval, and say the tasks land as drafts, without dropping an existing rule?"
  - "Test economy in raddlefile_test.go, proposal_test.go and loomshed raddlefile_test.go: are the new or changed tests covering the added behavior (order_departures, dependencies column, draft filing) without duplicating existing coverage or asserting static prose?"
  - "Claims in _lym/landing/summary.md and docs/reference/weaving-glossary.md, loomshed/doc.go, boardengine/doc.go: does the diff bear out each claim, and do the docs match the draft-only behavior?"
---

Seed focus from the decision record's four wanted items: chain ordering, departures named to the operator, draft-only filing, and a dependencies column.
