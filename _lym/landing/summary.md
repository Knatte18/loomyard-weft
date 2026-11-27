# Measure the parsers' and burlerengine's test time; no test change needed

The aim was to cut the test time of `internal/discussionparser`, `internal/planparser` and `internal/burlerengine` at least in half.
The current tree gives no reason to cut, so this change edits no file.

## Measured times

None of the three packages holds a tier1 test.
Their tests are tier0, apart from three tier3 smoke files in `internal/burlerengine` that no gate runs.
A tier1 run over the three packages starts no test binary.

Tier0 package times, measured on the current tree with `-v -count=1`:

| Package | Time |
|---|---|
| `internal/discussionparser` | 0.07 s |
| `internal/planparser` | 0.18 s |
| `internal/burlerengine` | 1.59 s |

The slowest single tests are subtests of `TestEngine_Review_TableShape` and `TestEngine_Fix_Outcomes` in `internal/burlerengine`, each between 0.2 s and 0.7 s.

Net test-line change: 0.

## Design decisions

- A package is cut only when its measured time is 15 s or more, in tier1, or in tier0 when it has no tier1 tests.
  The package list came from an older coverage-mode survey, and every package now runs well under that bar.

## Merge danger

**Door:** two-way
**Blast radius:** none
