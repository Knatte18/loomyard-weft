---
verdict: APPROVED
rationale: "All six round-1 findings are fixed at HEAD c7b8cc23d (F1 key/value slog attrs, F2 guid-a/guid-b pin with a negative check, F3-F6 comments and rename), the diff implements Cards 1-3 as written, and the plan's verify block passes; one leftover doc NIT does not block."
---

# Webster-Review round 1 — Bouncer verdict

Range: `product.parent` is `main`, read from `_lyx/shed/status-strand-by-driver/status.json`.
I reviewed `git diff $(git merge-base main HEAD)..HEAD`: the three card commits plus the six fixer commits.

## Round-1 findings, checked against HEAD

- **F1** — `removeStatusStrands` (`internal/loomcli/bootstrap.go:240`) now logs `"guid", s.GUID, "cause", err` and `"cause", err`, with `loomcli:`-prefixed messages matching the package's other warnings.
  Fixed.
- **F2** — the `first remove fails` row uses `guid-a`/`guid-b`, asserts `guid=guid-a` is present and `guid-b` absent, and the `status fails` row asserts `cause=boom`.
  The pin is no longer vacuous.
  Fixed.
- **F3** — every sentence whose wording the diff changed now uses semantic line breaks.
  A few touched lines still end mid-phrase where a reworded sentence runs into an unchanged legacy one: the `Third, it stays inside the region ...` line in `start.go`, `wire() itself spawns no process ...` in `integration_driverbootstrap_test.go`, and the `PreRun;` and `status and pause ...` lines in `shedcli/parity_test.go`.
  The wording of those sentences did not change, and CONSTRAINTS.md's Comment Line-Break Convention says untouched lines are never rewrapped and the rule covers only new and changed comments.
  I accept the fixer's boundary, so this is not a finding.
  Fixed.
- **F4** — the comment now reads "seed, commit and bring up reed".
  Fixed.
- **F5** — the `start_watchdog_test.go` header now says `c.reed.Up()` and the branch's `c.reed.Status()` run on a concrete engine.
  Fixed.
- **F6** — `verbSmokeTimeout` replaces `stepSmokeTimeout`; no references to the old name remain, and its doc covers both `step` and `start --no-attach`.
  Fixed.

## Independent read of the diff

- Card 1: `removeStatusStrands` reads the table once, removes every exact-name match non-recursively, and warns and carries on after each failure.
  The table test covers all six cases the card lists, and captures the log through `logger.SetOutput` with cleanup back to `os.Stderr`.
- Card 2: `loomPreStep` calls `c.reed.Up()` in place of `ensureStatusStrand`, and the lock release and `KindBootstrap` failure path are unchanged.
  `smoke_drivingsurface_test.go` holds the three tests the card names.
- Card 3: `start` runs `Up` and then branches on `mustUseLLMDriverArm(driver)`, reusing the value `seedAndCommitBootstrap` already returned, so the driver is still read in one place.
  The llm arm never fails; the go arm keeps the old failure path.
  The watchdog spawn follows the branch on both arms.
  `ensureStatusStrand` has one caller, which calls `Up` first.
  The invariant sentence, the `Long` help, `docs/overview.md` (both descriptions of `start` and the `step` sentence) and the smoke expectations all match the card.
- Per-card mechanical checks: no card has a Rename or Delete group.
  Card 1's Create target has no equivalent elsewhere (settled in the round-1 report).
  Card 2's created file exists with its tests.
- Verify block, run at HEAD: `go build ./...`, the unit and integration test commands, and `go vet -tags smoke` all pass.
  The working tree is clean.

## Non-blocking NIT

`ensureStatusStrand`'s second doc paragraph (`internal/loomcli/sharedbootstrap.go`, beside the reworded first paragraph) still justifies its lock-agnosticism with "the lock's position differs between the two calling verbs".
The paragraph just above now says only `start` calls it.
The file-level rationale still applies to the file's other shared helpers, so the sentence is stale, not wrong about behaviour.
It is recorded in the ledger as open and does not block.

## Focus departures

The round-1 report has no `## Focus departures` section, so there is nothing to ratify or reject.
