```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: no findings.

**Unchecked:** none.

- **F1 (the set of refusal rows that should name their key).**
  - The review asked only for `UnknownEffort` to pin `efforts.haiku`, and the fix does that.
  - Other `unusable` sites also build `effortsKey+"."+key` (`config.go:211` empty effort, `config.go:218` malformed pair key). The review did not name them, so leaving them unpinned is not a finding.
  - The message at `config.go:214` carries `effortsKey+"."+key`. I did not read `unusable` at `config.go:225` to confirm the key lands in `err.Error()`, but the fix gate ran green with the new `strings.Contains` check.
- **F2 (the sites that apply the effort default).**
  - `effortFor` is called in `claudeengine.go:135` for runs and in `command.go:435` for `OpenLaunch`, so `OpenLaunch` is the only open path that applies the default.
  - The `OpenLaunch` doc comment is the only comment that needed the added sentence. The interface doc at `engine.go:353` is engine-neutral and says nothing about the effort. `codexengine`'s `OpenLaunch` ignores the config argument, so the added sentence does not apply to it.

Tool calls:
1. Grepped Go sources for `effortFor(` and `OpenLaunch` sites. My context did not hold the callers or the interface doc.
2. Grepped `unusable(` in `internal/agentconfig`. My context did not hold the other effort-key refusal sites.
