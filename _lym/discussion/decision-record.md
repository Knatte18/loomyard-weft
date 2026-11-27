## Goal

```markdown
## Aim (operator, 2026-11-27)

Cut the tier1 test time of internal/loomcli at least in half, measured before and after on this machine with `lym gate test --tier tier1` naming these packages.
The cut removes tests, it does not move them to another tier: delete tests that repeat what another test already pins, merge table cases that walk the same path, share heavy fixtures (repos, binaries, fake sessions) across a package's tests, and replace sleeps and real timeouts with injected clocks or shortened bounds.
Keep one test for every behaviour a PATTERN or a criterion names.

## Scope

- Only the test files of internal/loomcli, plus test-only helpers they alone use.
- No production change, except a seam a test needs to drop a sleep or real timeout, such as an injectable clock or bound.
- Report the before and after times per package and the net test-line change in the change description.

```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
