```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-ran the F1 and F2 searches over production and test comments to check that the fix covers every member.

**Findings left unchecked:** none of the set-premise findings. I re-ran F1 and F2 by grep. F3, F4, F7 and F9 name closed sets, which I checked from context without a new search.

**Findings**

1. **LOW (arguable): the F1 comment sweep (commit `8505f978c`).** A few comments still point at one specific key through `<file>.yaml's <words>` or `<module> config's <words>`, while the fix rewrote the same kind of phrase elsewhere.
   - `internal/shuttleengine/requires.go:47`: "came from shuttle.yaml's default" means `shuttle.engine`.
   - `internal/orchcli/remotecontrol.go:33`: "whatever shuttle.yaml's default engine is" means `shuttle.engine`.
   - `internal/gatecli/cardfixer.go:143`: "its timeout is the gate config's fixer timeout" means `gate.fixer_timeout_min`. The fix rewrote this exact phrase to that name in `internal/battencli/wire.go`.
   - `internal/shuttleengine/codexengine/skillload.go:109`: "codex.yaml's skill roots" means `codex.skill_roots`.

   None of these spells the key's name, so reading them as generic prose is defensible.

I found no other bare member:
- **Remaining underscored hits:** the fixer's last uncached comment scan leaves only recipe, entry, state, envelope and marker fields, GitHub's `pull_request` rule, YAML examples, and the guard's own illustrations.
- **Field-like names I checked:** `notice_probe_s` (`internal/dobbyengine/loop.go:96`) is a recipe row key, not a module template key.
- **Other `X.yaml's` hits:** they name a file, its template, or keys generically.

Tool calls (4 of 6):
- Grep for `<file>.yaml's <word>` in comments: my context had only the pre-fix hits.
- Grep for `<module>'s <key>` in comments: my context did not have this phrasing search after the fix.
- Grep for `the <module> <key>`, backticked one-word keys and a line ending in `.yaml`: my context had no such search.
- Grep for `notice_probe_s` and `prime_owner` in templates and code: my context did not show whether they are template keys.
