# Verify localizes failures and Publish never runs on a dirty tree

`verify-publish-hardening` · Planned · feature

## Goal

A failing integration test stops on the card that caused it, Publish never verifies or pushes a dirty tree, and no gate is skipped after a crash.
refusal-audit stopped five times on integration-verify regressions that the plan-level verify could not localize.

## Also folded in, not in bolt before

- refusal-audit self-report 2: card `verify:` commands run package-scoped, so `cmd/lyx` repo-wide invariant scans first run at the plan level; Plan-Write could append `go test ./cmd/lyx/`.
- refusal-audit self-report 3: Webster's integration verify runs against the working tree, so an uncommitted edit can mask a non-compiling HEAD.
- Publish's conflict resolver reuses `.lyx/loom/conflict-resolution-r1.md` across Publishes (`internal/mergeresolve/spec.go:60`), so a later conflicting merge-in is refused while an earlier file exists; refusal-audit hit it.
- Deliver every agent prompt through a file, so no role can hit the 30000-byte launch limit (`internal/shuttleengine/claudeengine/command.go`); Master already does (`e39909bbc`).

## From `card-verify-integration`: Cards that touch integration-tagged tests verify with -tags integration

No refusal-audit card declared a verify: line; the fork gate is go build plus the package's unit tests (webster-body-implementer.md:30), untagged, while 11 of 48 touched test files are //go:build integration.
Want: Plan-Write adds `go test -tags integration <pkg>` to a card's verify when it touches a tagged file; a message-rewording card greps the repo for the old text first so stale exact-string assertions become part of the card; Master re-runs integration tests after a rework fork.

## From `publish-dirty-tree`: Publish must not verify or push a dirty tree after merge resolution

refusal-audit's Publish merge left historyfold_test.go uncommitted and verify ran on the dirty tree; earlier publish-review-fixback had the same uncommitted merge-resolution edits

## From `verify-gate-crash-after-merge`: Landing verify gate skips a merge interrupted before its marker write

Found reviewing PR #308 (`publish-verify-after-merge`).
Fabric reports a finished-but-interrupted merge as already up to date, so the gate cannot tell a verified tree from an unverified one.
Robust fix: record the task HEAD that last passed verify, and verify whenever HEAD differs from it.