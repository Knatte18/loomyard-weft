# Fixer-report, darn round 1

## Changes

- F1 (fixed as proposed): `pattern/PATTERN-config-strictness.md` lists `runprofile` as a third stance, with the reason a removed starting profile must stay removed; the `PATTERN.md` entry names the exception.
- F2 (fixed as proposed): `internal/boardcli/verbs_test.go` has an unknown-profile refusal row for `upsert-batch` and one for `merge`.
- F3 (fixed as proposed): `internal/boardcli/verbs_test.go` has a profile-change notice row past Seed-Child, plus an unchanged-profile row that expects no notice, and a task-session re-send of the stored profile that passes.
- F5 (fixed as proposed): `internal/configcli/configcli_test.go` has a `--layer local` row in `TestDispatchSetProfiles`, refused with the hub file untouched.

## Disputed

- F4, `premise false`: no input reaches the second `checkLocalDocument` in `LayerRunSettings` (`internal/configreg/runsettings.go:103`).
  Every check it runs is per key and per leaf: `configengine.ValidateLocal` checks each leaf path, and `CheckValues` is `sheddriver.CheckLocalOverride` and `reviewbudget.CheckLocal`, each testing single keys against fixed bounds.
  Two documents that each pass therefore merge, key by key through `yamlengine.Merge`, into a document that passes.
  A row asserting the refusal cannot be written without a test-only module.

## Deferred

None.

## Self-check

- fix-consistency, LOW: wording differs between `PATTERN.md` and its background file. Fixed as proposed (`PATTERN.md` says "exactly one" again).
- fix-consistency, LOW: `internal/configengine/doc.go:5` and `config.go:4` still say "the two" / "which policy". Disputed, `premise false`: both stay true, as they describe `Load` and `LoadOrTemplate`.
- fix-affected-set: observation only, below LOW: `modelspec`, `burlerengine` and `reviewbudget` also call `StackLayers` and were never in the lists. Not a finding; not changed.
- fix-code-correctness: no findings.
- fix-proposal: no findings.

## Gate repair

The gate refused the branch because `PATTERN.md` grew 58 bytes past its headroom line.
The `config-strictness` entry now reads "strict `Load` or `LoadOrTemplate`, bar `runprofile`", a net 5 bytes over the entry's original length; the reason and the third stance stay in `pattern/PATTERN-config-strictness.md`.
`lyx gate test ./cmd/lyx` passes.
