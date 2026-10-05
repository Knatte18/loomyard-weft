# Burler review with invalid YAML frontmatter fails the run after three attempts

`burler-yaml-frontmatter-repair` · Note · bug, burler

Imported from [issue #386](https://github.com/Knatte18/loomyard/issues/386).

A loom run halted as `failed` at the `Discussion-Burler` producer (burler adapter in `internal/shedadapters`).

Halt output:

```
shedadapters: Discussion-Burler (burler): round 3 attempt 3: run: burler: round reached done but its review file is invalid: burler: review file frontmatter is not valid YAML: yaml: line 12: did not find expected key (a summary containing a " character must be ONE double-quoted string covering the whole value, not a quoted fragment followed by trailing prose)
```

The reviewer agent wrote a review file whose frontmatter summary contained a quoted fragment followed by trailing prose, which is invalid YAML.
The error text already states the rule, yet the run still failed at round 3 attempt 3 with history_entries 8.
Possible fixes: have the burler round prompt show the quoting rule up front, feed the parse error back to the reviewer for a repair turn, or have the frontmatter reader tolerate or normalise an unquoted summary.

Provenance: task `test-suite-measure`, from note `loom-halt.md`.
