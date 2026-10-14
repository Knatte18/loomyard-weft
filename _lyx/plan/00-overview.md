---
format: 5
approved: false
language: go
---

# Plan: insert and remove plan cards, reachable prescribed refusals

Close the two gaps of board task `plan-gaps` (issues #508 and #516).
An agent gets sanctioned verbs to insert or remove one plan card and renumber the cards after it, with a journal and a restore verb for an interrupted call, and the Plan-Write stencil and Plan-Review rubric gain a paired rule that every prescribed refusal names its reaching input and test.
The plan-format rewrite lives in `internal/planparser`, the verbs in `internal/loomcli`, and the begun-card bound reads webster's run record.

## Card Index

1 — planparser-insert-remove-card — InsertCard and RemoveCard renumber the plan behind a journal and reverse a failed apply
2 — planparser-restore-renumber — RestoreRenumber restores the plan from a journal an interrupted call left
3 — webster-last-begun-card — LastBegunCard reads the last card of the last begun batch from webster's run record
4 — loom-renumber-verbs — the insert-card, remove-card and restore-renumber loom verbs, their refusal rows and docs
5 — stencils-and-spec — the stencils name the verbs and carry the paired reachability rule; the spec says how numbering changes

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
