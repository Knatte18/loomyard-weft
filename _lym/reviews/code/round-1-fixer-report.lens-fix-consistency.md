```yaml
lens: fix-consistency
outcome: reported
```

No findings, with one LOW inconsistency.

Task: check the fixer's change set (F1–F7) for twins and siblings left stale.

**LOW** — `internal/gatecli/redundant.go` `Long` help text (the changed output paragraph).
- Summary: the `--help` text lists what the verb prints but does not name the new `map_build_failure` key.
- Evidence: `redundant.go:52` says "It prints the map's commit and build time, the candidates, the kept tests with their reasons and the tests not judged with why." The fix put the key in `doc.go` but not here.
- The rubric's LOW class covers a sibling left stale in a way a reader would notice, and this is the same output contract in two places.

**Checked and consistent:**
- `testmap.go:375` still says "set the dir of the lowest of tier0 and tier1…". That is a different verb with a different failure path, so the F6 rewording of `redundant.go` does not make it a stale twin. The reviewer asked only for the redundant verb's wording.
- `docs/overview.md:934` describes the verb at a level that needs no mention of the new key.
- The `README.md` entry for the pilot report follows the neighbouring entries' format.
- The F7 clause ("one more than the 34 map entries") matches the "34 entries" in the Result row.
- The `JudgeMap` doc comment now stands alone and matches the behaviour its code shows.

Calls made:
1. `grep` for "lowest of tier0 and tier1" and for the verb's mentions in markdown: to find twins of the F6 wording and docs describing the verb's output, which my context lacked.
