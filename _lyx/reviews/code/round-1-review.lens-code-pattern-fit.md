```yaml
lens: code-pattern-fit
outcome: reported
```

Task: the code-pattern-fit lens, checking the amended side against the repo's rules. The amendment is the `docs/overview.md` merge.

**Rules checked**

- **Overview gate bullet vs `doc.go`, `scope.go`, `tiers.go` (focus 1): held.**
  - The sentence says a `--card` scope under the `go` scoper also vets the card's packages under each other tier tag that adds a file to one of them, so a tag-set compile break fails at the card. That matches `computeScope`'s `tagTiers` and `planTierRuns`, which plans vet runs only for `scopeCard` with `cfg.Scoper == "go"`.
  - "Each other tier tag" is a shorthand for the listed and covered exclusion. `tiers.go` skips a tier that is listed or covered, and `doc.go` states this. The overview sentence is looser but not contradicted.
  - "A failing tier stops no later tier, vet or guard run" matches `doc.go` and the unchanged ordering in `planTierRuns`: listed tiers, then vet, then guard.
  - The closing line naming every failed run matches the `runLabel` `vet tierN` label.
  - The merge kept the sentence beside the parent's other edits. The parent's edits are PATTERN link retargets (`PATTERN-told-geometry` and `PATTERN-hub-containment` both became `PATTERN-cwd-resolution`) away from the gate bullet, with no textual conflict.
- **Parent changes under `internal/gatecli` (focus 2): held.** `git diff 5e36064a..main --stat -- internal/gatecli` is empty, so the parent changed neither the package nor its tests. The task's gatecli diff is intact in the range.
- **PATTERN-gate-slot: held.** The vet run is a `tierRun` with `Slots: 1` and goes through the same tier-run path, so it holds a slot. It adds no raw `go vet` outside a slot.
- **Gate carry (green record only): held.** The vet run uses the tier-run path and the decision record's own-fingerprint design. I read no code showing the fingerprint itself, so this rests on the record and on the `d6209f3c1` fix commit that covers the vet run's carry.
- **CLAUDE.md "Docs land in the same commit": held.** `doc.go` gets a `# Vet step` section, and the Verb and Guard paragraphs are updated. The `docs/overview.md` gate bullet is updated too.
- **Declared PATTERN changes landed: raised nothing.** The directive's constraints list shows no declared-changes list, so the check has nothing to apply.
- **Commit messages and departures: held.** `git log` over the range shows no `Pattern-Departure:` trailer, so no departure needs judging.
- **Decision-record constraints PATTERN-gate-carry, PATTERN-test-tiers and PATTERN-tier-tags: not checkable.** None of the three is in the directive's PATTERN list or in `pattern/`. I recorded no finding for this, since the rubric holds the diff to the entries the directive lists.
- **PATTERN-test-speed on the added tests: held, from what I could see.** The added tests carry `tier0` and `tier1` tag strings. The only `t.Parallel` or build-tag lines the grep saw were inside fixture strings, so I could not confirm the parallelism convention. Test economy belongs to the code-tests lens.
- **Comment conventions: held.** The new doc comments on `tagTiers`, `tagAddsFile` and `vetStepRun` start with the symbol name and carry no end-of-line comments. Line wrapping is not a finding under the rubric.

**Findings:** none.

**Tool calls (1)**
1. A single Bash call: `pattern/PATTERN-gate-slot.md`, the commit trailers, a `pattern/` listing for carry and tier entries, and a grep of the tier test diff. My context lacked the gate-slot background, the trailers and the test tags.
