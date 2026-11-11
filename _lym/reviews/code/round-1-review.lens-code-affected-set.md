```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change, I list what it forces outside the touched lines, then give the findings.

**Set per change**
- **`decision store` admitted on no-approval runs and refused on filed raddle runs.** Callers, doc.go and docs/overview.md are updated. The store command's `Long` help in `decision.go` is not (finding 1).
- **`decisionNoticeOpening` and the new replace opening.** The opening is read only by the parent directive stencil, `decision.go` and `decision_test.go`. The stencil and the test pin both openings. Covered.
- **`perchFilingRefusal` renamed to `raddleFilingRefusal`.** No stale reference remains. Covered.
- **`recordHistory` cut.** Its callers (`wiring.go` lines 418 and 895, `decision.go`, `darnwiring.go`) all read the bounded history. Covered.
- **`CommitPathAllowEmpty`.** It is added to the pinned list in `cmd/lym/gitrepoboundary_test.go`. Covered.
- **`fabricengine` seams and `TrailerCommit.Position`.** `doc.go` is updated in fabricengine and gitrepo, and existing `TrailerCommit` literals in tests gained `Position`. Covered.
- **PATTERN-run-records.** The `PATTERN.md` line carries the declared text exactly, and `pattern/PATTERN-run-records.md` is updated. The wording of the Project record rule changed with it. Covered, except for the consumers in finding 2.
- **Discussion spec, parent directive and overview (D6).** All three name `replace` and `--record-file`. The "Parts no writer authors" bullet is added.
- **Spec lines 124 and 126** ("appended only by `lym loom decision add`") stay true, since `replace` never appends. Covered.
- **`--record-file` in `internal/boardcli` and `internal/boardengine`.** The flag exists on the parent branch. Covered.

**Findings**

1. MEDIUM, consistency: the `decision store` help text is stale. `internal/loomcli/decision.go` around lines 1875–1884 says store is "the same copy 'decision add' makes", "works once the run is past its Discussion approval" and "refuses a recipe with no Discussion approval". `decisionStoreVerb` now admits a no-approval run whose entry carries a stored record, and refuses a raddle run whose filing is done. The help states the opposite of the behaviour for the first case and omits the second. Evidence: help text at lines 1875–1884, behaviour at `decision.go` lines 1146–1180. The `--help` text is user-facing, and the D3 and D6 acceptance criteria assume it matches.

2. LOW, consistency: three beam project-record texts still say `decision add` is the only writer, against the declared `PATTERN-run-records` text ("`lym loom decision add` or `replace` `<beam-slug>`"). `replace` is admitted on beam runs, because `boardRecordStored` special-cases `record.project`. The three texts:
   - `docs/overview.md:814` ("is the one way its scope changes")
   - `contracts/stencils/loom/loom-directive-project-record.md:20`
   - `contracts/recipes/pawl-recipe.yaml:15` ("After that only `lym loom decision add <beam-slug>` changes it")

   The diff changed the pattern text and the overview's verb list but not these three.

3. LOW, consistency: `docs/overview.md:1160` still gives `decision add`'s signature as `--by parent|operator`, while `add` also admits `arbiter`. That predates the diff, so I do not count it. The new `replace` line at overview 1170 gives `--by parent|operator|arbiter`, so it is fine.

No finding on "Declared PATTERN changes landed" (landed, text identical), "Commit messages and departures" (no `Pattern-Departure:` trailer in the log, and none is needed) or "Stencil wording". The parent directive keeps its existing rules and adds the replace notice line and the four parent instructions. Those four sit under "When you are the parent of a run yourself", which is a placement choice the summary records.

**Tool calls (2)**
1. A grep batch for `perchFilingRefusal`, the notice opening, `decision add` and `decision store` mentions across contracts, docs, plugins and pattern, and the store refusal strings. The manifest held only the diff, not the rest of the repository.
2. Lines 1872–1900 of `decision.go` (the store command's `Long` help, not in the diff hunks), a grep for run-records tests, and a grep for `--record-file` and the overview's store mentions.
