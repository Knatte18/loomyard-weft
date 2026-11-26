# Guard rows: an update mode for the settings golden, and a row that can state an exception

`guard-row-tooling` · Task · enhancement, process, claude, gate · running · Code-Burler-Fix

## Part 1: an update mode for the Claude settings golden

Imported from [issue #257](https://github.com/Knatte18/loomyard/issues/257).

### What happened

The Claude settings golden, `internal/shuttleengine/claudeengine/testdata/settings-golden.json`, has no update flag.
`TestBuildSettings_MatchesGolden` only compares against it.
Each change to a shellguard steer or pattern needed a temporary test file to render the new golden.

### Why it is abnormal

Every guard-row change rewrites this golden, and each agent making one has to write and then remove its own renderer.
That costs turns, and a leftover or wrong renderer is a risk to the commit.

### What to do

Add an update mode to the golden test that rewrites the file from the rendered settings and deny notice, reachable through `lym gate test`.

Provenance: task `reed-client-and-bus-guard`, note `burler-code-r1.md`.

## Part 2: a shellguard row can state an exception

Imported from [issue #256](https://github.com/Knatte18/loomyard/issues/256).

### What happened

A review finding asked for an exception in a shellguard row: refuse a command unless it carries a given flag.
A `Row` in `internal/shellguard/rows.go` denies only when every one of its `Patterns` matches, and `grep -E` has no negation, so the row has no way to state the exception.
The first fix spelled "any word but `--system`" as one regex with quote and escape parsing.
It failed open on input it could not parse.
The fixer's self-check caught it, and a second version replaced it.

### Why it is abnormal

A guard row with an exception has to encode negation by hand in a regex, which is hard to read and review.
A mistake in it can let a refused command through.

### What to do

Give `Row` a way to state an exception, such as patterns that must not match, rendered by each engine's guard builder.
The design has to keep the guard failing closed: an exception matched anywhere in the payload would let a second, refused command in the same call through.
If rows are to stay AND-only, make the hand-built helper (`unquotedWordOtherThan`) the documented way to write an exception.

Provenance: task `reed-client-and-bus-guard`, note `burler-code-r1.md`.
