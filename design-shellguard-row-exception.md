# shellguard: a row cannot state an exception, so negation is hand-built in regex

`shellguard-row-exception` · Note · enhancement, process

Imported from [issue #256](https://github.com/Knatte18/loomyard/issues/256).

## What happened

A review finding asked for an exception in a shellguard row: refuse a command unless it carries a given flag.
A `Row` in `internal/shellguard/rows.go` denies only when every one of its `Patterns` matches, and `grep -E` has no negation, so the row has no way to state the exception.
The first fix spelled "any word but `--system`" as one regex with quote and escape parsing.
It failed open on input it could not parse.
The fixer's self-check caught it, and a second version replaced it.

## Why it is abnormal

A guard row with an exception has to encode negation by hand in a regex, which is hard to read and review.
A mistake in it can let a refused command through.

## What to do

Give `Row` a way to state an exception, such as patterns that must not match, rendered by each engine's guard builder.
The design has to keep the guard failing closed: an exception matched anywhere in the payload would let a second, refused command in the same call through.
If rows are to stay AND-only, make the hand-built helper (`unquotedWordOtherThan`) the documented way to write an exception.

Provenance: task `reed-client-and-bus-guard`, note `burler-code-r1.md`.
