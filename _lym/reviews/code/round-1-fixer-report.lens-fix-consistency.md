```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- **F1, `holdwrite.go`:** The `FixHold` doc comment still matches: "No hold standing is refused with the adopt that writes one". The line-403 refusal keeps `subjects` from `tries`, which is right because no hold exists and there is no set to match. The line-439 refusal now uses `holdSubjectNames(want.Subjects)`. That agrees with `WriteAdopt`'s refusal at lines 370-374, which uses the hold's own set, and with `planAdoptHold`'s same-set rule. The updated test row asserts both subjects.
- **F2, `takeoverwrite.go`:** The doc comment on `rewriteOverOrigin` is unchanged and still accurate. The new no-upstream message follows the form of the sibling messages in the function. `fastForwardToUpstream` returns a nil error for a missing upstream, but that is a separate function with its own contract, and the review did not name it.
- **F3, `cli.go`:** I found no other text saying a plain adopt waits. The remaining "wait" wording is `holdWaitWayForward`, which tells the caller to wait, and that matches the new help text. I did not read the `adoptcli` package doc for a "waits" sentence. My grep over `.go` and `.md` files found none.

Tool calls made:
1. A grep for "waits on" and "plain adopt" across `.go` and `.md` files, to find twins of the F3 wording that my context lacked.
2. A grep for "no upstream" in `internal/fabricengine`, to find sibling messages for F2 that my context lacked.
3. A grep for "another machine" and `holdWaitWayForward` in `internal/adoptcli` and `holdwrite.go`, to check the F3 refusal wording against the code that my context lacked.
