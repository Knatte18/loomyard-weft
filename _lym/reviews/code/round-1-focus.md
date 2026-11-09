---
round: 1
focus:
  - "internal/boardengine/projectrelease.go: does the release select only direct member tasks (`project` equals the project's slug) in `draft`, leaving released, running and done members, nested projects with their members, and entries outside the project untouched?"
  - "internal/boardengine/projectrelease.go: is each member validated as its own release would be, and does one failing member refuse the whole write with nothing released, including the project's own null status?"
  - "internal/boardengine/projectrelease.go: does a project with a run status or `done` get refused with each member's release, by slug, named as the way forward, and is that refusal reached by a test?"
  - "internal/boardcli/cli.go: is the new Board method called only from the set-status verb, with `Board.SetStatus`, `Store.SetStatus`, upsert, merge and the other null-status callers (batten cancel's claim clear) unchanged and releasing no member?"
  - "internal/boardcli/cli.go: does the verb take the project path only for a null status on a project and keep its old behavior for a non-null status and for a task?"
  - "internal/boardengine/projectrelease_test.go and internal/boardcli/verbs_test.go: do the tests cover the D8 acceptance criteria (draft members released in one write, other members and nested projects untouched, one failing member releases none, run-status or done project refused), without duplicating an existing test?"
  - "internal/boardcli/doc.go and internal/boardengine/doc.go: do the doc changes match what the code does, and do they follow the repository's comment conventions?"
---

Scope is D8 of the decision record: a project release through `lym board set-status` that releases its draft member tasks.
