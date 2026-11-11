```yaml
lens: fix-proposal
outcome: reported
```

No findings on the cap itself: the cap is not discoverable here (no test or lint pins it), so I could not show the F6 help sentence breaches it.

Findings:

1. **LOW**, `internal/creelcli/cli.go` post `Long`, the F6 sentence "The answer carries receiver_live, and a receiver that is not live gets a "creel: warning:" line on stderr naming how to start it."
   - The sentence claims `receiver_live` unconditionally, but `cli.go:463` omits it when `resp.ReceiverLive == nil`. That happens for a program receiver, a strand of another hub, and a `--wait-reply` resumed after a dropped connection. The fixer's own `doc.go` says as much.
   - The F6 proposal, taken verbatim, has an agent reading the help expect a key that is sometimes absent. The paragraph that commit 94372184 removed carried this exclusion: "A post to a program or to a strand of another hub answers no receiver_live."
   - A fix would be "The answer to a post to a strand carries receiver_live". Whether that fits the size cap is unknown.

2. **LOW**, `internal/creelcli/doc.go`, the F3 line "A `--wait-reply` resumed after a dropped connection answers no `receiver_live` and prints the warning no more than once."
   - The fixer took the doc alternative the review offered instead of keeping `ReceiverLive` from the first pending response. The text states a limitation the code does not test.
   - The new `cli_test.go` and `server_test.go` rows pin neither the resumed-wait path (an empty `postStamp`) nor the "no more than once" claim.
   - It is documented behaviour that rests on the code alone, so I count it a weak spot, not a defect.

Proposals I tried to break and could not:
- F1 (raw-connection row): it holds. A warning would have to arrive between the pending and final lines, and `toLive.next()` would then see `Progress` and fail. `SetIdle(alpha,false)` prevents the mail from being carried into the pane before alpha reads it. The `PanePID = os.Getpid()` strand claim has precedent at `server_test.go:477`.
- F2 (`--wait-reply` CLI test): it holds. `awaitStderr` guarantees the warning printed before the clock advance, and the exact-equality check after the timeout catches a doubled line. It mirrors the inject pattern at `cli_test.go:982-992`.
- F4 and F5: the wording is correct.

Further tool calls (1):
- Grepped the repo for a help-size cap, because my context did not show where the cap is defined. It found none.
