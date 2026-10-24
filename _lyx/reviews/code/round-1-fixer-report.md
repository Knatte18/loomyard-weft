# Round 1 fixer-report

## Changes

All five findings fixed as proposed, one commit each; tier-0 gate for `runprofile`, `battencli` and `boardcli` is green.

- F1: `internal/boardcli/doc.go` documents the `defaults` field and per-recipe `settings` of `lyx board profiles`.
- F2: `internal/boardcli/cli.go` upsert help says an empty profile takes the task's recipe's default profile.
- F3: `TestTemplateDocsProfile` in `internal/runprofile/runprofile_test.go` pins the docs darn block and that `Settings("docs", "loom")` errors.
- F4: `internal/battencli/runsettings_test.go` has a row for a profile-less loom task with an undecodable profiles file, expecting Stuck naming `profile "light"`.
- F5: the `decodeDefaults` comment says the defaults entry is never read as a profile.
- Self-check follow-up (fix-affected-set): `doc.go` states the empty case as `"profiles":[]`, not a bare `[]`.

## Gate repair

The verify gate failed `TestCLIForm_EveryCommand`: the F2 help text pushed `lyx board upsert`'s Long past the 2000-byte cap.
The F2 line now reads `"" takes the recipe's default`, and two words are trimmed elsewhere in the same help ("full" before "markdown body", "itself" before `"-"`).
Tier-0 for `cmd/lyx` and `internal/boardcli` is green.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: LOW, the `profiles` doc's "`[]` when there is none" was stale beside the new envelope; fixed as proposed.

## Disputed

None.

## Deferred

None.
