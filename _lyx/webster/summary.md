# test-suite-change-amplification: all 45 batches done

Every batch reached `status: done` through `record-batch`, with no stuck, failed or recovered batch.
The plan-level integration fork ran the `## verify:` command once and reported `status: OK`.

## What was built

- Batches 01–03: the testkit root and its enforcing scan, the `gitkit` query and commit helpers, and one shared `gitkit` spawn rule for the Tier Purity and Hermetic Env scans.
- Batches 04–07: private git helpers in `fabricengine`, `gitrepo`, `fabriccli`, `cmd/lyx`, `hubforge`, `ideengine`, `loomcli`, `landingshed` and the webster pair moved onto `gitkit`.
- Batches 08–13: `hubforge` gained `AddPairWith` and `OpenFabric`, and `fabricengine` tests moved onto `*hubforge.Hub`.
  The refusal sentinels were exported and asserted with `errors.Is`.
  The `fabriccli` and `fabricengine` duplicates collapsed onto one owner each.
- Batches 14–16: one scripted `fakeTmux` in `reedengine`, and the `reedcli` reap-cycle table.
- Batches 17–19: the gitrepo parity oracle moved to its own leaf package, and `locationkit` replaced three location builders.
- Batches 20–30: the `shuttlefake` and `shedfake` kits, the `shuttleengine` and `shedadapters` fixtures, and the conversions of the producers, `landingshed` and `shedrecipe`.
- Batches 31–36: the `envkit` kit, the `shedbuild`, `loomrecipe` and `battenrecipe` conversions, and one reflective every-seam-filled test each in `loomcli` and `battencli`.
- Batches 37–43: the `logcapture`, `envelope` and `lyxbin` kits and their callers.
- Batches 44–45: `slices` replaced `containsPath` and `equalStringSlices`, and the last old-policy citations were dropped.

## Deviations reported by `record-batch`

Files changed outside a batch's declared file-ops:

- 11: `internal/fabricengine/sentinel.go`, `internal/fabricengine/clone.go`
- 15: `internal/reedengine/directory_test.go`
- 21: `internal/websterengine/integration_test.go`
- 28: `internal/landingshed/export_test.go`
- 38: `internal/loomshed/gates_test.go`, five `shedadapters` and `shuttleengine` test files, and others listed on that batch's digest

## Items from the forks' own reports

These come from the forks' hand-back text rather than the digests, so treat them as unverified.

- Several forks reported 7 `go test -tags smoke ./internal/loomcli/...` failures.
  Batches 06 and 22 checked two of them against the unchanged tree and found the same failures.
  Batch 43 reported 3 further smoke failures in `reedcli` and `orchcli`.
  The plan-level verify does not run the smoke tier.
- Batch 09 diverged from card 08's text: `SkipPush` suppresses only the weft push.
- Batch 45 recorded the leaf test count at 9498, against 9457 at batch 01.

## Integration suite

`go build ./...`, `go vet -tags smoke ./...`, `go test ./...` and `go test -tags integration ./...` passed at HEAD `69a7acbf2`.
