---
round: 1
focus:
  - "internal/planparser renumber: does every apply step have a reversal path, and does the journal on disk always complete (temp journal moved into place by Rename only), per the move-mechanics decision?"
  - "internal/planparser RestoreRenumber: is it the journal's only reader, and does restoring after an interruption at each apply step leave the plan directory as it was before the call?"
  - "internal/loomcli verbs insert-card, remove-card, restore-renumber: are they bounded by websterengine.LastBegunCard and webster's state-mutation lease, and does each refusal wording live only in renumberRefusal with a matching refusal-table row?"
  - "websterengine.LastBegunCard: does it read every batch record's cards, terminal or not, matching the begun set DispatchScope derives rather than DoneCards?"
  - "Scratch paths and PATTERN invariants: do planparser's scratch and journal paths join lyxdirs.DotLyxDirName with no new file spelling _lyx or .lyx, and does the diff honour PATTERN-lyxdirs-single-declarer and PATTERN-durable-vs-ephemeral-state?"
  - "Test economy: do the injected-failure tests via RenumberIO each reach a distinct reversal or restore path, or do any duplicate another test's coverage or re-test a helper that a public-surface test covers?"
  - "Stencils and docs: do the writer, rework and review stencil changes and the doc.go and docs updates match the verbs' actual behavior, and is the reachability rule paired between the Plan-Write stencil and the Plan-Review rubric?"
---

Seeded from the plan's card index and shared decisions; the focus names where to look, not what to conclude.
