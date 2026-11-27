## Goal

```markdown
## Aim (operator, 2026-11-27)

Cut the tier1 test time of internal/orchcli and internal/orchengine at least in half, measured before and after on this machine with `lym gate test --tier tier1` naming these packages.
The cut removes tests, it does not move them to another tier: delete tests that repeat what another test already pins, merge table cases that walk the same path, share heavy fixtures (repos, binaries, fake sessions) across a package's tests, and replace sleeps and real timeouts with injected clocks or shortened bounds.
Keep one test for every behaviour a PATTERN or a criterion names.

## Scope

- Only the test files of internal/orchcli and internal/orchengine, plus test-only helpers they alone use.
- No production change, except a seam a test needs to drop a sleep or real timeout, such as an injectable clock or bound.
- Report the before and after times per package and the net test-line change in the change description.

```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-27): Cut only where test time is real

Decision: The entry's package list came from an old coverage-mode survey. orchcli's tier1 takes about 2 s, so it is not cut. Measure internal/orchengine's tier0 time once with -count=1. If it takes 15 s or more, cut it at least in half by the entry's means; if it takes less, the run makes no test change and its change description reports the measured times as the result.

Rationale: orchengine has no tier1 tests, and a cut of a package that runs in a few seconds saves no gate time.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
