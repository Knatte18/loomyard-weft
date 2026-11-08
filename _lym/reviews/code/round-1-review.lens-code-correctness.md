```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, covering the one code change in the diff (`routedDecisionsLine` in `internal/burlerengine/prompt.go`) and the paths that render it.

**List**

- `routedDecisionsLine` (constant, `prompt.go:238-241`): concatenation of two string literals. It has no `%` verb and no `{{`, so it is safe as a `fmt.Sprintf` `%s` argument and inside stencil `Fill`. Correct.
- `fixWork`, `gateFindingsPath == ""` and `decisionsRouted` true (`prompt.go:231`): the work line renders "…but its `decision`-class findings. Those went to your parent… until your parent's ruling on one reaches you this round, which the parent ruling rule below covers." The `{{.work}}` marker sits in "What to fix", and the "Parent ruling rule" heading is later in the same `burler-step-fix.md`. "Below" resolves, so the pointer is reachable. Correct.
- `fixWork`, gate-findings path with `decisionsRouted` true (`prompt.go:235`): the same constant is appended and the same pointer holds. Correct.
- `fixWork`, `decisionsRouted` false (`prompt.go:226-233`): the constant is not rendered. The parent ruling rule still ships in the step, but it only triggers on a ruling that arrives. Nothing is routed in this case, so the fixer already works every `decision` finding, and a ruling adds an entry to the report. No contradiction. Correct.
- `Fix` skip path with routed decisions and passing gates (`fix.go:141-148`): no fixer spawns, so no ruling can arrive, and the skipped report text is unchanged. Correct.
- Test coupling (`fix_test.go:282-283`, `:365`): the `wantWork` rows use the constant itself, and the non-routed check asserts the constant is absent from `instruction-fix.md`. The extended text still satisfies both. Correct.
- The `lym creel post --parent "<text>"` form the stencil names, with no `--expect-reply` (`creelcli/cli.go:413-423`): the refusals cover only `--parent` with `--reply` or with a receiver, and `--expect-reply` with `--reply`. `--parent` alone is valid. Correct.
- The "no parent recorded" branch: `parentReceiver` refuses with a way forward (`cli.go:255`), and the stencil says the fixer-report entry is then the only flag. Consistent.

No findings.

**Tool calls:** none. The manifest already held every definition I needed.
