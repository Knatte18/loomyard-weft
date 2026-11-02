---
round: 1
focus:
  - "internal/boardcli/runsettings.go refuseRunSettingsCaller: does the guard let a beam's orch change only entries whose project link resolves to its own beam, both as stored and as the payload leaves it? Check each path: a non-string `project` in fields, a stored entry with an empty slug, a payload that omits `project`, and an entry that is itself a beam project."
  - "internal/boardcli/runsettings.go: the old guard refused any strand where agentname.TaskSlug was non-empty; the new one calls agentname.Parse and returns nil on a parse error. Does any strand name the old guard refused now pass, and does agentname.MatchesRole and the Slug field select exactly the beam orch?"
  - "internal/boardengine/parentbeam.go BeamUnder: does it agree with ParentBeam's definition of the nearest open beam for a project link, including a done beam, a project the board does not hold and a nested project group? Does it read the board consistently with the other Board methods in the same file, and is the error and not-found handling right?"
  - "Scope against the Goal: the Goal asks that the orch role tell every orch to name a profile in the entry's `profile` field before the batten run starts, as well as the guard change. Does the diff change the orch role text or stencil, and if not, is that omission licensed by the Goal?"
  - "Same-file consistency: internal/configcli/hubwide.go still refuses on agentname.TaskSlug. Does the Goal license leaving it unchanged, and do the docs in internal/boardcli/doc.go and internal/boardengine/doc.go still say what the code does?"
  - "Test economy in internal/boardcli/verbs_test.go and internal/boardengine/parentbeam_test.go: does the BeamUnder unit test duplicate coverage the boardcli beam-orch test already gives, or the reverse? Is any behavior the diff adds left uncovered, such as the stored-link check when a payload moves an entry out of a beam?"
---

Seed pass: the review range holds one fix commit and one doc-comment commit; the decision record carries only a `## Goal` section, so the Goal is the measuring stick.
