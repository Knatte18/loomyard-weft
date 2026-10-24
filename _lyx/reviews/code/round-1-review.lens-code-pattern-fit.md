```yaml
lens: code-pattern-fit
outcome: reported
```

Task: check the diff against the repository's PATTERN entries, CLAUDE.md and the commit-message rules (lens code-pattern-fit).

**Rules checked**
- `PATTERN-gate-slot`: held. `vetStepRun` sets `Slots: 1`, and a vet run takes the `runListed` → `runGreenAware` → `runWithSlots` path, so the vet build runs only inside a held slot (`internal/gatecli/tiers.go:200-209`, `run.go:349-389,400-409`).
- `PATTERN-gate-carry`: held. A vet run skips only through the green record, with the same `subtract`/`recordPass` path as a listed tier. Its fingerprint differs from the listed tier's because `Command` carries `-tags tierN`, so a vet pass cannot be mistaken for a tier pass (`green.go:64-79`). `AllTargets` and `Dir` are empty, so the `Whole` shortcut at `green.go:350` cannot fire wrongly.
- `PATTERN-test-tiers`: held. No gate key, site, stencil or tier list changes, and the `--card` form stays internal to the verb.
- `PATTERN-tier-tags` and `PATTERN-test-speed`: held. `tiers_test.go` is `tier0` and pure; the two new `--card` cases are in `scopes_integration_test.go`, which is `tier1`. Every test file carries exactly one tier tag.
- `PATTERN-gate-self-check-parity`: not applicable, because no mechanical gate is added.
- CLAUDE.md "Docs land in the same commit": held. `doc.go` gains a "Vet step" section, and the Verb, guard-step and `planTierRuns` text is updated. `docs/overview.md` mentions the vet step. No new module or PATTERN invariant is added, so `PATTERN.md` needs no change.
- Declared PATTERN changes: none declared and none in the diff, so nothing to land.
- Commit messages and departures: no `Pattern-Departure:` trailer, and none is needed, since the diff does not depart from the answer key. Both commits are clear and the first closes #865.
- Comment conventions (Go): held. The new doc comments are self-contained, begin with the symbol name, and have no end-of-line comments. Line breaks are the lint-comments gate's to judge.
- CLAUDE.md markdown semantic line breaks: held in `doc.go`. The `overview.md` change is one table-style bullet that was already a single line.
- Scope against the board entry and decision record: held. Only `internal/gatecli`, `doc.go` and `overview.md` change, and `gate.yaml` and the other scope forms are untouched.

**Findings**

1. NIT, `consistency`, `internal/gatecli/scope.go:15` and `:402`: `tagTiers` adds the `sort` import and calls `sort.Ints(numbers)`, although the file already imports `slices` and uses it throughout. `slices.Sort(numbers)` matches the surrounding idiom and drops the extra import. Evidence: the import block at `scope.go:7-16` and the call at `scope.go:402`.

I found no other rule broken.

**Tool calls (1)**
1. One Bash call read the `docs/overview.md` diff, the build-tag headers of the two test files, `pattern/PATTERN-test-speed.md`, `tierFingerprint` in `green.go`, and the `goBinary` uses. My context lacked the overview diff, the test-file tags and the fingerprint body.
