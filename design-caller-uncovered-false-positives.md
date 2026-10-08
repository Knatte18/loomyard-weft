# caller-uncovered emits false-positive informational entries for type-name and method-name matches

`caller-uncovered-false-positives` · Note · bug

Imported from [issue #461](https://github.com/Knatte18/loomyard/issues/461).

**What happened:** Adding re-sign arrows to a plan card made the `caller-uncovered` check emit 75 informational entries, all false positives.
Every file that names a type counted as a caller of a re-signed method on a different type that happens to share the name, and any `.start(` call anywhere counted as a caller of a re-signed `start` method on one specific type.
The real callers had to be separated out by grep.

**Why it is abnormal:** The matching is by name only, not by receiver type, so the noise can hide a real uncovered caller.

**What to do:** Resolve callers by receiver type (or at least qualify the match by package and receiver) in `caller-uncovered`, and rate-limit or collapse name-only matches.

Provenance: task `webster-dead-ends`, note `burler-plan-r4.md`.
