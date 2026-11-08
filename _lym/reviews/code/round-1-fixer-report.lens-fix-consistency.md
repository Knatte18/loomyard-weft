```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens: I checked whether the F1–F3 changes left any twin text or sibling unchanged.

**Findings**

- **LOW** — `internal/orchcli/start_test.go:1337` and `internal/orchengine/watcher_test.go:1521`. Both test comments still say "sets the hub's presence to slack". F1 renamed the presence to "the machine's" in the production doc comments, so these twins now disagree. The review's `fix` key listed only the production files, but its premise (the presence is now the machine's) covers these two lines. The evidence is the grep for `hub's presence` across `.go` and `.md` files, which returns only these two hits.

I found nothing else. These other changes agree with their surroundings:

- The remaining "hub's" hits in `internal/slackengine/*.go` and `internal/slackcli/cli.go` refer to the hub's relay, orchs, beams or channels. Those are still per-hub, so they were correctly left alone.
- `prompt.go:29` keeps "the hub's SLACK program". That is a program name, not the presence.
- The new `logger.Warn` calls in `hook.go` follow the `package: area: message` style used in `sheddriver` and `slackengine`, with key-value pairs.
- The new `strand` variable in `replyHook` replaces `_`, and `orchClient`'s signature is unchanged.
- The new test file has a file header, uses tier 1, and sits beside `testmain_test.go`. The tier matches the external-binary-spawn definition in `docs/benchmarks/running-tests.md`.

**Tool calls**

1. `grep -rn "hub's presence|hub's Slack presence"` over `.go` and `.md` files. My context held only the files the fixer read, not the repo-wide set of matching comments.
