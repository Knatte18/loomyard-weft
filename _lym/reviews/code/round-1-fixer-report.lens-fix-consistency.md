```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for stale siblings and cross-references (fix-consistency lens).

no findings.

I checked these twins and each agrees with the changed text:
- **Merged pull request drops its record:** `internal/landingshed/prreview.go` (the `answerReopen` doc), `internal/landingshed/doc.go:24` and the `TestPRReview_Reopen` doc comment now all say "closed or merged". No other file words the closed-only rule.
- **Beam-Await retry:** `internal/pawlengine/doc.go` and the `enter`, `answerReopen` and `entered` comments describe the same retry. `contracts/recipes/pawl-recipe.yaml:76` ("Beam-Await removes the request on entry") and `internal/beamcli/doc.go:53` still hold. `docs/overview.md:459` doesn't mention either behaviour, so nothing there goes stale.
- **`Unready` skipping an empty landing directory:** the closure comment in `internal/beamcli/deps.go` and the sentence in `internal/beamcli/doc.go:52` agree.
- **F7 and F8:** the `TestDobbyGeometry` comment and map entry, and the `beamDescribeDeps` comment, match their code.

Further tool calls (3):
1. `grep` for the "loses its record", "reports closed" and "removes the request" wordings across the repo, to find unchanged twins.
2. `sed` of `contracts/recipes/pawl-recipe.yaml` and `internal/pawlengine/beamawait.go` around their reopen and removal wording, which my context lacked.
3. `grep` of `docs/overview.md` for reopen text, to see whether it describes the changed behaviour.
