```yaml
lens: fix-consistency
outcome: reported
```

no findings from the lens as scoped, apart from the two below.

Lens fix-consistency: I checked whether the changed text still agrees with its siblings and whether any twin was left unchanged.

- **LOW** — `contracts/stencils/webster/webster-body-verify-fix.md` (new first bullet under "The rules"). The body's header comment says the same body is joined behind both the fork prefix and the shared cold frame `gatefix-frame-cold.md`. That frame already carries an "External tools and processes" section with the same two rules. A cold verify fixer therefore gets both rules twice, once in the frame and once in the body. The wording also differs: the frame says "a test in the repository that the gate runs", while the body adds `--tier` and the report check. Evidence: `gatefix-frame-cold.md:30-33` and the added bullet. Suggested fix: keep the bullet, but note in the body's header comment that the cold frame repeats the first two rules, or drop the sentence from the frame.
- **LOW** — `internal/shellguard/doc.go:67`. It says the row's way forward is "a tmux-tier test run through `lym gate test`", which does not mention `--tier`. It is not false, since the steer now names `--tier`, but it is the one remaining description of the way forward that omits the flag the F1 fix added. It could read "run through `lym gate test --tier`".

Everything else agrees:
- No stale steer wording remains anywhere in the repo. The grep for "form your instructions give" finds nothing.
- The `rows.go:388` comment still fits the new steer.
- The darn ungated fragment now matches the gated fragment's measuring call, which also reaches the tmux tier.
- The `DarnTable` doc and the gated fragment both name two exceptions.
- `markers_test.go` and the other tests that reference the changed stencils were not read. The new sections add no `{{...}}` markers, so I expect nothing to break.

Tool calls (2 of 6):
1. `grep` for the old steer wording and for tmux-tier references. My context did not hold `doc.go`, the tests or the docs.
2. `grep` for Go files, JSON, TXT and YAML that reference the three changed stencils. This was to see whether any digest or pin covers them. It lists the files only, so I did not inspect their contents for pins.
