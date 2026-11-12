```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the `lym gate redundant` diff forces outside the lines it touches.

**Set per change**
- **New verb `lym gate redundant`.**
  - Covered: the `gatecli` registration in `cli.go:136`, `gatecli/doc.go`, `docs/overview.md` (both the package-tree line and the verb list) and `PATTERN-test-speed.md`'s coverage-map bullet.
  - Covered: no stencil, spec or shell-guard table lists the gate verbs. The grep for `gate report` hit only `gatecli` and webster code that uses the gate, not a verb list.
  - Covered: `gateslot/doc.go` and `lymdirs` mention `testmap` only for slot and exit-file behavior. The new verb takes no slot, so neither needs an edit.
- **New `testcover.JudgeMap` and `ReasonKept`.**
  - Covered: `testcover/doc.go` gained a line.
  - Covered: `ReasonKept` has one consumer, the verb. `gatecli/redundancy.go:590` judges the same map through `Measured()`, which is equivalent.
- **New report directory `docs/research/speed/coverage-map-pilot-2026-11-12/`.**
  - **Not covered:** `docs/research/README.md` indexes every report under `docs/research/speed/`. The last entry there is `testmap-upkeep-2026-11-11` at line 132, and the diff doesn't touch that file. The new report is unindexed.
- **PATTERN changes.** The rubric says the directive declares none, and `PATTERN-test-speed.md` only gained descriptive bullets. No `Pattern-Departure:` trailer is needed, since neither commit departs from a PATTERN entry.

**Findings**

- **F-A, LOW, consistency**
  - Location: `docs/research/README.md:132`
  - Summary: the new pilot report is missing from the research index, which lists each `speed/` report by link and one-line description.
  - Evidence: the index at `docs/research/README.md:125-132` ends at `testmap-upkeep-2026-11-11`, and `git diff --stat` of that file over the range is empty.
  - Fix: add a line after 132 linking `speed/coverage-map-pilot-2026-11-12/report.md` with a one-line description of the `internal/adoptcli` pilot.

**Tool calls (2 used, both parallel pairs)**
1. Grepped the repo for other lists of gate verbs and read `docs/research/README.md` 115-124, plus a diff stat of the README. The manifest lacked those.
2. Grepped `gateslot/doc.go` and `lymdirs` for `testmap` mentions, and `shellguard`/`clihelp` for `redundant`. These were one hop beyond the manifest.
