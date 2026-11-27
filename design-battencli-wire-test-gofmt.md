# battencli wire_test.go holds a line gofmt would rewrite, and no gate caught it

`battencli-wire-test-gofmt` · Note · bug, gate, batten

## Seen

`internal/battencli/wire_test.go` has `err =c.env.SeedChild.WriteSeed`, which gofmt would rewrite, and it landed on main.
No gate checks formatting, so a mis-formatted Go file passes every tier.

## Idea

Fix the line, and add a gofmt check to tier0 (for example `gofmt -l` over the changed files).
