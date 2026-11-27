## Goal

```markdown
## Aim (operator, 2026-11-27)

Cut the tier1 test time of internal/configengine, internal/configsync and internal/hubreconcile at least in half, measured before and after on this machine with `lym gate test --tier tier1` naming these packages.
The cut removes tests, it does not move them to another tier: delete tests that repeat what another test already pins, merge table cases that walk the same path, share heavy fixtures (repos, binaries, fake sessions) across a package's tests, and replace sleeps and real timeouts with injected clocks or shortened bounds.
Keep one test for every behaviour a PATTERN or a criterion names.

## Scope

- Only the test files of internal/configengine, internal/configsync and internal/hubreconcile, plus test-only helpers they alone use.
- No production change, except a seam a test needs to drop a sleep or real timeout, such as an injectable clock or bound.
- Report the before and after times per package and the net test-line change in the change description.

```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-27): Cut only packages at 15 s or more

Decision: The entry's package list came from an old coverage-mode survey. Cut only a package whose measured time, in tier1 or in tier0 when it has no tier1 tests, is 15 s or more; leave a faster one unchanged and report its measured time. Here that cuts internal/hubreconcile and leaves internal/configengine and internal/configsync unchanged.

Rationale: Cutting a package that runs in a few seconds saves no gate time; the measured baseline shows only hubreconcile above the bound.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
