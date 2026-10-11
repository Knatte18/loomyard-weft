```yaml
task: batten-notice-trim
notes_read:
  - loom-halt.md
issues_filed:
  - https://github.com/Knatte18/loomyard/issues/549
```

`loom-halt.md` was a `producer-hard-failure` at `Darn-Write`.
The trace showed idle detection rejecting a ready pane that had a stray `%` after the prompt, so it went into issue 549.
