# Tier0 fails on a Go file gofmt would rewrite

`battencli-wire-test-gofmt` · Task · enhancement, gate · draft

Depends on: [`gate-call-shows-slot-wait`](design-gate-call-shows-slot-wait.md)

## Seen

The `design-first-route` landing (`9acc400f8`) listed a mis-spaced assignment in `internal/battencli/wire_test.go`,
`err =c.env.SeedChild.WriteSeed`, which gofmt would rewrite.
That line does not stand in the file at `9acc400f8` or at `3fb0787e3`, and `gofmt -l cmd internal tools` lists no file at `3fb0787e3`.
No gate checks formatting, so a mis-formatted Go file passes every tier.

## Idea

Add a gofmt check to tier0 (for example `gofmt -l` over the changed files).
