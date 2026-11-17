---
round: 1
focus:
  - "internal/boardengine/store.go and internal/boardcli/cli.go: does every path that prints parent_beam (list, get, any other envelope or brief) map the prime to boardengine.RootSubject, with none left printing the empty slug?"
  - "internal/boardengine and hubgeom: do boardengine.ParentBeam, Board.ParentBeamOf and hubgeom.ParentOf still return the empty slug for the prime, and does any internal reader of BriefTask.ParentBeam now receive root where it expected empty?"
  - "internal/boardcli/text.go: does the --text P0 row take the title the board files give the root project, stay out of the JSON tasks array, and behave when the root has no title or the listing is empty?"
  - "contracts/orchrole/orch-role.md and plugins/ly/skills/board/SKILL.md: do they read root as the prime and address its orch as <shortname>:orch, with no leftover wording that treats an empty parent_beam as the prime?"
  - "boardcli help and package doc: is root described as the prime consistently with the amended D8, and is every stale empty-slug mention removed?"
  - "Test economy in parentbeam_test.go, store_test.go and verbs_test.go: do changed assertions cover the root mapping, the beam-slug case and the P0 row, and does any test assert static prose?"
---

Scope is D8 of the decision record plus its two parent amendments: the mapping sits at the printing boundary only.
