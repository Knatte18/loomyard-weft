# Burler review round fails hard on invalid review-file YAML frontmatter

`review-frontmatter-hard-fail` · Task · bug, burler, shed · running · Drift-Recheck

Imported from [issue #151](https://github.com/Knatte18/loomyard/issues/151).

## What happened

A `lym loom run` halted as `failed` at the `Code-Burler-Review` producer with anomaly `producer-hard-failure`.
The halt reason was:

```
shedadapters: Code-Burler-Review (burler-review): round 1 attempt 1: review: burler: round reached done but its review file is invalid: burler: review file frontmatter is not valid YAML: yaml: line 8: mapping values are not allowed in this context
```

The round reached `done`, but the review file the reviewer wrote had frontmatter that does not parse as YAML.
The run failed on round 1, attempt 1, with no repair or re-prompt of the reviewer.

## Why it is abnormal

An unquoted colon in a frontmatter value is a common slip for an agent writing a review file.
Treating it as a hard failure ends the whole run over a formatting error the reviewer could fix in one turn.
The halt note also shows `Code-Bouncer` as `stuck` in history while the halt names `Code-Burler-Review`, which makes the cause harder to trace.

## What to do

- On an invalid review-file frontmatter, send the parse error back to the reviewer for a repair attempt before failing the run.
- Have the review prompt state that frontmatter values containing `:` must be quoted.
- Have the halt note say which producer's output was invalid and why the preceding `stuck` step did not stop the run first.

Provenance: task `board-view-not-refreshing`; notes: `loom-halt.md`.


## From issue #152

Imported from [issue #152](https://github.com/Knatte18/loomyard/issues/152).

## What happened

`lym loom run` halted as `failed` with anomaly `producer-hard-failure` at the `Code-Burler-Review` producer.
The failure text was:

```
shedadapters: Code-Burler-Review (burler-review): round 1 attempt 1: review: burler: round reached done but its review file is invalid: burler: review file frontmatter is not valid YAML: yaml: line 23: mapping values are not allowed in this context
```

The review round itself had finished (`done`); only the review file's frontmatter failed to parse.
The halt came on round 1, attempt 1, so the run gave the reviewer no chance to correct the file.

## Why it is abnormal

A malformed review file is a recoverable, reviewer-side mistake (an unquoted value containing `: ` in a YAML frontmatter field is enough).
Treating it as a hard producer failure ends the whole run, and the review's findings are lost with it.
The halt note also lists the last history entry as `Code-Bouncer / stuck`, while the halt names `Code-Burler-Review`, so the record does not make clear which segment failed.

## What to do

- On an invalid review-file frontmatter, send the reviewer a repair prompt quoting the YAML error and retry the same attempt, instead of halting.
- Consider writing review-file frontmatter through a verb that serialises the YAML (quoting values) rather than having the agent hand-write it.
- Name the failing segment consistently in the halt note's `producer` and `history` lines.

Provenance: task `import-loses-issue-evidence`; note `loom-halt.md`.
