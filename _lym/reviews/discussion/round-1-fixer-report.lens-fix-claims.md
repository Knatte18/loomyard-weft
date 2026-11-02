```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked every bound, guarantee and code fact the fix added or rewrote against HEAD.

Claims checked and found to hold:
- D1: `VerifyFixers` records carry a status (`internal/websterengine/verifyfixer.go:81`, `:201`).
- D2: the deadline bound of up to two fixer timeouts plus four inner runs. A blocked evaluation that crosses the deadline is followed by the deadline finalize, which evaluates the gate again (`internal/shuttleengine/wait.go` header line 19 and the deadline exit near `:420`).
- D4: `frontDecision` caps only with a strand name set, no `--wait` and no enclosing slot (`detachfront.go:232-251`). The runner's environment drops the strand name (`detachfront.go:130`). `joinable` compares against the recorded tree identity (`detachfront.go:320`).
- Constraints: Darn-Write's first entry is `clean` (`darn-recipe.yaml:105-107`), and `NewCleanGate` fails on any uncommitted or untracked path (`cleangate.go:16-27`).
- The F13 fact: every record on disk has a call id, the only gate keys recorded are `""`, `adopt`, `code-burler-fix` and `darn-write`, so no `card` call is recorded. Twice 594 s is about 19.8 min, which is under 30.

Findings:

1. MEDIUM. D5 Bound, the sentences "Go enforces the re-run, the fixer's role and timeout, and the later rows below" and "Darn-Write: … the run's later Code-Bouncer and Code-Burler-Review rows judge them" (the Plan and Discussion bullet says the same of their Bouncer and Burler-Review rows).
   Summary: the bound treats the later review rows as a guaranteed check on the fixer's edits, but config can let them judge nothing.
   Evidence:
   - `contracts/recipes/darn-recipe.yaml:118`: "With loom.yaml's code_max_bounces at 0 the Bouncer settles approved at once".
   - Code-Bouncer also reads `auto_accept_setting: loom.code_auto_accept`.
   - Code-Burler-Review is reached only through Code-Bouncer's `on_stuck`, and its `on_done` goes to Describe (`darn-recipe.yaml:127-150`).
   - So Go enforces that these rows run, not that they review the fixer's diff. The tapestry Bouncer and Burler-Review pairs use the same Bouncer/BurlerReview engines and `on_stuck` chaining (`tapestry-recipe.yaml:126`, `:155`, `:272`, `:314`).
   - The bound should name that these rows judge only when their bounce and auto-accept settings let them review.

No other findings.

Tool calls:
- Read `darn-recipe.yaml:113-175` and grepped the tapestry recipe's review rows and entry gates: my context had only lines 95-124, so it lacked the review rows' settings and chaining that the D5 Bound claim rests on.
- Read `wait.go` lines 1-40 and 360-430: my context had only `evaluateGate`, so it lacked the deadline and `Expired` handling behind D2's overrun bound.
- Ran jq over `_bolt/.lym/gate/runs` for records with no call id and for every gate key: the fixer's measurement grouped by call id, so a `card` record without a call id would have been missed.
