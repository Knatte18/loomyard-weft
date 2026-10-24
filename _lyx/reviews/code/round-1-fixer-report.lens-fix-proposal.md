```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the proposals behind F1–F4 and found two findings, listed below.

1. **MEDIUM** — F3 fix, `pattern/PATTERN-test-isolation.md` lines 47–48 (now say "built without the `tier2` and `tier3` tags" and "tag the file `tier2`").
   - The respell leaves the code's own text on the old spellings.
   - `internal/testkit/tmuxkit/tmuxkit.go:146` still emits "a package without a tmux or llm test tag; tag the test file that starts it `tmux`".
   - `tmuxkit.go:137` says "tag the test file `tmux`", and `tmuxkit.go:19` says "built without the `tmux` and `llm` tags".
   - An author following that runtime message tags a file `tmux`, which `cmd/lyx/tiertags_test.go` now refuses.
   - PATTERN and the code now disagree, and the code gives the wrong instruction.
   - `tmuxkit.go` is outside the review's `location`, but the diff made the disagreement: `allow_tmux.go:1` is already `tier2 || tier3`, and `allow_none.go:6` still says "tmux or llm tag".
   - The F3 proposal covers only PATTERN, so it leaves the twin text stale.

2. **LOW** — F1 fix, `cmd/lyx/tiertags_test.go` `firstRetiredTierSpelling`.
   - The proposal is sound: the tree walk handles all four `constraint.Expr` node types, a nil `Expr` falls through to `""`, and the two new rows fail under the old `Eval` closure.
   - The same short-circuit defect remains in its twin, `mentionsIsolationTag` (`cmd/lyx/tmuxisolation_test.go:131-145`).
   - It returns false from every `Eval` callback, so in `linux && tier2` or `tier0 && tier2` the right operand is never queried and a file is judged to name no isolation tag.
   - The review never named this twin, so this is not a regression.
   - It is a sibling that "was left as it was" and a reader would notice.

F2 and F4 hold up. For F2, the new bullet's claim matches the scan: `tierTagFailure` fails any retired spelling in any `scankit.Test` file. For F4, `loom-plan-spec.md:506` now names `tier0` through `tier3` only. I found no case where either proposal is wrong.

Calls made:
1. `grep` for build tags with `tmux`, `llm` or `integration` in `internal/testkit` and `cmd/lyx`, plus `lyx:guard` files and tag mentions in `tmuxkit`. I needed to know whether the code still keys on the old spellings; it does only in comments and messages.
2. `grep -A25` for `buildConstraint`. I needed to check nil handling for the F1 walk; the context lacked that function.
3. `sed -n 130,150p cmd/lyx/tmuxisolation_test.go`. I needed `mentionsIsolationTag`, the twin of the F1 `Eval` closure; the context lacked it.
