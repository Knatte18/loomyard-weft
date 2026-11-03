---
round: 1
focus:
  - "Expected's second sentence asks for a test pinning that each way-forward command these packages render parses: check whether any test feeds a rendered command's payload to the remove verb's parser, or only compares strings, across the adoptcli, fronttask and boardengine tests."
  - "Seen lists way-forward sites in adoptcli/fixloop.go, adoptcli/green.go, adoptcli/doc.go, fronttask/fronttask.go (HandRemoval and the one-open-front-task refusal) and boardengine/front.go: grep the whole tree for any remaining `lym board remove <slug>` bare-slug text, including docs, stencils and other packages the diff did not touch."
  - "boardengine.RemoveCommand in boardengine/board.go: judge the shell quoting of the JSON payload for a slug containing a single quote or other characters json.Marshal escapes, the discarded Marshal error, and whether a new exported helper in boardengine is licensed by the entry."
  - "fronttask.go now calls boardengine.RemoveCommand: check the import already existed and no dependency cycle or layering rule in the repository's constraints document is crossed."
  - "adoptcli/fixloop.go and adoptcli/green.go: check each rendered command reads correctly where it is embedded in surrounding backticks or quotes, and that the notice still names the same entry."
  - "boardcli/verbs_test.go stepFrontTask: check the changed constant still pins the refusal text against the real remove payload, and that the comment rewrite matches what the step does."
---

Seed focus from the Goal in the decision record and a read of the diff stat and the boardengine, fronttask and boardcli hunks.
