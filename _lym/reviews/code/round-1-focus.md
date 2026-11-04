---
round: 1
focus:
  - "internal/shuttlecli/run.go and internal/shuttleengine/apierrorretry.go: does `run --model` resolve `<alias>[<effort>]` and `<engine>:<model-id>[<effort>]` the way `open` does, picking the engine from the alias, and does a model the engine rejects end the run at once on both the claude and codex paths, leaving no live strand?"
  - "internal/shuttlecli/open.go and internal/shuttleengine/codexengine/codexengine.go: does `open` start a no-prompt interactive Codex session, and where it cannot, is a codex spec refused before anything is created? Do `--list-models` and the help state which engines `open` can start, consistently with the code?"
  - "internal/shuttleengine/codexengine/prepare_test.go, fixture_test.go and open_test.go: does any test still reach the operator's real home (`~/.lym/codex`, `~/.codex`)? Does the new open_test.go duplicate coverage an existing test already gives?"
  - "internal/shuttleengine/codexengine/home.go, doc.go and codexengine.go: do the committed-`.codex` check and the package doc now agree on tracked versus untracked, and is the choice backed by the safety argument? Is the refusal's way forward worded for a plain `lym shuttle run` as well as a review round?"
  - "Scope: the diff touches claudeengine/signals.go, shuttleengine/sessionsignal.go, spec.go and docs/overview.md. Does each change trace to one of the four numbered items in the decision record's Goal?"
  - "Commit messages and `_lym/landing/summary.md`: does any claim there go unbacked by the diff?"
---

Seed focus drawn from the decision record's four numbered items and the diff's file list; the diff itself is unread beyond its stat.
