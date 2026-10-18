---
round: 1
focus:
  - "Scope against the board entry (`lyx board get` for slug `run-profiles`, plus the record's `Added after Discussion` entries): does the diff make every change the entry asks for, and nothing it does not license, such as the `JSONEntries` and `yamlengine.SetEntries` path or the `lyx board profiles` verb?"
  - "`internal/runprofile` and the `configreg` registration: is `Load` free of template fill-in, is the module hub-wide and seed-only, and does the load check name the profile on a refusal?"
  - "`configreg.LayerRunSettings` and the Seed-Child seam in battenshed and battencli: does the task's `run_settings` win over the profile through `yamlengine.Merge`, and does every failure (unknown profile, undecodable file, refused key) wrap `ErrInvalidRunSettings` while an unreadable file stays a hard error?"
  - "`boardcli` profile validation on `upsert`, `upsert-batch` and `merge`: is the type and known-name check applied on all three paths, and does a profile change get the same task-session refusal and running-pair notice as `run_settings`?"
  - "`boardengine.Task.Profile`: is it kept out of the run lock's scope check, cleared by `\"\"`, and rendered in the README entry cell as claimed?"
  - "Repository rules: do `doc.go`, `docs/overview.md`, the `ly:board` skill and `PATTERN.md` land in the same diff as the new behavior, and do the shortened help pages keep the facts an operator needs?"
  - "Test economy: which new behaviors lack coverage, and which new tests duplicate an existing test, naming it?"
  - "Change description: check each claim in `_lyx/landing/summary.md` against the diff."
---

Seeded from the change description and the commit list only; the board entry was not read in this pass.
