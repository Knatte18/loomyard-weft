```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 and F2 change set for stale siblings and twins (lens fix-consistency).

- **LOW** — `internal/preflightshed/doc.go:19-20`. The text says the check refuses a marker "as after a version skew where a newer binary refreshed the stencils mid-run" and that "the reason carries the check's own way forward, a re-step with a binary that fills them." F2 changed the refusal to name two causes and two ways forward: a newer-stencils skew or a blank value, fixed by a newer binary or by giving the blank value text. The doc names only the first cause and way forward. Evidence: the new `CheckMarkers` error text in `internal/seatengine/prompt.go:145`, "...or a value is blank; way forward: re-step with a lym binary that fills them, or give the blank value text".

- **LOW** — `internal/loomcli/darnwiring.go:224`. The comment says "so a stencil newer than the running binary halts the run at Preflight". It names only the skew cause, while the check now also refuses a blank value. This is the same staleness as the `preflightshed/doc.go` finding, in the wiring's twin comment. Evidence: the grep output earlier in the session shows the line, and F2's refusal wording now covers both causes.

Checked and consistent:
- The `table_test.go:210-211` substrings match the new error text exactly.
- The grep for the old phrases ("newer than this lym binary", "binary that fills") found no other test asserting the old wording.
- `wiring_darn_test.go` asserts only `seat "darn"`, the stencil name and the marker, so it is unaffected.
- The `seatengine/doc.go:35` and `prompt.go:122-123` comments agree with the new wording.
- The F1 note on `Env.PreflightStencils` in `recipe.go` agrees with `darnwiring.go:261`, which sets the check only for the Darn-Write writer table, and with the `wiring_test.go:183` nil case.

Further tool calls: none. The context already held every file and grep result I needed.
