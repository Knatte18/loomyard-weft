```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: check that the F1–F4 edits agree with their sibling texts and that no twin was left stale.

No findings.

- The recipe comment (`darn-recipe.yaml:137`), the `docs/overview.md` Darn-Write paragraph, `internal/shedrecipe/doc.go:65-66`, `contracts/stencils/darn/darn-template-write.md:111` and the `redfirstgate.go` doc comments (lines 41 and 201) all say the same two things. A non-go hub passes the gate. A diff with no test holds it.
- A repo-wide search found no other text saying an unjudged check holds the run.
- The summary's reworded no-test sentence and its widened blast radius agree with the code. The code exits 5 before it looks for tests on a non-go hub.
- The new crash row in `redfirstgate_test.go` matches the `res.RunnerCrash` branch, which sets the terminal flag and prints the "test runner crashed" findings.

Tool calls made:
1. `grep -rniE "cannot judge|could not judge|unjudged|RedFirstUnjudged"` over the repo's `.go`, `.md` and `.yaml` files, excluding `_lym`, `.lym` and `.scratch`. I used it to look for twin texts that still describe the old hold. Nothing in my context covered files outside the changed set.
