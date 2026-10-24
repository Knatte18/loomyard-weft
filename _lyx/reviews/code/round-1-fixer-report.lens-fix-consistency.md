```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix's consistency with its surroundings, meaning twins left unchanged elsewhere.

**Findings**

1. **MEDIUM**: `pattern/PATTERN-test-speed.md:70` (twin of the F2 rewrite at line 19).
   - The line says a `tmuxkit`-referencing file "is further constrained to the `tmux` tier ... so `tmux`, `llm`, `tmux || llm` and `tmux && !windows` pass while `integration || tmux` fails."
   - The F2 bullet now says `cmd/lyx/tiertags_test.go` refuses any constraint naming `integration`, `tmux` or `llm`. Every example on line 70 is such a constraint, so the two lines contradict each other.
   - This is the same retired-spelling premise as F2/F3, in a line the review did not name.

2. **MEDIUM**: the same file has more lines in the retired spellings that the fix left as they were.
   - Line 42: "Every gate compiles the `llm` tier through `go vet -tags llm`". The nearby rule at the top of the file says `go vet -tags tier3`.
   - Line 49: "a module-wide or `tmux`-tier test".
   - Line 61: "A `tmux`-tier test that drives `lyx`".
   - Line 103: "The `integration`-tier guard".
   - F3 fixed the same "tmux-tier" phrasing in `pattern/PATTERN-test-isolation.md`, so these are its unfixed twins.

3. **LOW**: other `pattern/` files still use the retired spellings as tier or tag names.
   - `pattern/PATTERN-test-economy.md:24`: "sits in a `tmux`-tier file".
   - `pattern/PATTERN-testkit.md:15`: "Only `llm`-tagged test files import it".

4. **LOW**: `internal/testkit/tmuxkit/tmuxkit.go:19` doc comment.
   - It says "a test binary built without the `tmux` and `llm` tags".
   - F3 respelled the same sentence at `pattern/PATTERN-test-isolation.md:47` to `tier2` and `tier3`, so the two now disagree.
   - I did not read the code to check which tags `Main` actually tests. If the code still keys on the old names, the PATTERN change is the wrong one. If it keys on `tier2`/`tier3`, the comment is stale.

**Checked, no finding**
- The F1 rows and the F4 and F2 edits agree with the tier-tag scan's refusal message.
- The PATTERN-test-isolation edits at lines 14, 27, 28, 47 and 48 are internally consistent.

**Tool calls**
1. Grep of `pattern`, `PATTERN.md`, `docs` and `contracts/specs` for retired-spelling phrases: to find twins of the changed lines, which my context did not hold.
2. Grep of `internal/testkit/tmuxkit/*.go` for the old tag names: to check whether the code behind the F3 respelling at isolation line 47 agrees.
