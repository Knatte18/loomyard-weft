```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Set searches, one per finding:
- F1 (verb literals): the only remaining Go-code literals are the declarations. Those are `RemoteControlBarVerb` and `RemoteControlOnVerb` in `internal/reedengine/bar.go:52-53`, and `RemoteControlLit = "rc-on"` at `bar.go:45`, which is the state word and not the verb. The rest are comments and test expectations. The `panebin_test.go` pinned-line literals deliberately pin the composed output.
- F2 (stale claims): no "always sends", "lit or not" or dim-Codex-button text remains in `.go` or `.md` files outside the review and `.lym` trees. `orchcli/doc.go` and `reedengine/doc.go` mention only the verbs and say nothing about always sending or a dim Codex button.
- F3 (live-strand rows): I did not rerun this search. I judged it from context. The RC-word assertion now runs for every row that reaches the success path. The `withdrawErr` row returns early, and its early return is the only exit before the assertion. The new `watcher: true` row with a `g1` record reads lit.

Calls:
1. `grep` over `.go` and `.md` for `rc-on|rc-bar`, plus a second pattern for the stale-claim phrases, in one Bash call. My context held only the earlier `.go` grep, and it excluded docs.
