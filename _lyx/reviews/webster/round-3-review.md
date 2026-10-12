---
verdict: APPROVED
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`: `_lyx/shed/test-tmp-cleanup/status.json` gives `product.parent` as `main`, and no `_lyx/loom/rework/` round exists.

## Focus question: does every test package carry a `TestMain` through `tmuxkit.Main`?

Yes.
`internal/fabricvocab/testmain_test.go` (commit a85a498ee) defines `TestMain` calling `os.Exit(tmuxkit.Main(m))`, the form `PATTERN-test-isolation` requires.
The Publish failure's cause was the missing `TestMain` in the merge-in package `internal/fabricvocab`, and that cause is gone.

I scanned every directory holding a `_test.go` file for a reference to `tmuxkit.Main`.
Three directories lack one, and none is a violation:

- `internal/planglyph/testdata/glyphchain/callees` and `internal/planglyph/testdata/glyphchain/shapes` are `testdata` fixtures, which `go test` does not build as packages.
- `internal/testkit/tmuxkit` is the kit itself and calls the unqualified `Main(m)`, a form `TestTmuxIsolation_FixtureTrees` pins as accepted.

I ran `lyx gate test ./cmd/lyx` in the foreground and it passed.
That run includes `TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain`, the test the Publish verify reported failing.

## Rejected

- No new test, and no test-economy finding: the fix adds one `TestMain`, and the cmd/lyx guard already covers its behavior.
