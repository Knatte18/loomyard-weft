# Fixer-report, webster round 1

## Changes

- F1 (NIT, fixed as proposed): the `orchDirName` doc comment in `internal/orchcli/paths.go` now says it is the module's own subdirectory name beneath the anchor's `.lyx` directory and beneath `_lyx`, where the rules files live.
  Committed as `55b91404a`.
- Gate finding (`TestCLIForm_EveryCommand`: `lyx orch rules` names `--from` in its `Use` without marking it required): `internal/orchcli/rules.go` now calls `MarkFlagRequired("from")`.
  `writeHubRules` keeps its own empty-`from` refusal, which its table test still covers.
  `lyx gate test ./cmd/lyx ./internal/orchcli` passes.

## Self-check

Run once for F1, before the gate finding; the gate fix is a one-line registration.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
