# No sanctioned Go formatter verb for agents that may not run rewrite scripts

`agent-gofmt-verb` · Note · enhancement, shuttle

Imported from [issue #578](https://github.com/Knatte18/loomyard/issues/578).

An implementing agent ran `gofmt -w` once on a test file to fix a map literal's formatting, and recorded that this was against the agents' no-rewrite-script rule.
It only reformatted the literal, but the agent had no sanctioned way to format Go files: Edit and Write are the only permitted file changes, and no `lyx` verb formats.

This is abnormal because the rule and the tooling conflict: formatting-only fixes force either a rule breach or manual whitespace edits.
What to do: provide a `lyx` formatting verb (or gate step) that agents may run, or state in the agent prompt that `gofmt -w` on a single file is allowed.

Provenance: task `merge-review`; notes 18-merge-in-row.md.
