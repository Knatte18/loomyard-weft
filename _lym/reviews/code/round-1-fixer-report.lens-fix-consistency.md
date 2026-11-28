```yaml
lens: fix-consistency
outcome: reported
```

Task: checking the fix change set for agreement with its surroundings.

1. **LOW**
   - **Changed text:** `internal/testkit/tmuxrec/record.go`, `placeholdCall` (a paste's text now holds only `{hub}`).
   - **Summary:** the package doc in `internal/testkit/tmuxrec/tmuxrec.go`, the twin of this rule, was left as it was.
   - **Evidence:** it still says "A recording keeps the paths and process ids of the machine that made it as placeholders". The new `placeholdCall` doc says that in `Stdin` every bound value but the hub directory stays literal, including `{tmp}`, which is a path. The new test pins `/tmp/x` staying unchanged.

2. **LOW**
   - **Changed text:** the same `placeholdCall` change, against the Design decisions in `_lym/landing/summary.md`.
   - **Summary:** the existing decision on paste text no longer describes what is held.
   - **Evidence:** "A recording keeps the numbers in a paste's text as they are. Only process ids that tmux handed out become placeholders, and the text is what the code under test sent." The hub directory in that text also becomes `{hub}`. The decision also does not record that the other bound values (`{tmp}`, `{user}`, `{host}`, `{socket}`) are now deliberately kept literal.

No other disagreement found: the inventory index, `webster-shed.md` (its `noopSamplerSessions` is `shedrecipe`'s own, still a separate row) and the `reedengine` and `envkit` file headers agree with the change.

Tool calls:
- A `grep` over `docs`, `pattern` and `contracts` for twins of the changed symbols. It failed on pattern complexity and read nothing.
- A `git grep` for the same symbols in those docs, and for `envkit` in the inventory index. My context held only `loom-landing.md` of the inventory.
