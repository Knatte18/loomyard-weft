```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind the three findings and check the fix covers every member.

Findings:

- **LOW** — stale text in `internal/beamcli/land.go:36`. The `DraftDescription` seam doc still reads "writes a starting point over the beam's change description". `DraftBeamDescription` now refuses when `summary.md` already stands, so "over" is wrong. This is the one remaining site that describes the old overwrite behaviour. The sibling text in `cli.go`, `describe.go` and `beamcli/doc.go` was updated.
- **LOW** — `contracts/orchrole/role-beams.md:33` was not re-read, so I can't say whether it implies `--draft` can be re-run over an existing file. Other text may also say the draft is run "first" without giving the refusal and the delete-to-redraft way forward. I did not check that the orch role contract covers the refusal.

Findings I left unchecked:

- F2: no set search applies.
- F3: no set search applies.
- F1's other callers: `DraftBeamDescription` has one production caller (`deps.go:256`), and `BeamDraftDeps` is built in one place (`deps.go:468`), which now sets `Slug`. The only test construction (`beamdescribe_test.go`) sets `Slug` too. No other construction was missed.

Calls made:
1. One grep across `.go` and `.md` files for the draft symbols and the "over summary" and "replaces the file" phrasing. It was needed to enumerate callers, `BeamDraftDeps` constructions and stale doc sites, which my context lacked.
